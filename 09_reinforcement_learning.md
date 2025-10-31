# Reinforcement Learning with Q-Learning

## Overview

Reinforcement Learning (RL) is a type of machine learning where an agent learns to make decisions by interacting with an environment to maximize cumulative reward. Q-Learning is a model-free RL algorithm that learns the value of actions in particular states.

## Key Concepts

### Agent-Environment Interaction
```
Agent -- Action --> Environment -- State & Reward --> Agent
```

### Q-Learning Fundamentals

- **State (s)**: The current situation (position in maze)
- **Action (a)**: The possible move (up, right, down, left)  
- **Reward (r)**: Feedback from environment (positive for goal, negative for each step)
- **Q-Value (Q(s,a))**: Expected future reward for taking action a in state s

### Q-Value Update Rule

```
Q(s, a) = Q(s, a) + α [r + γ max(Q(s', a')) - Q(s, a)]
```

Where:
- α (alpha) = learning rate
- γ (gamma) = discount factor  
- s' = next state
- max(Q(s', a')) = best expected future reward

## Q-Learning Agent Implementation

### Basic Agent Structure

```python
class QLearningAgent:
    def __init__(self, maze: Maze, end_pos: Tuple[int, int], 
                 learning_rate: float = 0.1, discount_factor: float = 0.9, 
                 exploration_rate: float = 0.1):
        self.maze = maze
        self.end_pos = end_pos
        self.learning_rate = learning_rate
        self.discount_factor = discount_factor
        self.exploration_rate = exploration_rate
        
        # Initialize Q-table: state -> {action -> value}
        # State is (row, col), action is direction (0: up, 1: right, 2: down, 3: left)
        self.q_table = self._initialize_q_table()
    
    def _initialize_q_table(self) -> Dict[Tuple[int, int], Dict[int, float]]:
        """Initialize Q-table with all state-action pairs set to 0."""
        q_table = {}
        for i in range(self.maze.rows):
            for j in range(self.maze.cols):
                # 4 possible actions: up(0), right(1), down(2), left(3)
                q_table[(i, j)] = {0: 0.0, 1: 0.0, 2: 0.0, 3: 0.0}
        return q_table
```

### Reward System

```python
def get_reward(self, pos: Tuple[int, int]) -> float:
    """Get the reward for being at a specific position."""
    if pos == self.end_pos:
        return 100  # High reward for reaching the goal
    else:
        return -1   # Small penalty for each step (encourages shortest path)
```

### Action Selection

```python
def get_possible_actions(self, pos: Tuple[int, int]) -> List[int]:
    """Get possible actions from a given position."""
    row, col = pos
    actions = []
    
    # Check if we can move in each direction
    if not self.maze.has_top_wall(row, col):  # Up
        actions.append(0)
    if not self.maze.has_right_wall(row, col):  # Right
        actions.append(1)
    if not self.maze.has_bottom_wall(row, col):  # Down
        actions.append(2)
    if not self.maze.has_left_wall(row, col):  # Left
        actions.append(3)
    
    return actions

def choose_action(self, pos: Tuple[int, int]) -> int:
    """Choose an action based on current Q-table and exploration rate."""
    possible_actions = self.get_possible_actions(pos)
    if not possible_actions:
        return 0  # Default if no actions possible
    
    # Epsilon-greedy exploration: explore with probability exploration_rate
    if random.random() < self.exploration_rate:
        # Explore: choose random action
        return random.choice(possible_actions)
    else:
        # Exploit: choose best known action
        return self._get_best_action(pos)

def _get_best_action(self, pos: Tuple[int, int]) -> int:
    """Get the action with the highest Q-value in the current position."""
    possible_actions = self.get_possible_actions(pos)
    best_action = possible_actions[0]
    best_value = self.q_table[pos][best_action]
    
    for action in possible_actions[1:]:
        if self.q_table[pos][action] > best_value:
            best_action = action
            best_value = self.q_table[pos][action]
    
    return best_action
```

### Q-Value Update

```python
def update_q_value(self, state: Tuple[int, int], action: int, 
                  reward: float, next_state: Tuple[int, int]):
    """Update the Q-value for a state-action pair."""
    current_q = self.q_table[state][action]
    
    # Calculate the maximum Q-value for next state
    possible_next_actions = self.get_possible_actions(next_state)
    if possible_next_actions:
        max_next_q = max(self.q_table[next_state][a] for a in possible_next_actions)
    else:
        max_next_q = 0  # Terminal state
    
    # Q-Learning update rule
    new_q = current_q + self.learning_rate * (
        reward + self.discount_factor * max_next_q - current_q
    )
    self.q_table[state][action] = new_q
```

### Training Process

```python
def train(self, episodes: int = 1000, max_steps_per_episode: int = 1000):
    """Train the agent using Q-learning."""
    for episode in range(episodes):
        # Start from a consistent position (or random - your choice)
        pos = (0, 0)  # Starting position
        
        for step in range(max_steps_per_episode):
            # Check if reached goal
            if pos == self.end_pos:
                break
            
            # Choose and take action
            action = self.choose_action(pos)
            new_pos = self._get_new_position(pos, action)
            
            # Get reward for the new position
            reward = self.get_reward(new_pos)
            
            # Update Q-value
            self.update_q_value(pos, action, reward, new_pos)
            
            # Move to next position
            pos = new_pos
```

