# Разработка графического интерфейса

## Обзор

Графический интерфейс обеспечивает интерактивный интерфейс для генерации, визуализации и решения лабиринтов. Pygame выбран для его простоты в обработке 2D графики, пользовательского ввода и игровых циклов.

## Основы Pygame

### Основные компоненты

1. **Поверхность дисплея**: Окно/холст, где всё рисуется
2. **Цикл событий**: Обрабатывает пользовательский ввод (мышь, клавиатура)
3. **Игровой цикл**: Обновляет состояние и перерисовывает экран
4. **Примитивы рисования**: Функции для рисования фигур, текста и изображений

### Базовая настройка Pygame

```python
import pygame
import sys

def main():
    pygame.init()
    
    # Создать дисплей
    screen = pygame.display.set_mode((500, 500))
    pygame.display.set_caption("Игра лабиринт")
    
    clock = pygame.time.Clock()
    
    # Основной игровой цикл
    running = True
    while running:
        # Обрабатывать события
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
        
        # Обновить состояние игры
        
        # Нарисовать всё / Нарисовать Всё
        screen.fill((255, 255, 255))  # Белый фон
        
        # Нарисовать игровые элементы здесь
        
        pygame.display.flip()  # Обновить дисплей
        clock.tick(60)  # Ограничить 60 FPS
    
    pygame.quit()
    sys.exit()
```

## Реализация отображения лабиринта

### Система координат и масштабирования

Каждая ячейка лабиринта должна быть нарисована, чтобы поместиться в область дисплея 500×500 пикселей:

```python
def calculate_cell_size(self) -> Tuple[int, int]:
    """Рассчитать размер каждой ячейки, чтобы заполнить область дисплея."""
    if self.maze is None:
        return 10, 10  # Размер по умолчанию, если нет лабиринта
        
    # Учесть толщину стены
    available_width = self.WINDOW_WIDTH - self.WALL_THICKNESS
    available_height = self.WINDOW_HEIGHT - self.WALL_THICKNESS
    
    cell_width = available_width // self.maze.cols
    cell_height = available_height // self.maze.rows
    
    # Использовать меньшее из двух, чтобы убедиться, что лабиринт помещается
    cell_size = min(cell_width, cell_height)
    
    return cell_size, cell_size
```

### Рисование лабиринта

```python
def draw_maze(self):
    """Нарисовать лабиринт на экране."""
    if self.maze is None:
        return
        
    cell_width, cell_height = self.calculate_cell_size()
    self.screen.fill(self.BG_COLOR)  # Белый фон
    
    # Нарисовать стены и ячейки
    for row in range(self.maze.rows):
        for col in range(self.maze.cols):
            x = col * cell_width
            y = row * cell_height
            
            # Нарисовать правую стену, если она существует
            if self.maze.has_right_wall(row, col):
                pygame.draw.line(
                    self.screen, 
                    self.WALL_COLOR,
                    (x + cell_width, y),
                    (x + cell_width, y + cell_height),
                    self.WALL_THICKNESS
                )
            
            # Нарисовать нижнюю стену, если она существует
            if self.maze.has_bottom_wall(row, col):
                pygame.draw.line(
                    self.screen, 
                    self.WALL_COLOR,
                    (x, y + cell_height),
                    (x + cell_width, y + cell_height),
                    self.WALL_THICKNESS
                )
    
    # Нарисовать границы
    pygame.draw.rect(self.screen, self.WALL_COLOR, 
                     (0, 0, self.WINDOW_WIDTH, self.WINDOW_HEIGHT), 
                     self.WALL_THICKNESS)
    
    # Нарисовать путь решения, если он существует
    self.draw_solution()
    
    # Нарисовать начальную/конечную точки, если установлены
    self.draw_points()
```

## Обработка пользовательского ввода

### События мыши

```python
def handle_click(self, pos: Tuple[int, int]):
    """Обрабатывать события щелчков мыши."""
    if self.maze is None:
        return
        
    cell_width, cell_height = self.calculate_cell_size()
    
    # Преобразовать координаты пикселей в координаты лабиринта
    col = pos[0] // cell_width
    row = pos[1] // cell_height
    
    # Убедиться, что щелчок находится в пределах лабиринта
    if 0 <= row < self.maze.rows and 0 <= col < self.maze.cols:
        if self.state == "select_start":
            self.start_pos = (row, col)
            self.state = "select_end"
        elif self.state == "select_end":
            self.end_pos = (row, col)
            if self.start_pos and self.end_pos:
                # Решить лабиринт
                self.solution = solve_maze(self.maze, self.start_pos, self.end_pos)
            self.state = "display"
```

### События клавиатуры / События Клавиатуры

