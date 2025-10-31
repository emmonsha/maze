# Maze Generation Algorithms

## Overview

Maze generation is the process of creating a maze that is "perfect" - meaning it has exactly one path between any two points, with no loops and no isolated areas. This guide covers Eller's algorithm, which is one of the most efficient algorithms for generating perfect mazes.

## Eller's Algorithm

Eller's algorithm is a row-by-row maze generation algorithm that ensures the maze is perfect by maintaining specific set properties throughout the generation process.

### Key Concepts

1. **Sets**: Each cell in the current row belongs to a set
2. **Horizontal Connections**: Join cells in the same row that are in different sets
3. **Vertical Connections**: Connect downward to ensure all sets have at least one connection
4. **Final Row**: Join all remaining sets to complete the maze

### Algorithm Steps

#### Step 1: Initialize Sets
Each cell in the first row starts in its own set (identified by its column index).

#### Step 2: Process Each Row
For each row in the maze:
1. **Join horizontally**: Randomly join adjacent cells in the same row if they're in different sets
2. **Connect vertically**: For each set, ensure at least one cell connects downward to the next row
3. **Prepare next row**: Initialize sets for the next row

#### Step 3: Final Row
In the last row, join all cells that are still in different sets.

### Implementation

```python
def generate_eller_maze(rows: int, cols: int) -> Maze:
    maze = Maze(rows, cols)
    
    # Each cell in the current row starts in its own set
    cell_sets = list(range(cols))  # [0, 1, 2, ..., cols-1]
    
    for row in range(rows):
        # Step 1: Join adjacent cells horizontally (randomly)
        for col in range(cols - 1):
            set1, set2 = cell_sets[col], cell_sets[col + 1]
            
            # Only join if in different sets (to prevent loops)
            if set1 != set2 and random.random() < 0.5:
                # Remove wall between (row, col) and (row, col+1)
                maze.vertical_walls[row][col + 1] = False
                
                # Union: merge set2 into set1
                old_set_id = set2
                new_set_id = set1
                for c in range(cols):
                    if cell_sets[c] == old_set_id:
                        cell_sets[c] = new_set_id

        # Step 2: Add downward connections (except for last row)
        if row < rows - 1:
            # Group cells by their set
            sets_dict = {}
            for col in range(cols):
                set_id = cell_sets[col]
                if set_id not in sets_dict:
                    sets_dict[set_id] = []
                sets_dict[set_id].append(col)
            
            # For each set, connect at least one cell downward
            for set_id, cells_in_set in sets_dict.items():
                # Connect at least one cell per set (random number)
                num_connections = random.randint(1, len(cells_in_set))
                cells_to_connect = random.sample(cells_in_set, num_connections)
                
                for col in cells_to_connect:
                    # Remove wall between (row, col) and (row+1, col)
                    maze.horizontal_walls[row + 1][col] = False
            
            # Prepare sets for next row
            # Cells that connected downward keep same set ID
            # Cells that didn't connect start in their own new set
            next_row_sets = []
            for col in range(cols):
                if not maze.horizontal_walls[row + 1][col]:  # If connected downward
                    next_row_sets.append(cell_sets[col])  # Keep same set ID
                else:
                    # Use new set ID for cells that didn't connect downward
                    next_row_sets.append((row + 1) * cols + col)
            
            cell_sets = next_row_sets
        else:
            # Step 3: Final row - join all remaining different sets
            for col in range(cols - 1):
                set1, set2 = cell_sets[col], cell_sets[col + 1]
                if set1 != set2:
                    # Join different sets - remove wall
                    maze.vertical_walls[rows - 1][col + 1] = False
                    
                    # Union: merge set2 into set1
                    old_set_id = set2
                    new_set_id = set1
                    for c in range(cols):
                        if cell_sets[c] == old_set_id:
                            cell_sets[c] = new_set_id
    
    return maze
```

## Why Eller's Algorithm Works

### Perfect Maze Properties

1. **Connectivity**: Each set has at least one downward connection, ensuring all cells are reachable
2. **No loops**: Horizontal connections only happen between different sets, preventing loops
3. **Complete**: The final row joins all remaining sets, ensuring the maze is fully connected

### Memory Efficiency

Eller's algorithm only needs to remember the current row's set information, making it memory-efficient with O(w) space complexity (where w is the width/maze columns).

## Other Generation Algorithms

### Recursive Backtracking
- Uses DFS approach
- Creates long passages with fewer dead ends
- Good for mazes with a more "natural" feel

### Kruskal's Algorithm
- Uses Union-Find data structure
- Considers all possible walls at once
- Produces mazes with many short dead ends

### Prim's Algorithm
- Randomized version of the minimum spanning tree algorithm
- Creates mazes with many short passages

## Algorithm Comparison

| Algorithm | Memory | Characteristics |
|-----------|--------|-----------------|
| Eller's | O(w) | Row-by-row, good balance |
| Recursive Backtracking | O(w×h) | Long passages |
| Kruskal's | O(w×h) | Many short dead ends |
| Prim's | O(w×h) | Many short passages |

## Practical Implementation Tips

1. **Randomization**: Use appropriate randomization to avoid predictable patterns
2. **Set Management**: Efficient set union operations are crucial for performance
3. **Border Handling**: Always ensure border walls remain intact
4. **Validation**: Check that generated mazes are actually perfect

Eller's algorithm is particularly well-suited for this project due to its memory efficiency and ability to generate perfect mazes row-by-row.