### Navigation After Training

```python
def get_path(self, start_pos: Tuple[int, int]) -> List[Tuple[int, int]]:
    """Get the path from start position to end position using learned policy."""
    path = [start_pos]
    pos = start_pos
    
    max_path_length = self.maze.rows * self.maze.cols * 2  # Prevent infinite loops
    
    while pos != self.end_pos and len(path) < max_path_length:
        # Always choose best action (no exploration during navigation)
        possible_actions = self.get_possible_actions(pos)
        if not possible_actions:
            break  # No valid moves
        
        # Choose action with highest Q-value
        best_action = possible_actions[0]
        best_value = self.q_table[pos][best_action]
        
        for action in possible_actions[1:]:
            if self.q_table[pos][action] > best_value:
                best_action = action
                best_value = self.q_table[pos][action]
        
        # Get new position
        new_pos = self._get_new_position(pos, best_action)
        
        if new_pos != pos:  # Only add to path if move is valid
            path.append(new_pos)
            pos = new_pos
        else:
            # Something's wrong - avoid infinite loops
            break
    
    return path

def _get_new_position(self, pos: Tuple[int, int], action: int) -> Tuple[int, int]:
    """Get new position after taking an action."""
    row, col = pos
    
    if action == 0:  # Up
        return (max(0, row - 1), col)
    elif action == 1:  # Right
        return (row, min(self.maze.cols - 1, col + 1))
    elif action == 2:  # Down
        return (min(self.maze.rows - 1, row + 1), col)
    elif action == 3:  # Left
        return (row, max(0, col - 1))
    
    return pos  # Shouldn't happen
```

## Parameter Tuning

### Learning Rate (α)
- **High (0.7-1.0)**: Learning happens quickly but may be unstable
- **Low (0.1-0.3)**: Learning is stable but slow
- **Typical**: 0.1-0.2

### Discount Factor (γ)
- **High (0.9-0.99)**: Considers long-term rewards more
- **Low (0.1-0.5)**: Focuses on immediate rewards
- **Typical**: 0.9

### Exploration Rate (ε)
- **High initially**: Encourages exploration of environment
- **Decays over time**: Eventually exploits learned knowledge
- **Typical decay**: From 1.0 to 0.1 over training

## Advanced Techniques

### Epsilon Decay

```python
def train_with_decay(self, episodes: int = 1000):
    """Train with decaying exploration rate."""
    initial_epsilon = 1.0
    final_epsilon = 0.1
    decay_rate = (initial_epsilon - final_epsilon) / episodes
    
    for episode in range(episodes):
        self.exploration_rate = max(final_epsilon, 
                                   initial_epsilon - episode * decay_rate)
        
        # Training loop...
```

### Reward Shaping

Better reward functions can improve learning:

```python
def get_reward(self, pos: Tuple[int, int]) -> float:
    """More nuanced reward system."""
    if pos == self.end_pos:
        return 100  # Goal reached
    else:
        # Reward based on distance to goal (closer = better reward)
        distance_to_goal = abs(pos[0] - self.end_pos[0]) + abs(pos[1] - self.end_pos[1])
        return -1 - distance_to_goal * 0.01  # Encourage moving toward goal
```

## Integration with Maze Environment

### Maze Compatibility

The Q-learning agent must work with the existing maze structure:

```python
# The agent needs to understand maze walls
def get_possible_actions(self, pos: Tuple[int, int]) -> List[int]:
    row, col = pos
    
    actions = []
    if not self.maze.has_top_wall(row, col):  # Check wall before allowing move
        actions.append(0)  # Up
    if not self.maze.has_right_wall(row, col):
        actions.append(1)  # Right
    # ... and so on
```

### Visualization of Learning

```python
def visualize_learning(self, start_pos: Tuple[int, int]):
    """Visualize the agent's learned behavior."""
    path = self.get_path(start_pos)
    
    # This path can be displayed in the GUI or web interface
    # similar to how maze solving solutions are shown
```

## Performance Considerations

### Convergence
- The algorithm should converge to an optimal policy
- Monitor average reward over episodes to detect convergence
- May need thousands of episodes for complex mazes

### State Space
- Grid-based mazes have limited state space (rows × cols)
- Still, larger mazes require more training time

### Generalization
- Agent learns optimal path for specific maze
- Would need retraining for different maze structures
- Perfect for static environments (like fixed mazes)

## Common Challenges

### Exploration vs Exploitation
- Balance between exploring new paths and using known good paths
- Solution: Epsilon decay, or more advanced methods like UCB

### Convergence Issues
- May get stuck in local optima
- Solution: Ensure sufficient exploration and reward shaping

### Maze Complexity
- More complex mazes need longer training
- Consider maze topology during training

Q-Learning provides a powerful approach to maze solving that learns optimal paths through experience rather than explicit pathfinding algorithms.