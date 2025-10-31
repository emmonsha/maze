# GUI Development with Pygame

## Overview

The GUI provides an interactive interface for maze generation, visualization, and solving. Pygame is chosen for its simplicity in handling 2D graphics, user input, and game-like loops.

## Pygame Fundamentals

### Core Components

1. **Display Surface**: The window/canvas where everything is drawn
2. **Event Loop**: Handles user input (mouse, keyboard)
3. **Game Loop**: Updates state and redraws the screen
4. **Drawing Primitives**: Functions to draw shapes, text, and images

### Basic Pygame Setup

```python
import pygame
import sys

def main():
    pygame.init()
    
    # Create the display
    screen = pygame.display.set_mode((500, 500))
    pygame.display.set_caption("Maze Game")
    
    clock = pygame.time.Clock()
    
    # Main game loop
    running = True
    while running:
        # Handle events
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
        
        # Update game state
        
        # Draw everything
        screen.fill((255, 255, 255))  # White background
        
        # Draw game elements here
        
        pygame.display.flip()  # Update the display
        clock.tick(60)  # Cap at 60 FPS
    
    pygame.quit()
    sys.exit()
```

## Maze Display Implementation

### Coordinate System and Scaling

Each maze cell needs to be drawn to fit in the 500×500 pixel display:

```python
def calculate_cell_size(self) -> Tuple[int, int]:
    """Calculate the size of each cell to fill the display area."""
    if self.maze is None:
        return 10, 10  # Default size if no maze
        
    # Account for wall thickness
    available_width = self.WINDOW_WIDTH - self.WALL_THICKNESS
    available_height = self.WINDOW_HEIGHT - self.WALL_THICKNESS
    
    cell_width = available_width // self.maze.cols
    cell_height = available_height // self.maze.rows
    
    # Use the smaller of the two to ensure the maze fits
    cell_size = min(cell_width, cell_height)
    
    return cell_size, cell_size
```

### Drawing the Maze

```python
def draw_maze(self):
    """Draw the maze on the screen."""
    if self.maze is None:
        return
        
    cell_width, cell_height = self.calculate_cell_size()
    self.screen.fill(self.BG_COLOR)  # White background
    
    # Draw walls and cells
    for row in range(self.maze.rows):
        for col in range(self.maze.cols):
            x = col * cell_width
            y = row * cell_height
            
            # Draw right wall if it exists
            if self.maze.has_right_wall(row, col):
                pygame.draw.line(
                    self.screen, 
                    self.WALL_COLOR,
                    (x + cell_width, y),
                    (x + cell_width, y + cell_height),
                    self.WALL_THICKNESS
                )
            
            # Draw bottom wall if it exists
            if self.maze.has_bottom_wall(row, col):
                pygame.draw.line(
                    self.screen, 
                    self.WALL_COLOR,
                    (x, y + cell_height),
                    (x + cell_width, y + cell_height),
                    self.WALL_THICKNESS
                )
    
    # Draw borders
    pygame.draw.rect(self.screen, self.WALL_COLOR, 
                     (0, 0, self.WINDOW_WIDTH, self.WINDOW_HEIGHT), 
                     self.WALL_THICKNESS)
    
    # Draw solution path if it exists
    self.draw_solution()
    
    # Draw start/end points if set
    self.draw_points()
```

## Handling User Input

### Mouse Events

```python
def handle_click(self, pos: Tuple[int, int]):
    """Handle mouse click events."""
    if self.maze is None:
        return
        
    cell_width, cell_height = self.calculate_cell_size()
    
    # Convert pixel coordinates to maze coordinates
    col = pos[0] // cell_width
    row = pos[1] // cell_height
    
    # Ensure click is within maze bounds
    if 0 <= row < self.maze.rows and 0 <= col < self.maze.cols:
        if self.state == "select_start":
            self.start_pos = (row, col)
            self.state = "select_end"
        elif self.state == "select_end":
            self.end_pos = (row, col)
            if self.start_pos and self.end_pos:
                # Solve the maze
                self.solution = solve_maze(self.maze, self.start_pos, self.end_pos)
            self.state = "display"
```

