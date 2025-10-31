# Web Interface with Flask

## Overview

The web interface provides a browser-based alternative to the desktop GUI, implementing the same functionality using Flask (Python web framework) and HTML/CSS/JavaScript. This allows the maze game to be accessed from any device with a web browser.

## Flask Fundamentals

### Flask Application Structure

```python
from flask import Flask, render_template, request, jsonify
from maze_game.maze.maze import Maze
from maze_game.solver.solver import solve_maze
from maze_game.generator.generator import generate_maze

def create_app():
    app = Flask(__name__)
    
    # Global state (in a real app, use sessions or databases)
    current_maze = generate_maze(10, 10)
    current_solution = None
    current_start = None
    current_end = None
    
    @app.route('/')
    def index():
        """Main page for the maze game."""
        return render_template('maze.html')
    
    # API endpoints for AJAX calls
    @app.route('/api/maze', methods=['GET'])
    def get_maze():
        # Return current maze as JSON
        return jsonify({
            'rows': current_maze.rows,
            'cols': current_maze.cols,
            'horizontal_walls': current_maze.horizontal_walls,
            'vertical_walls': current_maze.vertical_walls
        })
    
    return app

if __name__ == '__main__':
    app = create_app()
    app.run(debug=True, host='0.0.0.0', port=8080)
```

### Flask Routes and Endpoints

Flask uses decorators to define routes:

```python
@app.route('/api/maze', methods=['GET'])
def get_maze():
    """Get the current maze as JSON."""
    return jsonify({
        'rows': current_maze.rows,
        'cols': current_maze.cols,
        'horizontal_walls': current_maze.horizontal_walls,
        'vertical_walls': current_maze.vertical_walls
    })

@app.route('/api/maze', methods=['POST'])
def create_maze():
    """Generate a new maze with specified dimensions."""
    data = request.json
    rows = data.get('rows', 10)
    cols = data.get('cols', 10)
    
    # Validate size
    rows = min(rows, 50)
    cols = min(cols, 50)
    
    global current_maze
    current_maze = generate_maze(rows, cols)
    
    return jsonify({'status': 'success'})
```

## Frontend Implementation

### HTML Template Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Maze Game</title>
    <style>
        /* CSS styles */
        body { font-family: Arial, sans-serif; margin: 20px; }
        #maze-canvas { border: 1px solid #000; }
        .controls { display: inline-block; margin-left: 20px; }
    </style>
</head>
<body>
    <h1>Maze Game</h1>
    
    <div id="maze-container">
        <canvas id="maze-canvas" width="500" height="500"></canvas>
    </div>
    
    <div class="controls">
        <div class="control-group">
            <h3>Generate Maze</h3>
            <label for="rows">Rows:</label>
            <input type="number" id="rows" value="10" min="5" max="50">
            
            <label for="cols">Cols:</label>
            <input type="number" id="cols" value="10" min="5" max="50">
            
            <button onclick="generateMaze()">Generate New Maze</button>
        </div>
        <!-- More controls -->
    </div>

    <script>
        // JavaScript code
    </script>
</body>
</html>
```

### JavaScript AJAX Integration

```javascript
function loadMaze() {
    fetch('/api/maze', {
        method: 'GET',
        headers: {
            'Content-Type': 'application/json'
        }
    })
    .then(response => response.json())
    .then(data => {
        currentMaze = data;
        drawMaze();
        updateStatus('Maze loaded. Set start and end points to solve.');
    })
    .catch(error => {
        console.error('Error loading maze:', error);
        updateStatus('Error loading maze');
    });
}

function generateMaze() {
    const rows = document.getElementById('rows').value;
    const cols = document.getElementById('cols').value;
    
    fetch('/api/maze', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({rows: parseInt(rows), cols: parseInt(cols)})
    })
    .then(response => response.json())
    .then(data => {
        if (data.status === 'success') {
            loadMaze();
            solution = null;
            start = null;
            end = null;
            mode = 'none';
        }
    })
    .catch(error => {
        console.error('Error generating maze:', error);
        updateStatus('Error generating maze');
    });
}
```

### Canvas Drawing with JavaScript

```javascript
function drawMaze() {
    if (!currentMaze) return;
    
    const canvas = document.getElementById('maze-canvas');
    const ctx = canvas.getContext('2d');
    
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    const cellWidth = canvas.width / currentMaze.cols;
    const cellHeight = canvas.height / currentMaze.rows;
    
    // Draw cells and walls
    for (let row = 0; row < currentMaze.rows; row++) {
        for (let col = 0; col < currentMaze.cols; col++) {
            // Draw cell background
            ctx.fillStyle = '#ffffff';
            ctx.fillRect(col * cellWidth, row * cellHeight, cellWidth, cellHeight);
            
            // Draw right wall
            if (currentMaze.vertical_walls[row][col + 1]) {
                ctx.beginPath();
                ctx.moveTo((col + 1) * cellWidth, row * cellHeight);
                ctx.lineTo((col + 1) * cellWidth, (row + 1) * cellHeight);
                ctx.strokeStyle = '#000000';
                ctx.lineWidth = 2;
                ctx.stroke();
            }
            
            // Draw bottom wall
            if (currentMaze.horizontal_walls[row + 1][col]) {
                ctx.beginPath();
                ctx.moveTo(col * cellWidth, (row + 1) * cellHeight);
                ctx.lineTo((col + 1) * cellWidth, (row + 1) * cellHeight);
                ctx.strokeStyle = '#000000';
                ctx.lineWidth = 2;
                ctx.stroke();
            }
        }
    }
    
    // Draw borders, solution, start/end points
    drawSolution(ctx, cellWidth, cellHeight);
    drawPoints(ctx, cellWidth, cellHeight);
}

