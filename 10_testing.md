# Testing Strategy

## Overview

Testing ensures that all components of the maze project work correctly and maintain code quality. A comprehensive testing strategy includes unit tests, integration tests, and validation of complex algorithms.

## Python Testing Framework: unittest

### Basic Test Structure

```python
import unittest
from maze_game.maze.maze import Maze

class TestMaze(unittest.TestCase):
    """Test cases for the Maze class."""
    
    def test_maze_initialization(self):
        """Test that maze is properly initialized with walls."""
        maze = Maze(5, 5)
        
        # Check dimensions
        self.assertEqual(maze.rows, 5)
        self.assertEqual(maze.cols, 5)
        
        # Check that borders have walls
        # Top border
        for j in range(5):
            self.assertTrue(maze.horizontal_walls[0][j])

if __name__ == '__main__':
    unittest.main()
```

## Unit Testing Components

### Maze Class Tests

```python
class TestMaze(unittest.TestCase):
    def test_has_walls(self):
        """Test wall checking methods."""
        maze = Maze(3, 3)
        
        # Initially all internal walls should exist
        self.assertTrue(maze.has_right_wall(0, 0))
        self.assertTrue(maze.has_bottom_wall(0, 0))
        self.assertTrue(maze.has_left_wall(0, 0)) 
        self.assertTrue(maze.has_top_wall(0, 0))
        
        # Border walls should exist
        self.assertTrue(maze.has_top_wall(0, 0))  # Top border
        self.assertTrue(maze.has_left_wall(0, 0))  # Left border
        self.assertTrue(maze.has_right_wall(0, 2))  # Right border
        self.assertTrue(maze.has_bottom_wall(2, 0))  # Bottom border
    
    def test_file_io(self):
        """Test loading and saving maze to file."""
        import tempfile
        import os
        
        # Create a simple maze
        original_maze = Maze(2, 2)
        # Remove one wall to create a simple path
        original_maze.vertical_walls[0][1] = False
        
        # Create a temporary file
        with tempfile.NamedTemporaryFile(mode='w', delete=False, suffix='.txt') as f:
            temp_filename = f.name
        
        try:
            # Save maze to file
            original_maze.save_to_file(temp_filename)
            
            # Load maze from file
            loaded_maze = Maze(1, 1)  # Create dummy to use load method
            loaded_maze = loaded_maze.load_from_file(temp_filename)
            
            # Check that dimensions match
            self.assertEqual(loaded_maze.rows, original_maze.rows)
            self.assertEqual(loaded_maze.cols, original_maze.cols)
            
            # Check that walls match
            for i in range(loaded_maze.rows + 1):
                for j in range(loaded_maze.cols):
                    self.assertEqual(
                        loaded_maze.horizontal_walls[i][j],
                        original_maze.horizontal_walls[i][j]
                    )
            
            for i in range(loaded_maze.rows):
                for j in range(loaded_maze.cols + 1):
                    self.assertEqual(
                        loaded_maze.vertical_walls[i][j],
                        original_maze.vertical_walls[i][j]
                    )
        finally:
            # Clean up temporary file
            os.remove(temp_filename)
    
    def test_is_perfect(self):
        """Test the is_perfect method."""
        # Create a simple perfect maze (all cells connected, no loops)
        perfect_maze = Maze(2, 2)
        # Create a path that connects all cells
        perfect_maze.vertical_walls[0][1] = False  # (0,0) to (0,1)
        perfect_maze.horizontal_walls[1][1] = False  # (0,1) to (1,1) 
        perfect_maze.vertical_walls[1][1] = False  # (1,1) to (1,0)
        
        self.assertTrue(perfect_maze.is_perfect())
```

### Generator Tests

