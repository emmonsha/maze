# Documentation Creation and Management

## Overview

Documentation is crucial for code maintainability, team collaboration, and project understanding. This guide covers creating, maintaining, and accessing documentation for the Maze project.

## Types of Documentation

### 1. Code Documentation (Docstrings)

Python uses docstrings to document modules, classes, methods, and functions:

```python
class Maze:
    """
    Represents a maze with thin walls.

    A maze is represented as a grid of cells with walls that can exist between cells
    and around the perimeter of the maze. This class provides methods to create,
    manipulate, and query maze properties.
    """

    def __init__(self, rows: int, cols: int):
        """
        Initialize an empty maze with the specified dimensions.

        Args:
            rows (int): Number of rows in the maze
            cols (int): Number of columns in the maze
        """
        self.rows = rows
        self.cols = cols
        self.horizontal_walls = [[True for _ in range(cols)] for _ in range(rows + 1)]
        self.vertical_walls = [[True for _ in range(cols + 1)] for _ in range(rows)]
        self._set_border_walls()
```

### 2. Google Style Documentation

Google style is used for this project:

```python
def solve_maze(maze: Maze, start: Tuple[int, int], end: Tuple[int, int]) -> Optional[List[Tuple[int, int]]]:
    """
    Find the shortest path in a maze using BFS algorithm.

    This function implements breadth-first search to find the shortest path
    between start and end points in a maze. It checks for valid moves based
    on the presence or absence of walls between cells.

    Args:
        maze (Maze): The maze to solve
        start (Tuple[int, int]): Starting position (row, col)
        end (Tuple[int, int]): Ending position (row, col)

    Returns:
        Optional[List[Tuple[int, int]]]: List of positions representing the path,
        or None if no path exists

    Raises:
        ValueError: If start or end positions are outside maze bounds

    Example:
        >>> maze = generate_maze(5, 5)
        >>> path = solve_maze(maze, (0, 0), (4, 4))
        >>> if path:
        ...     print(f"Found path with {len(path)} steps")
    """
    # Implementation details...
```

### 3. Type Hints

Documentation is enhanced with Python type hints:

```python
from typing import List, Tuple, Optional, Dict

def generate_eller_maze(rows: int, cols: int) -> Maze:
    """Generate a perfect maze using Eller's algorithm.
    
    Args:
        rows: Number of rows in the maze
        cols: Number of columns in the maze
        
    Returns:
        A perfectly generated Maze object with no loops or isolated areas
    """
    # Implementation with type hints makes documentation clearer
    cell_sets: List[int] = list(range(cols))
    maze: Maze = Maze(rows, cols)
    # ...
```

## Documentation Tools

### 1. Pydoc (Built-in)

Python's built-in documentation generator:

```bash
# Generate HTML documentation
python -m pydoc -w maze_game

# View documentation in terminal
python -m pydoc maze_game.maze.maze.Maze
```

### 2. pdoc3 (Third-party)

More advanced documentation generator:

```bash
# Install pdoc3
pip install pdoc3

# Generate HTML documentation
pdoc3 --html --output-dir ./docs maze_game

# Generate for specific modules
pdoc3 --html --output-dir ./docs maze_game.maze
```

### 3. Sphinx (Advanced)

For extensive documentation with cross-references:

```python
# In docs/conf.py
extensions = [
    'sphinx.ext.autodoc',
    'sphinx.ext.viewcode',
    'sphinx.ext.napoleon'  # For Google/NumPy style docstrings
]

# Generate documentation
sphinx-build -b html source build
```

## Auto-Documentation Generation

### Makefile Integration

The dvi target in the Makefile automatically generates documentation:

```makefile
dvi:
	@echo "Generating documentation..."
	if command -v pdoc3 > /dev/null; then \
		pdoc3 --html --output-dir ./docs maze_game --force; \
		echo "Documentation generated in ./docs/maze_game"; \
	elif command -v pydoc > /dev/null; then \
		pydoc -w maze_game; \
		echo "Documentation generated in maze_game.html"; \
	else \
		echo "Neither pdoc3 nor pydoc found. Install one to generate documentation."; \
	fi
	# OS-specific opening of documentation
	if [ "$$(uname)" = "Darwin" ]; then \
		# Open documentation on macOS \
		open ./docs/maze_game/index.html 2>/dev/null || open maze_game.html 2>/dev/null || echo "Could not open browser"; \
	fi
```

### With Error Handling

```makefile
dvi:
	@echo "Generating documentation..."
	@mkdir -p docs
	@if command -v pdoc3 > /dev/null; then \
		echo "Using pdoc3..."; \
		pdoc3 --html --output-dir ./docs --force maze_game 2>/dev/null && \
		echo "Documentation generated in ./docs/maze_game/"; \
	elif command -v pydoc > /dev/null; then \
		echo "Using pydoc..."; \
		python -m pydoc -w maze_game && \
		echo "Documentation generated in maze_game.html"; \
	else \
		echo "Error: Neither pdoc3 nor pydoc found. Install one to generate documentation."; \
		exit 1; \
	fi
```

## Project-level Documentation