function drawSolution(ctx, cellWidth, cellHeight) {
    if (!solution) return;
    
    ctx.strokeStyle = '#ff0000'; // Red solution line
    ctx.lineWidth = 2;
    ctx.beginPath();
    
    for (let i = 0; i < solution.length; i++) {
        const [row, col] = solution[i];
        const x = col * cellWidth + cellWidth / 2;
        const y = row * cellHeight + cellHeight / 2;
        
        if (i === 0) {
            ctx.moveTo(x, y);
        } else {
            ctx.lineTo(x, y);
        }
    }
    ctx.stroke();
}
```

### Mouse Interaction

```javascript
const canvas = document.getElementById('maze-canvas');

canvas.addEventListener('click', function(e) {
    const rect = canvas.getBoundingClientRect();
    const x = e.clientX - rect.left;
    const y = e.clientY - rect.top;
    
    if (!currentMaze) {
        updateStatus('Please generate a maze first');
        return;
    }
    
    // Calculate which cell was clicked
    const cellWidth = canvas.width / currentMaze.cols;
    const cellHeight = canvas.height / currentMaze.rows;
    const col = Math.floor(x / cellWidth);
    const row = Math.floor(y / cellHeight);
    
    if (mode === 'start') {
        start = [row, col];
        mode = 'end';
        updateStatus(`Start set at (${row}, ${col}). Now click for end point.`);
        drawMaze();
    } else if (mode === 'end') {
        end = [row, col];
        mode = 'none';
        updateStatus(`End set at (${row}, ${col}). Solving maze...`);
        
        // Solve the maze
        solveMaze();
    }
});
```

## Advanced Features

### Cave Mode Implementation

```javascript
function generateCave() {
    const rows = document.getElementById('cave-rows').value;
    const cols = document.getElementById('cave-cols').value;
    const birthLimit = document.getElementById('birth-limit').value;
    const deathLimit = document.getElementById('death-limit').value;
    const initialChance = document.getElementById('initial-chance').value;
    
    fetch('/api/cave', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            rows: parseInt(rows), 
            cols: parseInt(cols),
            birth_limit: parseInt(birthLimit),
            death_limit: parseInt(deathLimit),
            initial_chance: parseFloat(initialChance),
            steps: 5
        })
    })
    .then(response => response.json())
    .then(data => {
        currentCave = data;
        currentMode = 'cave';
        drawCurrent(); // Switch to drawCave();
        updateStatus(`Cave generated: ${data.rows}x${data.cols}`);
    })
    .catch(error => {
        console.error('Error generating cave:', error);
        updateStatus('Error generating cave');
    });
}

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
            
            // Draw live (wall) cells in brown, dead (empty) cells in light gray
            if (currentCave.grid[row][col]) {
                ctx.fillStyle = '#8B4513'; // Brown
                ctx.fillRect(x, y, cellWidth, cellHeight);
            } else {
                ctx.fillStyle = '#f0f0f0'; // Light gray
                ctx.fillRect(x, y, cellWidth, cellHeight);
            }
            
            // Draw border for visibility
            ctx.strokeStyle = '#cccccc';
            ctx.lineWidth = 1;
            ctx.strokeRect(x, y, cellWidth, cellHeight);
        }
    }
    
    // Draw borders
    ctx.strokeRect(0, 0, canvas.width, canvas.height);
}
```

## Flask API Integration

### State Management

```python
def create_app():
    # Store the current state
    current_maze = generate_maze(10, 10)
    current_solution = None
    current_start = None
    current_end = None
    
    @app.route('/api/solve', methods=['POST'])
    def solve():
        """Solve the maze with given start and end points."""
        nonlocal current_maze, current_solution, current_start, current_end
        data = request.json
        start = tuple(data.get('start'))
        end = tuple(data.get('end'))
        
        current_start = start
        current_end = end
        
        current_solution = solve_maze(current_maze, start, end)
        
        return jsonify({
            'solution': current_solution,
            'start': current_start,
            'end': current_end
        })
    
    return app
```

### Template Rendering with Jinja2

Flask's templating engine (Jinja2) allows dynamic HTML generation:

```html
<!-- In maze.html -->
<div class="status" id="status">
    {% if current_maze %}
        Maze loaded with {{ current_maze.rows }} x {{ current_maze.cols }} dimensions
    {% else %}
        Click "Generate New Maze" to start
    {% endif %}
</div>
```

## Deployment Considerations

### Running the Server

```bash
# Direct run
python -m maze_game.web.app

# Using Make
make web
```

### Configuration for Production

```python
def create_app():
    app = Flask(__name__)
    
    # Configuration for production
    if not app.debug:
        app.config['SECRET_KEY'] = os.environ.get('SECRET_KEY')
    
    # ... routes
    return app
```

## Common Challenges and Solutions

### 1. State Management
- **Problem**: Web requests are stateless
- **Solution**: Use server-side session storage or global variables for development

### 2. Real-time Updates
- **Problem**: Need to update display without page reload
- **Solution**: Use AJAX for API calls and JavaScript for dynamic updates

### 3. Cross-Origin Issues
- **Problem**: Frontend and backend may be on different ports
- **Solution**: Proper CORS headers (not needed when served from same Flask app)

### 4. File Upload
- **Problem**: Loading maze files through web interface
- **Solution**: Use HTML file input with XMLHttpRequest or fetch API

## Best Practices

1. **Separation of Concerns**: Keep Flask, HTML, CSS, and JavaScript organized
2. **Error Handling**: Provide clear error messages for API failures
3. **Security**: Validate all input on the server side
4. **Performance**: Optimize drawing and avoid unnecessary API calls
5. **User Experience**: Provide loading indicators and clear feedback

Flask provides an excellent platform for creating web interfaces that can complement desktop applications, offering accessibility and platform independence.