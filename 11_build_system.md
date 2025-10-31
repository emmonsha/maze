# Build System with Makefile

## Overview

A build system automates the compilation, installation, testing, and distribution of software. For Python projects like the Maze game, Make is commonly used to provide standard GNU targets that developers expect.

## GNU Make Standard Targets

The Makefile should include these standard targets:

- `all`: Build the package
- `install`: Install the package
- `uninstall`: Remove the installed package  
- `clean`: Remove generated files
- `dvi`: Generate documentation
- `dist`: Create distribution packages
- `tests`: Run test suite

## Basic Makefile Structure

```makefile
# Makefile for Maze Game
# Implements standard targets for GNU programs

# Variables
PYTHON = python3
PIP = pip3
INSTALL_DIR = /usr/local/bin
SRC_DIR = .
BUILD_DIR = build
DIST_DIR = dist
TEST_DIR = tests

# Default target
all: build

# Build the project
build:
	@echo "Building the Maze Game project..."
	$(PYTHON) setup.py build

# Install the project
install:
	@echo "Installing Maze Game..."
	$(PYTHON) setup.py install
	@echo "Maze Game installed successfully."

# Uninstall the project
uninstall:
	@echo "Uninstalling Maze Game..."
	pip uninstall -y maze_game || echo "Package may not be installed via pip"
	@echo "Maze Game uninstalled."

# Clean build artifacts
clean:
	@echo "Cleaning build artifacts..."
	rm -rf $(BUILD_DIR) $(DIST_DIR) *.egg-info
	find . -type f -name "*.pyc" -delete
	find . -type d -name "__pycache__" -delete

# Generate documentation with OS-specific opening
dvi:
	@echo "Generating documentation..."
	# Using pdoc to generate documentation if available
	if command -v pdoc3 > /dev/null; then \
		pdoc3 --html --output-dir ./docs maze_game --force; \
		echo "Documentation generated in ./docs/maze_game"; \
	else \
		if command -v pydoc > /dev/null; then \
			pydoc -w maze_game; \
			echo "Documentation generated in maze_game.html"; \
		else \
			echo "Neither pdoc3 nor pydoc found. Install one to generate documentation."; \
		fi \
	fi
	# Automatically open documentation based on OS
	if [ "$$(uname)" = "Darwin" ]; then \
		if [ -d "./docs/maze_game" ]; then \
			echo "Opening documentation in web browser (macOS)..."; \
			open ./docs/maze_game/index.html; \
		elif [ -f "maze_game.html" ]; then \
			echo "Opening documentation in web browser (macOS)..."; \
			open maze_game.html; \
		fi \
	elif [ "$$(expr substr $$(uname -s) 1 5)" = "Linux" ]; then \
		if [ -d "./docs/maze_game" ]; then \
			echo "Opening documentation in web browser (Linux)..."; \
			xdg-open ./docs/maze_game/index.html 2>/dev/null || sensible-browser ./docs/maze_game/index.html 2>/dev/null || echo "Could not open browser"; \
		elif [ -f "maze_game.html" ]; then \
			echo "Opening documentation in web browser (Linux)..."; \
			xdg-open maze_game.html 2>/dev/null || sensible-browser maze_game.html 2>/dev/null || echo "Could not open browser"; \
		fi \
	else \
		echo "OS type not recognized. Documentation generated but not opened automatically. Check docs/ or maze_game.html"; \
	fi

# Create distribution package
dist:
	@echo "Creating distribution package..."
	$(PYTHON) setup.py sdist bdist_wheel

# Run tests
tests:
	@echo "Running tests..."
	$(PYTHON) -m unittest discover $(TEST_DIR) -v

# Run the application
run:
	@echo "Starting Maze Game..."
	$(PYTHON) -m maze_game

# Start web interface
web:
	@echo "Starting Web version Maze Game on http://localhost:8080..."
	$(PYTHON) -m maze_game.web.app

# Generate a sample maze
sample:
	@echo "Generating a sample maze..."
	$(PYTHON) -c "from maze_game.generator.generator import generate_maze; m = generate_maze(10, 10); m.save_to_file('sample_maze.txt'); print('Sample maze saved to sample_maze.txt')"

.PHONY: all build install uninstall clean dvi dist tests run sample web
```

## Detailed Target Explanations

### 1. The `all` Target

```makefile
all: build
```
This is the default target that gets executed when you just run `make`. It builds the project.

### 2. The `build` Target

```makefile
build:
	@echo "Building the Maze Game project..."
	$(PYTHON) setup.py build
```
This creates the necessary build files without installing the package. The `@` symbol suppresses the command itself from being printed, only showing the echo.

### 3. The `install` Target

```makefile
install:
	@echo "Installing Maze Game..."
	$(PYTHON) setup.py install
	@echo "Maze Game installed successfully."
```
This installs the package to the system. For Python, `setup.py install` handles this.

### 4. The `uninstall` Target