```python
def run(self):
    running = True
    
    while running:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_s:  # Выбрать начало
                    self.state = "select_start"
                elif event.key == pygame.K_e:  # Выбрать конец
                    self.state = "select_end"
                elif event.key == pygame.K_g:  # Сгенерировать новый лабиринт
                    self.maze = generate_maze(10, 10)
                    self.reset_solution()
    
    # ... остальная реализация
```

## Управление состоянием

Графический интерфейс должен управлять различными состояниями:

```python
class MazeGUI:
    def __init__(self, maze: Optional[Maze] = None):
        # ... инициализация
        
        # Для пользовательского интерфейса
        self.state = "display"  # "display", "select_start", "select_end"
        self.mode = "maze"      # "maze", "cave"
        
    def reset_solution(self):
        self.solution = None
        self.start_pos = None
        self.end_pos = None
        self.state = "display"
```

## Расширенные функции графического интерфейса

### Интеграция диалога файлов

```python
import tkinter as tk
from tkinter import filedialog

# Скрыть главное окно tkinter
tk.Tk().withdraw()

def load_maze_from_file(self):
    file_path = filedialog.askopenfilename(
        title="Загрузить файл лабиринта",
        filetypes=[("Текстовые файлы", "*.txt"), ("Все файлы", "*.*")]
    )
    if file_path:
        try:
            # Загрузить лабиринт из файла
            temp_maze = Maze(1, 1)
            self.maze = temp_maze.load_from_file(file_path)
            self.reset_solution()
        except Exception as e:
            print(f"Ошибка загрузки лабиринта из файла: {e}")
```

### Система занавеса

```python
def draw_curtain(self):
    """Нарисовать полупрозрачный занавес поверх информации."""
    # Создать полупрозрачную поверхность
    curtain = pygame.Surface((self.WINDOW_WIDTH, self.WINDOW_HEIGHT))
    curtain.set_alpha(180)  # Настроить прозрачность
    curtain.fill((200, 200, 200))  # Светло-серый занавес
    self.screen.blit(curtain, (0, 0))
    
    # Визуализировать центрированный текст
    lines = [
        "Игра лабиринт - Помощь",
        "",
        "S - Выбрать начальную точку",  # Выбрать начальную точку
        "E - Выбрать конечную точку",    # Выбрать конечную точку
        "G - Сгенерировать новый лабиринт",  # Сгенерировать новый лабиринт
        "L - Загрузить лабиринт из файла",   # Загрузить лабиринт из файла
        "C - Очистить решение",             # Очистить решение
        "H - Помощь (этот экран)"           # Помощь (этот экран)
    ]
    
    line_height = 30
    total_height = len(lines) * line_height
    start_y = (self.WINDOW_HEIGHT - total_height) // 2
    
    for i, line in enumerate(lines):
        text_surface = self.font.render(line, True, (0, 0, 0))
        text_rect = text_surface.get_rect(center=(self.WINDOW_WIDTH // 2, start_y + i * line_height))
        self.screen.blit(text_surface, text_rect)
```

## Распространенные проблемы и решения

### 1. Проблемы производительности
- **Проблема**: Рисование больших лабиринтов может быть медленным
- **Решение**: Перерисовывать только измененные области или использовать грязные прямоугольники pygame

### 2. Преобразование координат
- **Проблема**: Преобразование между координатами пикселей и лабиринта
- **Решение**: Создать вспомогательные функции с четкими именами переменных

### 3. Масштабируемость
- **Проблема**: Разные размеры лабиринтов требуют разных размеров ячеек
- **Решение**: Динамическое вычисление на основе доступного пространства дисплея

### 4. Пользовательский опыт
- **Проблема**: Обеспечение четкой обратной связи и инструкций
- **Решение**: Четкие сообщения пользовательского интерфейса и визуальные индикаторы

## Лучшие практики

1. **Разделение ответственности**: Держать логику рисования отдельно от игровой логики
2. **Производительность**: Использовать соответствующие методы рисования для больших лабиринтов
3. **Обратная связь пользователя**: Предоставлять визуальную и текстовую обратную связь для всех действий
4. **Обработка ошибок**: Изящно обрабатывать недействительные операции
5. **Поддерживаемость**: Использовать четкие имена переменных и модульные функции

## Интеграция с другими компонентами

Графический интерфейс должен интегрироваться с:
- Генерацией лабиринта (для отображения сгенерированных лабиринтов)
- Решением лабиринта (для визуализации решений)
- Файловым вводом-выводом (для операций загрузки/сохранения)
- Другими алгоритмами (генерация пещер и т.д.)

Pygame обеспечивает отличный фундамент для создания интерактивных, визуальных приложений, таких как игра лабиринт, балансируя простоту с мощью для 2D графики и обработки ввода.