### README.md Structure

```markdown
# Maze Project

Implementation of the Maze project with generation, solving, and visualization.

## Table of Contents
1. [Installation](#installation)
2. [Usage](#usage) 
3. [Features](#features)
4. [API Reference](#api-reference)
5. [Contributing](#contributing)
6. [License](#license)

## Installation

```bash
pip install -r requirements.txt
```

## Usage

### Running the GUI
```bash
python -m maze_game
```

### Starting the Web Interface
```bash
make web
# or
python -m maze_game.web.app
```

## Features

- Perfect maze generation using Eller's algorithm
- Maze solving with BFS
- Cave generation using cellular automata
- Reinforcement learning with Q-learning
- Cross-platform GUI with pygame
- Web interface with Flask
```

### API Documentation Structure

```python
"""
Maze Game API Documentation

This module provides the core functionality for maze generation, solving, 
and visualization. The API is organized as follows:

Maze Core:
    - maze.maze: Basic maze representation and operations
    - maze.generator: Maze generation algorithms
    - maze.solver: Pathfinding and solving algorithms

User Interfaces:  
    - gui: Pygame-based desktop interface
    - web: Flask-based web interface

Special Features:
    - cave: Cellular automata cave generation
    - reinforcement_learning: Q-learning implementation

Testing:
    - tests: Comprehensive test suite
"""
```

## Documentation Best Practices

### 1. Be Clear and Concise

```python
def has_right_wall(self, row: int, col: int) -> bool:
    """
    Check if there's a wall to the right of the specified cell.

    Args:
        row: Row index of the cell
        col: Column index of the cell

    Returns:
        True if there's a wall to the right, False otherwise
    """
    # Good: Clear parameter names and return value explanation
```

### 2. Include Examples

```python
def load_from_file(self, filename: str) -> 'Maze':
    """
    Load a maze from a file.

    The file format is:
    rows cols
    vertical_walls_for_row_0
    vertical_walls_for_row_1
    ...
    horizontal_walls_for_row_0
    ...

    Args:
        filename: Path to the file containing the maze

    Returns:
        Loaded maze object

    Example:
        maze = Maze(1, 1)
        loaded_maze = maze.load_from_file('maze.txt')
    """
```

### 3. Document Complex Logic

```python
def is_perfect(self) -> bool:
    """
    Check if the maze is perfect (no isolated areas or loops).

    A perfect maze has exactly one path between any two points.
    This is verified by checking:
    1. All cells are reachable (no isolated areas)
    2. Number of connections equals (rows*cols - 1) (no loops)

    Returns:
        True if the maze is perfect, False otherwise
    """
    # BFS to check connectivity
    visited = [[False for _ in range(self.cols)] for _ in range(self.rows)]
    stack = [(0, 0)]  # Start from top-left corner
    visited[0][0] = True
    count = 1  # Count of visited cells

    # Implementation continues...
```

### 4. Use Cross-References

When using external tools like Sphinx:

```python
def solve_maze(maze: 'Maze', start: Tuple[int, int], end: Tuple[int, int]) -> Optional[List[Tuple[int, int]]]:
    """
    Find the shortest path in a maze using BFS algorithm.

    This function works with the :class:`maze_game.maze.Maze` class to find
    paths between points. It uses the wall information to determine valid moves.

    Args:
        maze: The :class:`maze_game.maze.Maze` instance to solve
        start: Starting position as (row, col)
        end: Ending position as (row, col)

    Returns:
        A list of (row, col) tuples representing the shortest path,
        or None if no path exists
    """
```

## Maintaining Documentation

### Documentation Updates

1. **Keep docs synchronized**: Update documentation when changing interfaces
2. **Use version control**: Track documentation changes with code changes
3. **Review process**: Include documentation in code reviews
4. **Automated checks**: Use tools to ensure docstring completeness

### Automated Documentation Quality

```bash
# Install pydocstyle to check docstring style
pip install pydocstyle

# Check documentation style
pydocstyle maze_game/
```

### Continuous Documentation

Set up CI/CD to automatically generate and deploy documentation:

```yaml
# .github/workflows/docs.yml
name: Documentation
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: 3.8
    - name: Install dependencies
      run: |
        pip install -r requirements.txt
        pip install pdoc3
    - name: Generate documentation  
      run: |
        make dvi
```

## Accessing Documentation

### Local Access

1. **Via Make**: `make dvi` generates and opens documentation
2. **Terminal access**: `python -m pydoc maze_game.maze.Maze`
3. **IDE integration**: Most IDEs display docstrings automatically

### Generated Documentation Structure

```
docs/
├── maze_game/
│   ├── index.html          # Main package documentation
│   ├── maze/
│   │   ├── index.html      # Maze submodule
│   │   └── maze.html       # Maze class documentation
│   ├── generator/
│   │   ├── index.html
│   │   └── generator.html
│   ├── solver/
│   │   ├── index.html  
│   │   └── solver.html
│   └── ... (other modules)
```

Proper documentation makes the codebase accessible to new developers, ensures maintainability, and serves as a reference for the project's functionality and architecture.