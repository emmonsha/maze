# Maze Data Structure

## Understanding Maze Representation

A maze is a 2D grid where each cell can have walls on its borders. The key to representing a maze programmatically is to track which walls exist between adjacent cells.

### The "Thin Wall" Approach

In the "thin wall" representation:
- Each cell is a position in the grid
- Walls exist between cells (not inside them)
- The maze is surrounded by a border of walls

```text
+---+---+---+
|   |   |   |  <- Top border walls
+   +   +   +  <- Horizontal internal walls
|   |   |   |
+   +   +   +  <- Horizontal internal walls  
|   |   |   |
+---+---+---+  <- Bottom border walls
```

### Wall Storage

We store walls using two matrices:
- **Vertical walls**: A matrix indicating walls to the right of each cell
- **Horizontal walls**: A matrix indicating walls below each cell

For an n×m maze:
- Vertical walls matrix: n rows × (m+1) columns
- Horizontal walls matrix: (n+1) rows × m columns

## Implementing the Maze Class

### Basic Structure

```python
class Maze:
    def __init__(self, rows: int, cols: int):
        self.rows = rows
        self.cols = cols
        # Vertical walls: walls to the right of each cell (including borders)
        self.vertical_walls = [[True for _ in range(cols + 1)] for _ in range(rows)]
        # Horizontal walls: walls below each cell (including borders)  
        self.horizontal_walls = [[True for _ in range(cols)] for _ in range(rows + 1)]
        
        # Set border walls
        self._set_border_walls()
```

### Wall Access Methods

```python
def has_right_wall(self, row: int, col: int) -> bool:
    """Check for wall to the right of cell (row, col)"""
    if 0 <= row < self.rows and 0 <= col < self.cols:
        return self.vertical_walls[row][col + 1]
    return True  # Border case

def has_bottom_wall(self, row: int, col: int) -> bool:
    """Check for wall below cell (row, col)"""
    if 0 <= row < self.rows and 0 <= col < self.cols:
        return self.horizontal_walls[row + 1][col]
    return True  # Border case

def has_left_wall(self, row: int, col: int) -> bool:
    """Check for wall to the left of cell (row, col)"""
    if 0 <= row < self.rows and 0 <= col < self.cols:
        return self.vertical_walls[row][col]
    return True  # Border case

def has_top_wall(self, row: int, col: int) -> bool:
    """Check for wall above cell (row, col)"""
    if 0 <= row < self.rows and 0 <= col < self.cols:
        return self.horizontal_walls[row][col]
    return True  # Border case
```

## File Format

The maze file format is structured as:
```
rows cols
vertical_walls_row_0
vertical_walls_row_1
...
vertical_walls_row_(n-1)
horizontal_walls_row_0
horizontal_walls_row_1
...
horizontal_walls_row_n
```

### File I/O Implementation

```python
def save_to_file(self, filename: str):
    with open(filename, 'w') as f:
        # Write dimensions
        f.write(f"{self.rows} {self.cols}\n\n")
        
        # Write vertical walls
        for i in range(self.rows):
            row = [int(self.vertical_walls[i][j]) for j in range(self.cols + 1)]
            f.write(" ".join(map(str, row)) + "\n")
        f.write("\n")
        
        # Write horizontal walls
        for i in range(self.rows + 1):
            row = [int(self.horizontal_walls[i][j]) for j in range(self.cols)]
            f.write(" ".join(map(str, row)) + "\n")

def load_from_file(self, filename: str) -> 'Maze':
    with open(filename, 'r') as f:
        lines = [line.strip() for line in f if line.strip()]
    
    # First line has rows and columns
    rows, cols = map(int, lines[0].split())
    maze = Maze(rows, cols)
    
    # Next 'rows' lines represent vertical walls
    idx = 1
    for i in range(rows):
        wall_row = list(map(int, lines[idx].split()))
        for j in range(cols + 1):
            maze.vertical_walls[i][j] = bool(wall_row[j]) if j < len(wall_row) else True
        idx += 1
    
    # Next 'rows+1' lines represent horizontal walls
    for i in range(rows + 1):
        wall_row = list(map(int, lines[idx].split()))
        for j in range(cols):
            maze.horizontal_walls[i][j] = bool(wall_row[j]) if j < len(wall_row) else True
        idx += 1
    
    return maze
```

## Perfect Maze Validation

A perfect maze has exactly one path between any two points (no loops, no isolated areas). We can check this:

```python
def is_perfect(self) -> bool:
    # 1. Check connectivity (all cells reachable from start)
    visited = [[False for _ in range(self.cols)] for _ in range(self.rows)]
    stack = [(0, 0)]
    visited[0][0] = True
    count = 1

    directions = [(0, 1), (1, 0), (0, -1), (-1, 0)]
    
    while stack:
        row, col = stack.pop()
        for dr, dc in directions:
            new_row, new_col = row + dr, col + dc
            if (0 <= new_row < self.rows and 0 <= new_col < self.cols and 
                not visited[new_row][new_col]):
                
                # Check if we can move to adjacent cell (no wall)
                can_move = self._can_move(row, col, new_row, new_col)
                if can_move:
                    visited[new_row][new_col] = True
                    stack.append((new_row, new_col))
                    count += 1

    # All cells should be reachable
    if count != self.rows * self.cols:
        return False

    # 2. Check for correct number of connections (n*m - 1 for perfect maze)
    path_count = self._count_paths()
    return path_count == (self.rows * self.cols - 1)
```

## Key Concepts

1. **Modular Walls**: Walls are stored separately from cells
2. **Boundary Handling**: Border walls are always present
3. **Flexibility**: The structure allows easy wall modification
4. **Scalability**: Works for any reasonable maze size (≤50x50 per requirements)
5. **Persistence**: Easy to save/load with simple file format

This structure provides a solid foundation for maze generation, solving, and visualization algorithms.