```python
class TestGenerator(unittest.TestCase):
    def test_generate_basic_maze(self):
        """Test generating a basic maze."""
        maze = generate_maze(5, 5)
        
        # Check dimensions
        self.assertEqual(maze.rows, 5)
        self.assertEqual(maze.cols, 5)
        
        # The maze should be perfect
        self.assertTrue(maze.is_perfect())
    
    def test_generate_different_sizes(self):
        """Test generating mazes of different sizes."""
        # Test small maze
        small_maze = generate_maze(2, 2)
        self.assertEqual(small_maze.rows, 2)
        self.assertEqual(small_maze.cols, 2)
        
        # Test larger maze
        large_maze = generate_maze(10, 8)
        self.assertEqual(large_maze.rows, 10)
        self.assertEqual(large_maze.cols, 8)
        
        # Both should be perfect
        self.assertTrue(small_maze.is_perfect())
        self.assertTrue(large_maze.is_perfect())
```

### Solver Tests

```python
class TestSolver(unittest.TestCase):
    def test_solve_simple_path(self):
        """Test solving a simple straight path."""
        maze = Maze(3, 3)
        # Create a simple path from (0,0) to (0,2)
        maze.vertical_walls[0][1] = False  # (0,0) to (0,1)
        maze.vertical_walls[0][2] = False  # (0,1) to (0,2)
        
        start = (0, 0)
        end = (0, 2)
        path = solve_maze(maze, start, end)
        
        self.assertIsNotNone(path)
        self.assertGreater(len(path), 0)
        self.assertEqual(path[0], start)
        self.assertEqual(path[-1], end)
    
    def test_solve_no_path(self):
        """Test solving when no path exists."""
        maze = Maze(3, 3)
        # Keep all walls, so no path exists from (0,0) to (2,2)
        
        start = (0, 0)
        end = (2, 2)
        path = solve_maze(maze, start, end)
        
        self.assertIsNone(path)
    
    def test_solve_same_start_end(self):
        """Test solving when start and end are the same."""
        maze = Maze(3, 3)
        start = (1, 1)
        end = (1, 1)
        path = solve_maze(maze, start, end)
        
        self.assertEqual(path, [start])
```

### Cave Tests

```python
class TestCave(unittest.TestCase):
    def test_cave_initialization(self):
        """Test that cave is properly initialized."""
        rows, cols = 10, 10
        cave = Cave(rows, cols, initial_chance=0.45)
        
        self.assertEqual(cave.rows, rows)
        self.assertEqual(cave.cols, cols)
    
    def test_cave_simulation_step(self):
        """Test one step of cave simulation."""
        cave = Cave(5, 5, initial_chance=0.5)
        initial_state = [row[:] for row in cave.grid]  # Copy grid
        
        changed = cave.step_simulation(birth_limit=4, death_limit=3)
        
        # The grid should have changed in most cases
        # (random initialization might create a stable state)
        self.assertIsInstance(changed, bool)
    
    def test_cave_simulation_run(self):
        """Test running multiple steps of cave simulation."""
        cave = Cave(10, 10, initial_chance=0.45)
        steps_performed = cave.run_simulation(birth_limit=4, death_limit=3, steps=5)
        
        self.assertGreater(steps_performed, 0)
        self.assertLessEqual(steps_performed, 5)
```

### Q-Learning Tests

```python
class TestQLearning(unittest.TestCase):
    def test_agent_initialization(self):
        """Test that Q-learning agent is properly initialized."""
        maze = Maze(3, 3)
        end_pos = (2, 2)
        agent = QLearningAgent(maze, end_pos)
        
        # Check that Q-table has correct structure
        self.assertIn((0, 0), agent.q_table)
        self.assertIn(0, agent.q_table[(0, 0)])  # Action 0 (up)
        self.assertIn(1, agent.q_table[(0, 0)])  # Action 1 (right)
        self.assertIn(2, agent.q_table[(0, 0)])  # Action 2 (down)
        self.assertIn(3, agent.q_table[(0, 0)])  # Action 3 (left)
    
    def test_get_reward(self):
        """Test reward function."""
        maze = Maze(3, 3)
        end_pos = (2, 2)
        agent = QLearningAgent(maze, end_pos)
        
        # Reward at end position should be high
        end_reward = agent.get_reward(end_pos)
        self.assertGreater(end_reward, 50)  # Should be 100
        
        # Other positions should have negative reward
        other_reward = agent.get_reward((0, 0))
        self.assertLess(other_reward, 0)  # Should be -1
    
    def test_get_possible_actions(self):
        """Test getting possible actions from a position."""
        maze = Maze(3, 3)
        # Add some walls to limit movement
        maze.horizontal_walls[1][0] = True  # Block moving down from (0,0)
        
        end_pos = (2, 2)
        agent = QLearningAgent(maze, end_pos)
        
        actions = agent.get_possible_actions((0, 0))
        # Should not include action 2 (down) due to wall
        self.assertNotIn(2, actions)
    
    def test_agent_training(self):
        """Test that agent can be trained without errors."""
        maze = Maze(3, 3)
        end_pos = (2, 2)
        agent = QLearningAgent(maze, end_pos)
        
        # Training should not raise exceptions
        try:
            agent.train(episodes=10)  # Small number for testing
        except Exception as e:
            self.fail(f"Training failed with exception: {e}")
```

