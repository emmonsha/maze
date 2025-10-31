# Setting Up the Environment

## Prerequisites

Before starting the Maze project, you'll need:

- **Python 3.6+**: The project is developed in Python
- **pip**: Python package installer
- **Git**: For version control (if using repository)

## Installing Dependencies

### 1. Create Virtual Environment (Recommended)

```bash
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

### 2. Install Required Packages

Create a requirements.txt file with the following:

```txt
pygame>=2.0.0
flask>=2.0.0
numpy>=1.20.0
```

Then install:

```bash
pip install -r requirements.txt
```

### 3. Verify Installation

```bash
python -c "import pygame; print('Pygame version:', pygame.version.ver)"
python -c "import flask; print('Flask version:', flask.__version__)"
python -c "import numpy; print('NumPy version:', numpy.__version__)"
```

## Project Structure Setup

Create the project directory structure:

```
maze_project/
├── src/
│   └── maze_game/
│       ├── __init__.py
│       ├── __main__.py
│       ├── maze/
│       │   ├── __init__.py
│       │   └── maze.py
│       ├── generator/
│       │   ├── __init__.py
│       │   └── generator.py
│       ├── solver/
│       │   ├── __init__.py
│       │   └── solver.py
│       ├── gui/
│       │   ├── __init__.py
│       │   └── gui.py
│       ├── web/
│       │   ├── __init__.py
│       │   ├── app.py
│       │   └── templates/
│       │       └── maze.html
│       ├── cave/
│       │   ├── __init__.py
│       │   └── cave.py
│       ├── reinforcement_learning/
│       │   ├── __init__.py
│       │   └── q_learning.py
│       └── tests/
│           ├── __init__.py
│           ├── test_maze.py
│           ├── test_generator.py
│           ├── test_solver.py
│           ├── test_cave.py
│           └── test_q_learning.py
├── requirements.txt
├── setup.py
└── Makefile
```

## Testing the Setup

Create a simple test file to verify everything works:

```python
# test_setup.py
def test_imports():
    try:
        import pygame
        import flask
        import numpy
        print("✓ All required libraries imported successfully")
        print(f"  Pygame: {pygame.version.ver}")
        print(f"  Flask: {flask.__version__}")
        print(f"  NumPy: {numpy.__version__}")
    except ImportError as e:
        print(f"✗ Import error: {e}")

if __name__ == "__main__":
    test_imports()
```

Run: `python test_setup.py`

## Troubleshooting

### Pygame Installation Issues
On some systems, you might need additional packages:
```bash
# For macOS
brew install sdl2 sdl2_image sdl2_mixer sdl2_ttf

# For Ubuntu/Debian
sudo apt-get install libsdl2-dev libsdl2-image-dev libsdl2-mixer-dev libsdl2-ttf-dev
```

### Flask Import Error
Make sure you have the latest pip:
```bash
pip install --upgrade pip
pip install flask
```

## Setting up the Makefile

Create a basic Makefile to manage the project:

```makefile
# Makefile for Maze Project
PYTHON = python3
PIP = pip3

install:
	@echo "Installing Maze Project dependencies..."
	$(PIP) install -r requirements.txt

run:
	@echo "Starting Maze Game..."
	$(PYTHON) -m maze_game

web:
	@echo "Starting Web Interface..."
	$(PYTHON) -m maze_game.web.app

clean:
	rm -rf __pycache__
	find . -type f -name "*.pyc" -delete

.PHONY: install run web clean
```

With this setup, you're ready to start building the Maze project!