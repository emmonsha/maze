# Cave Generation with Cellular Automata

## Overview

Cave generation uses cellular automata to create organic, cave-like structures. A cellular automaton is a grid of cells that evolves according to simple rules based on the states of neighboring cells.

## Cellular Automata Fundamentals

### Core Concepts

1. **Grid**: 2D array of cells
2. **States**: Each cell is either alive (wall) or dead (empty space)
3. **Neighbors**: Typically 8 adjacent cells (Moore neighborhood)
4. **Rules**: Simple conditions that determine next state
5. **Iterations**: Repeated application of rules

### The Algorithm

The cave generation algorithm has two phases:
1. **Initialization**: Randomly populate the grid
2. **Simulation**: Apply cellular rules repeatedly to evolve the pattern

## Implementation

### Basic Cave Class

```python
class Cave:
    def __init__(self, rows: int, cols: int, initial_chance: float = 0.45):
        self.rows = rows
        self.cols = cols
        # True = alive/wall, False = dead/empty
        self.grid = [[False for _ in range(cols)] for _ in range(rows)]
        self.initial_chance = initial_chance
        self._initialize_randomly()
    
    def _initialize_randomly(self):
        """Initialize the cave grid randomly based on the initial chance."""
        for i in range(self.rows):
            for j in range(self.cols):
                self.grid[i][j] = random.random() < self.initial_chance
```

### Simulation Rules

```python
def step_simulation(self, birth_limit: int = 4, death_limit: int = 3) -> bool:
    """
    Perform one step of the cellular automaton simulation.
    
    Rules:
    - If a dead cell has > birth_limit living neighbors, it becomes alive
    - If a living cell has < death_limit living neighbors, it dies
    """
    new_grid = [[False for _ in range(self.cols)] for _ in range(self.rows)]
    changed = False
    
    for i in range(self.rows):
        for j in range(self.cols):
            # Count living neighbors
            neighbors = self._count_living_neighbors(i, j)
            
            # Apply rules
            if self.grid[i][j]:  # If cell is alive
                if neighbors < death_limit:
                    # Dies due to underpopulation
                    new_grid[i][j] = False
                    changed = True
                else:
                    # Survives
                    new_grid[i][j] = True
            else:  # If cell is dead
                if neighbors > birth_limit:
                    # Becomes alive due to reproduction
                    new_grid[i][j] = True
                    changed = True
                else:
                    # Remains dead
                    new_grid[i][j] = False
    
    # Update the grid
    self.grid = new_grid
    return changed

def _count_living_neighbors(self, row: int, col: int) -> int:
    """Count living neighbors around a cell (Moore neighborhood)."""
    neighbors = 0
    
    # Check all 8 directions: up, down, left, right, and diagonals
    for di in [-1, 0, 1]:
        for dj in [-1, 0, 1]:
            if di == 0 and dj == 0:
                continue  # Skip the cell itself
            
            ni, nj = row + di, col + dj
            
            # Cells outside the cave are treated as alive (walls)
            if 0 <= ni < self.rows and 0 <= nj < self.cols:
                if self.grid[ni][nj]:
                    neighbors += 1
            else:
                # Treat out-of-bounds as alive (creates border)
                neighbors += 1
    
    return neighbors
```

### Running Multiple Steps

```python
def run_simulation(self, birth_limit: int = 4, death_limit: int = 3, steps: int = 5) -> int:
    """
    Run multiple steps of the cellular automaton simulation.
    Returns number of steps performed before stabilization.
    """
    for step in range(steps):
        changed = self.step_simulation(birth_limit, death_limit)
        if not changed:
            # If no changes occurred, the cave has stabilized
            return step + 1
    
    return steps
```

## Parameter Control

### Birth and Death Limits

- **Birth Limit**: Minimum number of living neighbors for a dead cell to become alive
- **Death Limit**: Maximum number of living neighbors for a living cell to remain alive

```text
Low Birth Limit (e.g., 2-3): More cells become alive → denser caves
High Birth Limit (e.g., 5-6): Fewer cells become alive → sparser caves
Low Death Limit (e.g., 1-2): More cells die → less connected caves  
High Death Limit (e.g., 4-5): More cells survive → more connected caves
```

### Initial Chance

The initial chance determines how many cells start as alive:

```text
Low initial chance (0.3-0.4): Fewer initial walls → more open caves
High initial chance (0.5-0.7): More initial walls → more enclosed caves
```

## File I/O Implementation

