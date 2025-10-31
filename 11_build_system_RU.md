# Система сборки с Makefile

## Обзор

Система сборки автоматизирует компиляцию, установку, тестирование и распространение программного обеспечения. Для проектов на Python, таких как игра лабиринта, Make обычно используется для предоставления стандартных целей GNU, которые ожидают разработчики.

## Стандартные цели Make GNU

Makefile должен включать эти стандартные цели:

- `all`: Построить пакет
- `install`: Установить пакет
- `uninstall`: Удалить установленный пакет
- `clean`: Удалить сгенерированные файлы
- `dvi`: Создать документацию
- `dist`: Создать пакеты распространения
- `tests`: Запустить набор тестов

## Основная структура Makefile

```makefile
# Makefile для игры лабиринт
# Реализует стандартные цели для программ GNU

# Переменные
PYTHON = python3
PIP = pip3
INSTALL_DIR = /usr/local/bin
SRC_DIR = .
BUILD_DIR = build
DIST_DIR = dist
TEST_DIR = tests

# Цель по умолчанию
all: build

# Построить проект
build:
	@echo "Построение проекта игры лабиринт..."
	$(PYTHON) setup.py build

# Установить проект install:
	@echo "Установка игры лабиринт..."
	$(PYTHON) setup.py install
	@echo "Игра лабиринт успешно установлена."

# Удалить проект
uninstall:
	@echo "Удаление игры лабиринт..."
	pip uninstall -y maze_game || echo "Пакет может не быть установлен через pip"
	@echo "Игра лабиринт удалена."

# Очистить артефакты сборки
clean:
	@echo "Очистка артефактов сборки..."
	rm -rf $(BUILD_DIR) $(DIST_DIR) *.egg-info docs/ maze_game.html
	find . -type f -name "*.pyc" -delete
	find . -type d -name "__pycache__" -delete

# Создать документацию с автоматическим открытием в зависимости от ОС / Создать Документацию с Автоматическим Открытием в Зависимости от ОС
dvi:
	@echo "Создание документации..."
	# Проверить наличие инструментов документации и использовать доступный
	if command -v pdoc3 > /dev/null; then \
		pdoc3 --html --output-dir ./docs maze_game --force; \
		echo "Документация создана в ./docs/maze_game"; \
	elif command -v pydoc > /dev/null; then \
		pydoc -w maze_game; \
		echo "Документация создана в maze_game.html"; \
	else \
		echo "Не найдены ни pdoc3, ни pydoc. Установите один из них для создания документации."; \
	fi
	# Автоматически открыть документацию в зависимости от ОС
	if [ "$$(uname)" = "Darwin" ]; then \
		if [ -d "./docs/maze_game" ]; then \
			echo "Открытие документации в веб-браузере (macOS)..."; \
			open ./docs/maze_game/index.html; \
		elif [ -f "maze_game.html" ]; then \
			echo "Открытие документации в веб-браузере (macOS)..."; \
			open maze_game.html; \
		fi \
	elif [ "$$(expr substr $$(uname -s) 1 5)" = "Linux" ]; then \
		if [ -d "./docs/maze_game" ]; then \
			echo "Открытие документации в веб-браузере (Linux)..."; \
			xdg-open ./docs/maze_game/index.html 2>/dev/null || sensible-browser ./docs/maze_game/index.html 2>/dev/null || echo "Не удалось открыть браузер"; \
		elif [ -f "maze_game.html" ]; then \
			echo "Открытие документации в веб-браузере (Linux)..."; \
			xdg-open maze_game.html 2>/dev/null || sensible-browser maze_game.html 2>/dev/null || echo "Не удалось открыть браузер"; \
		fi \
	elif [ "$$(expr substr $$(uname -s) 1 10)" = "MINGW32_NT" ] || [ "$$(expr substr $$(uname -s) 1 10)" = "MINGW64_NT" ] || [ "$$(expr substr $$(uname -s) 1 7)" = "MSYS_NT" ]; then \
		if [ -d "./docs/maze_game" ]; then \
			echo "Открытие документации в веб-браузере (Windows)..."; \
			start ./docs/maze_game/index.html; \
		elif [ -f "maze_game.html" ]; then \
			echo "Открытие документации в веб-браузере (Windows)..."; \
			start maze_game.html; \
		fi \
	else \
		echo "Тип ОС не распознан. Документация создана, но не открыта автоматически. Проверьте docs/ или maze_game.html"; \
	fi

# Создать пакет распространения
dist:
	@echo "Создание пакета распространения..."
	$(PYTHON) setup.py sdist bdist_wheel

# Запустить тесты
tests:
	@echo "Запуск тестов..."
	$(PYTHON) -m unittest discover $(TEST_DIR) -v

# Запустить приложение
run:
	@echo "Запуск игры лабиринт..."
	$(PYTHON) -m maze_game

# Запустить веб-интерфейс
web:
	@echo "Запуск веб-версии игры лабиринт по адресу http://localhost:8080..."
	$(PYTHON) -m maze_game.web.app

# Создать пример лабиринта
sample:
	@echo "Создание примера лабиринта..."
	$(PYTHON) -c "from maze_game.generator.generator import generate_maze; m = generate_maze(10, 10); m.save_to_file('sample_maze.txt'); print('Пример лабиринта сохранен в sample_maze.txt')"

.PHONY: all build install uninstall clean dvi dist tests run sample web
```

## Подробные объяснения целей

### 1. Цель `all`

```makefile
all: build
```
Это цель по умолчанию, которая выполняется при запуске просто `make`. Она строит проект. / Это цель по умолчанию, которая выполняется при запуске просто `make`. Она строит проект.

