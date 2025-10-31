# Настройка среды

## Обзор

Перед началом проекта лабиринта вам понадобится:

- **Python 3.6+**: Основной язык программирования
- **pip**: Установщик пакетов Python
- **Git**: Для контроля версий (если используется репозиторий)

## Установка зависимостей

### 1. Создать виртуальную среду (Рекомендуется)

```bash
python3 -m venv venv
source venv/bin/activate  # В Windows: venv\Scripts\activate
```

### 2. Установить требуемые пакеты

Создать файл requirements.txt со следующим содержимым:

```txt
pygame>=2.0.0
flask>=2.0.0
numpy>=1.20.0
```

Затем установить:

```bash
pip install -r requirements.txt
```

### 3. Проверить установку

```bash
python -c "import pygame; print('Версия Pygame:', pygame.version.ver)"
python -c "import flask; print('Версия Flask:', flask.__version__)"
python -c "import numpy; print('Версия NumPy:', numpy.__version__)"
```

## Настройка структуры проекта

Создать структуру каталогов проекта:

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

## Тестирование настройки

Создать простой тестовый файл для проверки работы:

```python
# test_setup.py
def test_imports():
    try:
        import pygame
        import flask
        import numpy
        print("✓ Все требуемые библиотеки успешно импортированы")
        print(f"  Pygame: {pygame.version.ver}")
        print(f"  Flask: {flask.__version__}")
        print(f"  NumPy: {numpy.__version__}")
    except ImportError as e:
        print(f"✗ Ошибка импорта: {e}")

if __name__ == "__main__":
    test_imports()
```

Запустить: `python test_setup.py`

## Устранение неполадок

### Проблемы с установкой Pygame

На некоторых системах могут потребоваться дополнительные пакеты:

```bash
# Для macOS
brew install sdl2 sdl2_image sdl2_mixer sdl2_ttf

# Для Ubuntu/Debian
sudo apt-get install libsdl2-dev libsdl2-image-dev libsdl2-mixer-dev libsdl2-ttf-dev
```

### Ошибка импорта Flask

Убедитесь, что у вас последняя версия pip:

```bash
pip install --upgrade pip
pip install flask
```

## Настройка Makefile

Создать базовый Makefile для управления проектом:

```makefile
# Makefile для проекта лабиринта
PYTHON = python3
PIP = pip3

install:
	@echo "Установка зависимостей проекта лабиринта..."
	$(PIP) install -r requirements.txt

run:
	@echo "Запуск игры лабиринт..."
	$(PYTHON) -m maze_game

web:
	@echo "Запуск веб-интерфейса..."
	$(PYTHON) -m maze_game.web.app

clean:
	rm -rf __pycache__
	find . -type f -name "*.pyc" -delete

.PHONY: install run web clean
```

С этой настройкой вы готовы начать создание проекта лабиринта!