```python
def save_to_file(self, filename: str):
    """Save the cave to a file."""
    with open(filename, 'w') as f:
        # Write dimensions
        f.write(f"{self.rows} {self.cols}\n")
        
        # Write grid (1 for alive, 0 for dead)
        for i in range(self.rows):
            row = [int(self.grid[i][j]) for j in range(self.cols)]
            f.write(" ".join(map(str, row)) + "\n")

def load_from_file(self, filename: str) -> 'Cave':
    """Load a cave from a file."""
    with open(filename, 'r') as f:
        lines = [line.strip() for line in f if line.strip()]
    
    # First line has rows and columns
    rows, cols = map(int, lines[0].split())
    cave = Cave(rows, cols)
    
    # Remaining lines represent the cave grid
    for i in range(rows):
        row_data = list(map(int, lines[i + 1].split()))
        for j in range(cols):
            cave.grid[i][j] = bool(row_data[j]) if j < len(row_data) else False
    
    return cave
```

## Visualization in GUI

### Pygame Drawing

```python
def draw_cave(self):
    """Draw the cave on the pygame screen."""
    cell_width, cell_height = self.calculate_cell_size()
    self.screen.fill(self.BG_COLOR)
    
    # Draw cave cells
    for row in range(self.cave.rows):
        for col in range(self.cave.cols):
            x = col * cell_width
            y = row * cell_height
            
            # Draw live (wall) cells in brown, dead (empty) cells in light gray
            if self.cave.grid[row][col]:
                # Cave wall
                pygame.draw.rect(self.screen, self.CAVE_WALL_COLOR, 
                                (x, y, cell_width, cell_height))
                # Draw border for visibility
                pygame.draw.rect(self.screen, (101, 67, 33), 
                                (x, y, cell_width, cell_height), 1)
            else:
                # Empty cave space
                pygame.draw.rect(self.screen, self.BG_COLOR, 
                                (x, y, cell_width, cell_height))
                # Draw border for visibility
                pygame.draw.rect(self.screen, (200, 200, 200), 
                                (x, y, cell_width, cell_height), 1)
```

### Web Interface Drawing

```javascript
function drawCave() {
    if (!currentCave) return;
    
    const canvas = document.getElementById('maze-canvas');
    const ctx = canvas.getContext('2d');
    
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    const cellWidth = canvas.width / currentCave.cols;
    const cellHeight = canvas.height / currentCave.rows;
    
    // Draw cave cells
    for (let row = 0; row < currentCave.rows; row++) {
        for (let col = 0; col < currentCave.cols; col++) {
            const x = col * cellWidth;
            const y = row * cellHeight;
            
            if (currentCave.grid[row][col]) {
                // Draw wall in brown
                ctx.fillStyle = '#8B4513'; // Brown
                ctx.fillRect(x, y, cellWidth, cellHeight);
            } else {
                // Draw empty space in light gray
                ctx.fillStyle = '#f0f0f0'; // Light gray
                ctx.fillRect(x, y, cellWidth, cellHeight);
            }
            
            // Draw cell border
            ctx.strokeStyle = '#cccccc';
            ctx.lineWidth = 1;
            ctx.strokeRect(x, y, cellWidth, cellHeight);
        }
    }
}
```

## Advanced Techniques

### Smoothing Iterations

Multiple simulation steps refine the cave structure:

```python
def smooth_cave(self, iterations: int = 5):
    """Apply multiple simulation steps to smooth the cave."""
    for _ in range(iterations):
        self.step_simulation(birth_limit=4, death_limit=3)
```

### Edge Treatment

Decide how to treat cells outside the cave boundary:

1. **Always Alive (Walls)**: Creates natural borders
2. **Random**: Adds randomness to edges
3. **Based on Average**: Smooths edges

### Tunnel Formation

To create more tunnel-like structures, you can:
- Use different birth/death rules for different areas
- Apply post-processing to create tunnels between large open spaces
- Use different initial distributions

## Parameter Tuning Guidelines

### For Open Caves
- Lower initial chance (0.35-0.45)
- Lower birth limit (3-4)
- Higher death limit (4-5)

### For Dense Caves
- Higher initial chance (0.55-0.65)  
- Higher birth limit (5-6)
- Lower death limit (2-3)

### For Natural-Looking Caves
- Initial chance: 0.40-0.50
- Birth limit: 4 (default)
- Death limit: 3 (default)

## Integration with Other Components

The cave class can be integrated with:
- File I/O system for loading/saving
- GUI for visualization
- Web interface for browser access
- Parameter controls for experimentation

Cellular automata provide an elegant and efficient way to generate organic, natural-looking cave systems with just a few simple rules and parameters.