### 2. Цель `build`

```makefile
build:
	@echo "Построение проекта игры лабиринт..."
	$(PYTHON) setup.py build
```
Это создает необходимые файлы сборки без установки пакета. Символ `@` подавляет вывод самой команды, показывая только эхо.

### 3. Цель `install`

```makefile
install:
	@echo "Установка игры лабиринт..."
	$(PYTHON) setup.py install
	@echo "Игра лабиринт успешно установлена."
```
Это устанавливает пакет в систему. Для Python `setup.py install` обрабатывает это.

### 4. Цель `uninstall`

```makefile
uninstall:
	@echo "Удаление игры лабиринт..."
	pip uninstall -y maze_game || echo "Пакет может не быть установлен через pip"
	@echo "Игра лабиринт удалена."
```
Это удаляет установленный пакет. Поскольку пакеты Python обычно управляются через pip, мы используем pip для удаления.

### 5. Цель `clean`

```makefile
clean:
	@echo "Очистка артефактов сборки..."
	rm -rf $(BUILD_DIR) $(DIST_DIR) *.egg-info docs/ maze_game.html
	find . -type f -name "*.pyc" -delete
	find . -type d -name "__pycache__" -delete
```
Это удаляет все сгенерированные файлы для очистки каталога проекта. / Это удаляет все сгенерированные файлы для очистки каталога проекта.

### 6. Цель `dvi` / Цель `dvi`

```makefile
dvi:
	@echo "Создание документации..."
	# Проверить наличие инструментов документации и использовать доступный
	if command -v pdoc3 > /dev/null; then \
		pdoc3 --html --output-dir ./docs maze_game --force; \
		echo "Документация создана в ./docs/maze_game"; \
	elif command -v pydoc > /dev/null; then \
		pydoc -w maze_game; \
		echo "Документация создана в maze_game.html"; \
	else \
		echo "Не найдены ни pdoc3, ни pydoc. Установите один из них для создания документации."; \
	fi
```
Это создает документацию в формате HTML с использованием доступных инструментов. / Это создает документацию в формате HTML с использованием доступных инструментов.

### 7. Цель `dist`

```makefile
dist:
	@echo "Создание пакета распространения..."
	$(PYTHON) setup.py sdist bdist_wheel
```
Это создает распространяемые пакеты (исходный дистрибутив и колесо).

### 8. Цель `tests`

```makefile
tests:
	@echo "Запуск тестов..."
	$(PYTHON) -m unittest discover $(TEST_DIR) -v
```
Это запускает все тесты в каталоге тестов с подробным выводом.

## Расширенные возможности Makefile

### Правила шаблонов

```makefile
# Компилировать все .py файлы в байт-код
%.pyc: %.py
	$(PYTHON) -m py_compile $<
```

### Автоматическая генерация зависимостей

Для Python зависимости обычно обрабатываются setup.py:

```makefile
# Установить зависимости перед сборкой
build: install-dependencies
	@echo "Построение проекта игры лабиринт..."
	$(PYTHON) setup.py build

install-dependencies:
	$(PIP) install -r requirements.txt
```

### Условная логика

```makefile
# Различное поведение для сборок отладки и релизов
ifdef DEBUG
    BUILD_ARGS = --debug
else
    BUILD_ARGS = --release
endif

build:
	$(PYTHON) setup.py build $(BUILD_ARGS)
```

## Интеграция с setup.py / Интеграция с setup.py

Makefile координирует с setup.py:

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
    author="Ваше имя",
    description="Приложение для генерации, решения и визуализации лабиринтов",
    python_requires='>=3.6',
)
```

## Лучшие практики / Лучшие Практики

### 1. Использование переменных / Использование Переменных

```makefile
PYTHON = python3
PIP = pip3
TEST_DIR = maze_game/tests
```
Это делает Makefile более поддерживаемым и настраиваемым.

### 2. Использование фиктивных целей

```makefile
.PHONY: all build install uninstall clean dvi dist tests run sample web
```
Это указывает Make, что эти цели не соответствуют реальным файлам.

### 3. Обработка ошибок

```makefile
# Тихая версия (показывать только эхо)
build:
	@echo "Построение..."
	@$(PYTHON) setup.py build

# Подробная версия (показывать команды тоже)
build:
	@echo "Построение..."
	$(PYTHON) setup.py build
```

### 4. Подавление или отображение

```makefile
# Тихая версия (только показывает эхо)
build:
	@echo "Построение..."
	@$(PYTHON) setup.py build

# Подробная версия (показывает команды тоже)
build:
	@echo "Построение..."
	$(PYTHON) setup.py build
```

## Соображения, специфичные для платформы

### Межплатформенные команды

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
		echo "Неподдерживаемая платформа для очистки"; \
	fi
```

## Интеграция с рабочим процессом разработки

Хорошо спроектированный Makefile становится центральной частью рабочего процесса разработки:

```bash
make                # Построить проект (цель по умолчанию)
make all           # Явно построить
make tests         # Запустить тесты
make install       # Установить локально
make clean         # Очистить артефакты сборки
make dist          # Создать пакеты распространения
make dvi           # Создать документацию
make run           # Запустить приложение
make web           # Запустить веб-интерфейс 
```

Makefile обеспечивает единый интерфейс для выполнения общих задач разработки, что упрощает работу разработчиков с проектом независимо от их платформы или настройки среды. / Makefile обеспечивает единый интерфейс для выполнения общих задач разработки, что упрощает работу разработчиков с проектом независимо от их платформы или настройки среды.