### Keyboard Events

```python
def run(self):
    running = True
    
    while running:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_s:  # Select start
                    self.state = "select_start"
                elif event.key == pygame.K_e:  # Select end
                    self.state = "select_end"
                elif event.key == pygame.K_g:  # Generate new maze
                    self.maze = generate_maze(10, 10)
                    self.reset_solution()
    
    # ... rest of implementation
```

## State Management

The GUI needs to manage different states:

```python
class MazeGUI:
    def __init__(self, maze: Optional[Maze] = None):
        # ... initialization
        
        # For user interface
        self.state = "display"  # "display", "select_start", "select_end"
        self.mode = "maze"      # "maze", "cave"
        
    def reset_solution(self):
        self.solution = None
        self.start_pos = None
        self.end_pos = None
        self.state = "display"
```

## Advanced GUI Features

### File Dialog Integration

```python
import tkinter as tk
from tkinter import filedialog

# Hide the main tkinter window
tk.Tk().withdraw()

def load_maze_from_file(self):
    file_path = filedialog.askopenfilename(
        title="Load Maze File",
        filetypes=[("Text files", "*.txt"), ("All files", "*.*")]
    )
    if file_path:
        try:
            # Load maze from file
            temp_maze = Maze(1, 1)
            self.maze = temp_maze.load_from_file(file_path)
            self.reset_solution()
        except Exception as e:
            print(f"Error loading maze from file: {e}")
```

### Curtain/Overlay System

```python
def draw_curtain(self):
    """Draw a semi-transparent curtain overlay with information."""
    # Create a semi-transparent surface
    curtain = pygame.Surface((self.WINDOW_WIDTH, self.WINDOW_HEIGHT))
    curtain.set_alpha(180)  # Adjust transparency
    curtain.fill((200, 200, 200))  # Light gray curtain
    self.screen.blit(curtain, (0, 0))
    
    # Render centered text
    lines = [
        "Maze Game - Help",
        "",
        "S - Select start point",
        "E - Select end point", 
        "G - Generate new maze",
        "L - Load maze from file",
        "C - Clear solution",
        "H - Help (this screen)"
    ]
    
    line_height = 30
    total_height = len(lines) * line_height
    start_y = (self.WINDOW_HEIGHT - total_height) // 2
    
    for i, line in enumerate(lines):
        text_surface = self.font.render(line, True, (0, 0, 0))
        text_rect = text_surface.get_rect(center=(self.WINDOW_WIDTH // 2, start_y + i * line_height))
        self.screen.blit(text_surface, text_rect)
```

## Common Challenges and Solutions

### 1. Performance Issues
- **Problem**: Drawing large mazes can be slow
- **Solution**: Only redraw changed areas or use pygame's dirty rectangle updates

### 2. Coordinate Conversion
- **Problem**: Converting between pixel coordinates and maze coordinates
- **Solution**: Create helper functions with clear variable names

### 3. Scalability
- **Problem**: Different maze sizes requiring different cell sizes
- **Solution**: Dynamic calculation based on available display space

### 4. User Experience
- **Problem**: Providing clear feedback and instructions
- **Solution**: Clear UI messages and visual indicators

## Best Practices

1. **Separation of Concerns**: Keep drawing logic separate from game logic
2. **Performance**: Use appropriate drawing techniques for large mazes
3. **User Feedback**: Provide visual and textual feedback for all actions
4. **Error Handling**: Gracefully handle invalid operations
5. **Maintainability**: Use clear variable names and modular functions

## Integration with Other Components

The GUI must integrate with:
- Maze generation (to display generated mazes)
- Maze solving (to visualize solutions)  
- File I/O (for load/save operations)
- Other algorithms (cave generation, etc.)

Pygame provides an excellent foundation for creating interactive, visual applications like the maze game, balancing simplicity with power for 2D graphics and input handling.