## Testing Strategies

### Test-Driven Development (TDD)

```python
# Example: Write test first, then implementation
def test_new_feature(self):
    """Test for a feature that doesn't exist yet."""
    maze = Maze(5, 5)
    # We expect this method to exist and work properly
    complexity = maze.calculate_complexity()
    self.assertGreater(complexity, 0)
```

### Boundary Testing

```python
def test_boundary_conditions(self):
    """Test with boundary values."""
    # Test smallest possible maze
    small_maze = Maze(1, 1)
    self.assertEqual(small_maze.rows, 1)
    self.assertEqual(small_maze.cols, 1)
    
    # Test largest allowed maze
    large_maze = Maze(50, 50)  # Maximum size per requirements
    self.assertEqual(large_maze.rows, 50)
    self.assertEqual(large_maze.cols, 50)
```

### Edge Cases

```python
def test_edge_cases(self):
    """Test edge cases."""
    # Empty maze (not valid, but should handle gracefully)
    with self.assertRaises(Exception):  # Or whatever appropriate handling
        invalid_maze = Maze(0, 0)
    
    # Single cell maze
    single_cell = Maze(1, 1)
    self.assertTrue(single_cell.has_top_wall(0, 0))  # Border wall
```

## Running Tests

### Using unittest

```bash
# Run all tests
python -m unittest discover maze_game/tests -v

# Run specific test file
python -m unittest maze_game.tests.test_maze -v

# Run specific test class
python -m unittest maze_game.tests.test_maze.TestMaze -v

# Run single test method
python -m unittest maze_game.tests.test_maze.TestMaze.test_maze_initialization -v
```

### Using Makefile

```makefile
tests:
	@echo "Running tests..."
	$(PYTHON) -m unittest discover maze_game/tests -v
```

## Quality Metrics

### Test Coverage

While not using coverage tools in this project, good practices include:

- **100% of public methods** should have tests
- **Edge cases** should be covered
- **Error conditions** should be tested
- **Boundary values** should be tested

### Test Isolation

Each test should:
- Be independent of other tests
- Not rely on global state when possible
- Clean up after itself
- Have clear, descriptive names

## Integration Testing

Test how components work together:

```python
def test_maze_generation_to_solving(self):
    """Test generating a maze and then solving it."""
    maze = generate_maze(10, 10)
    
    # Should be solvable from (0,0) to (9,9)
    start = (0, 0)
    end = (maze.rows - 1, maze.cols - 1)
    path = solve_maze(maze, start, end)
    
    # Path should exist in a perfect maze
    self.assertIsNotNone(path)
    self.assertGreater(len(path), 0)
```

## Mocking and Stubbing

For complex dependencies, use mocking:

```python
from unittest.mock import Mock

def test_with_mock_dependencies(self):
    """Test with mocked dependencies."""
    # Create a mock maze with predetermined behavior
    mock_maze = Mock()
    mock_maze.rows = 5
    mock_maze.cols = 5
    mock_maze.has_right_wall.return_value = False  # Always allow right movement
    
    # Test solver behavior with mock maze
    path = solve_maze(mock_maze, (0, 0), (0, 4))
    # Verify solver works with mocked maze
```

A comprehensive testing strategy ensures that all components work as expected and provides confidence when making changes to the codebase.