```makefile
uninstall:
	@echo "Uninstalling Maze Game..."
	pip uninstall -y maze_game || echo "Package may not be installed via pip"
	@echo "Maze Game uninstalled."
```
This removes the installed package. Since Python packages are typically managed by pip, we use pip to uninstall.

### 5. The `clean` Target

```makefile
clean:
	@echo "Cleaning build artifacts..."
	rm -rf $(BUILD_DIR) $(DIST_DIR) *.egg-info
	find . -type f -name "*.pyc" -delete
	find . -type d -name "__pycache__" -delete
```
This removes all generated files to clean up the project directory.

### 6. The `dvi` Target

```makefile
dvi:
	@echo "Generating documentation..."
	# Check for documentation tools and use the available one
	if command -v pdoc3 > /dev/null; then \
		pdoc3 --html --output-dir ./docs maze_game --force; \
		echo "Documentation generated in ./docs/maze_game"; \
	elif command -v pydoc > /dev/null; then \
		pydoc -w maze_game; \
		echo "Documentation generated in maze_game.html"; \
	else \
		echo "Neither pdoc3 nor pydoc found. Install one to generate documentation."; \
	fi
```
This generates documentation in HTML format using available tools.

### 7. The `dist` Target

```makefile
dist:
	@echo "Creating distribution package..."
	$(PYTHON) setup.py sdist bdist_wheel
```
This creates distributable packages (source distribution and wheel).

### 8. The `tests` Target

```makefile
tests:
	@echo "Running tests..."
	$(PYTHON) -m unittest discover $(TEST_DIR) -v
```
This runs all tests in the test directory with verbose output.

## Advanced Makefile Features

### Pattern Rules

```makefile
# Compile all .py files to bytecode
%.pyc: %.py
	$(PYTHON) -m py_compile $<
```

### Automatic Dependency Generation

For Python, dependencies are typically handled by setup.py:

```makefile
# Install dependencies before building
build: install-dependencies
	@echo "Building the Maze Game project..."
	$(PYTHON) setup.py build

install-dependencies:
	$(PIP) install -r requirements.txt
```

### Conditional Logic

```makefile
# Different behavior for debug vs release builds
ifdef DEBUG
    BUILD_ARGS = --debug
else
    BUILD_ARGS = --release
endif

build:
	$(PYTHON) setup.py build $(BUILD_ARGS)
```

## Integration with setup.py

The Makefile coordinates with setup.py:

```python
# setup.py
from setuptools import setup, find_packages

setup(
    name="maze_game",
    version="1.0.0",
    packages=find_packages(),
    install_requires=[
        "pygame>=2.0.0",
        "flask>=2.0.0", 
        "numpy>=1.20.0",
    ],
    entry_points={
        'console_scripts': [
            'maze-game=maze_game.__main__:main',
        ],
    },
    author="Your Name",
    description="A maze generation, solving and visualization application",
    python_requires='>=3.6',
)
```

## Best Practices

### 1. Use Variables

```makefile
PYTHON = python3
PIP = pip3
TEST_DIR = maze_game/tests
```
This makes the Makefile more maintainable and configurable.

### 2. Use Phony Targets

```makefile
.PHONY: all build install uninstall clean dvi dist tests run sample web
```
This tells Make that these targets don't correspond to actual files.

### 3. Error Handling

```makefile
install:
	@echo "Installing Maze Game..."
	-$(PYTHON) setup.py install  # The - means ignore errors
```
Or handle errors explicitly:

```makefile
install:
	@echo "Installing Maze Game..."
	$(PYTHON) setup.py install || echo "Installation failed"
```

### 4. Verbose vs Silent

```makefile
# Silent version (only shows echo)
build:
	@echo "Building..."
	@$(PYTHON) setup.py build

# Verbose version (shows commands too)
build:
	@echo "Building..."
	$(PYTHON) setup.py build
```

## Platform-Specific Considerations

### Cross-Platform Commands

```makefile
clean:
	if [ "$$(uname)" = "Darwin" ]; then \
		rm -rf build dist *.egg-info; \
		find . -name "*.pyc" -delete; \
		find . -name "__pycache__" -type d -exec rm -rf {} +; \
	elif [ "$$(expr substr $$(uname -s) 1 5)" = "Linux" ]; then \
		rm -rf build dist *.egg-info; \
		find . -name "*.pyc" -delete; \
		find . -name "__pycache__" -type d -exec rm -rf {} +; \
	else \
		echo "Unsupported platform for cleaning"; \
	fi
```

## Integration with Development Workflow

A well-designed Makefile becomes a central part of the development workflow:

```bash
make                # Build the project (default target)
make all           # Explicitly build
make tests         # Run tests
make install       # Install locally
make clean         # Clean up build artifacts
make dist          # Create distribution packages
make dvi           # Generate documentation
make run           # Run the application
make web           # Start web interface
```

The Makefile provides a consistent interface for performing common development tasks, making it easier for developers to work with the project regardless of their platform or environment setup.