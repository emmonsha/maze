# Maze Solving Algorithms

## Overview

Maze solving involves finding a path from a start point to an end point. The most common approach for finding the shortest path in an unweighted graph like a maze is Breadth-First Search (BFS).

## Breadth-First Search (BFS) Algorithm

### Why BFS?

BFS is ideal for maze solving because:
- It finds the shortest path (minimum number of steps)
- It explores all nodes at the current depth before moving to the next
- It's guaranteed to find the solution if one exists
- It's simple to implement and understand

### Algorithm Steps

1. **Initialize**: Add start position to queue, mark as visited
2. **Process**: Take first position from queue
3. **Check**: If it's the end, we found a solution
4. **Explore**: Add all unvisited neighbors to queue
5. **Repeat**: Until queue is empty or solution is found

### Implementation

```python
from collections import deque

def solve_maze(maze: Maze, start: Tuple[int, int], end: Tuple[int, int]) -> Optional[List[Tuple[int, int]]]:
    start_row, start_col = start
    end_row, end_col = end

    # Validate start and end points
    if not (0 <= start_row < maze.rows and 0 <= start_col < maze.cols):
        return None
    if not (0 <= end_row < maze.rows and 0 <= end_col < maze.cols):
        return None

    # If start and end are the same
    if start == end:
        return [start]

    # BFS setup
    queue = deque([start])
    visited = [[False for _ in range(maze.cols)] for _ in range(maze.rows)]
    visited[start_row][start_col] = True
    predecessors = {}  # To reconstruct the path

    # Directions: right, down, left, up
    directions = [(0, 1), (1, 0), (0, -1), (-1, 0)]

    while queue:
        curr_row, curr_col = queue.popleft()

        # Check if we reached the destination
        if (curr_row, curr_col) == end:
            break

        # Explore neighbors
        for dr, dc in directions:
            new_row, new_col = curr_row + dr, curr_col + dc

            # Check if the new position is valid and not visited
            if (0 <= new_row < maze.rows and 0 <= new_col < maze.cols and
                    not visited[new_row][new_col]):

                # Check if we can move in that direction (no wall)
                can_move = _can_move_in_direction(maze, curr_row, curr_col, dr, dc)

                if can_move:
                    visited[new_row][new_col] = True
                    queue.append((new_row, new_col))
                    predecessors[(new_row, new_col)] = (curr_row, curr_col)

    # If we didn't reach the end
    if not visited[end_row][end_col]:
        return None

    # Reconstruct path
    path = []
    current = end
    while current != start:
        path.append(current)
        current = predecessors[current]
    path.append(start)
    path.reverse()

    return path

def _can_move_in_direction(maze: Maze, row: int, col: int, dr: int, dc: int) -> bool:
    """Check if movement in given direction is possible (no wall)"""
    if dr == 0 and dc == 1:  # Moving right
        return not maze.has_right_wall(row, col)
    elif dr == 1 and dc == 0:  # Moving down
        return not maze.has_bottom_wall(row, col)
    elif dr == 0 and dc == -1:  # Moving left
        return not maze.has_left_wall(row, col)
    elif dr == -1 and dc == 0:  # Moving up
        return not maze.has_top_wall(row, col)
    return False
```

## Alternative Approaches

### Depth-First Search (DFS)

- Uses a stack instead of a queue
- May find a solution faster but not necessarily the shortest
- Uses less memory in some cases
- Implementation is similar but uses `append()` and `pop()` (LIFO)

```python
def solve_maze_dfs(maze: Maze, start: Tuple[int, int], end: Tuple[int, int]) -> Optional[List[Tuple[int, int]]]:
    stack = [start]
    visited = [[False for _ in range(maze.cols)] for _ in range(maze.rows)]
    predecessors = {}
    visited[start[0]][start[1]] = True
    
    # Similar to BFS but with stack operations
    # ... implementation details
```

### A* Algorithm

- More complex but can be faster for large mazes
- Uses heuristic to guide search toward the goal
- Guarantees shortest path if heuristic is admissible
- More computation per step but potentially fewer total steps

## Path Reconstruction

The `_reconstruct_path` function is crucial for getting the actual path:

```python
def _reconstruct_path(predecessors, start, end):
    path = []
    current = end
    while current != start:
        path.append(current)
        current = predecessors[current]
    path.append(start)
    path.reverse()
    return path
```

## Performance Considerations

### Time Complexity
- **BFS**: O(V + E) where V is vertices (cells) and E is edges (possible moves)
- For a maze: O(rows × cols)

### Space Complexity  
- **BFS**: O(V) for the queue and visited array
- For a maze: O(rows × cols)

### Optimization Tips

1. **Early Termination**: Stop as soon as end is reached
2. **Bidirectional Search**: Search from both start and end simultaneously
3. **Preprocessing**: Check if start and end are in the same connected component

## Visualization Integration

In the GUI context, the solved path needs to be rendered:

```python
def draw_solution_path(screen, path, cell_width, cell_height):
    if not path:
        return
        
    # Draw line through the center of each cell in the path
    for i in range(len(path) - 1):
        row1, col1 = path[i]
        row2, col2 = path[i + 1]
        
        x1 = col1 * cell_width + cell_width // 2
        y1 = row1 * cell_height + cell_height // 2
        x2 = col2 * cell_width + cell_width // 2  
        y2 = row2 * cell_height + cell_height // 2
        
        pygame.draw.line(screen, (255, 0, 0), (x1, y1), (x2, y2), 2)  # Red, 2px thick
```

## Error Handling

The solver should gracefully handle:
- Invalid start/end positions
- No possible path exists
- Start and end are the same location
- Start and end are in isolated maze sections

BFS is the optimal choice for this project because it guarantees the shortest path while maintaining simplicity and reliability.