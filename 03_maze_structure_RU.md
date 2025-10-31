# Структура данных лабиринта

## Понимание представления лабиринта

Лабиринт - это 2D сетка, где каждая ячейка может иметь стены на своих границах. Ключ к представлению лабиринта программно - это отслеживание того, какие стены существуют между соседними ячейками.

### Подход "тонкой стены" / Подход "Тонкой Стены"

В "тонком стенном" представлении:
- Каждая ячейка - это позиция в сетке
- Стены существуют между ячейками (не внутри них)
- Лабиринт окружен стеной

```text
+---+---+---+
|   |   |   |  <- Верхние стены границ
+   +   +   +  <- Горизонтальные внутренние стены
|   |   |   |
+   +   +   +  <- Горизонтальные внутренние стены
|   |   |   |
+---+---+---+  <- Нижние стены границ
```

### Хранение стен

Мы храним стены, используя две матрицы:
- **Вертикальные стены**: матрица, указывающая стены справа от каждой ячейки
- **Горизонтальные стены**: матрица, указывающая стены ниже каждой ячейки

Для лабиринта n×m:
- Матрица вертикальных стен: n строк × (m+1) столбцов
- Матрица горизонтальных стен: (n+1) строк × m столбцов

## Реализация класса Maze

### Базовая структура

```python
class Maze:
    def __init__(self, rows: int, cols: int):
        self.rows = rows
        self.cols = cols
        # Вертикальные стены: стены справа от каждой ячейки (включая границы) / Вертикальные Стены: Стены Справа от Каждой Ячейки (Включая Границы)
        self.vertical_walls = [[True for _ in range(cols + 1)] for _ in range(rows)]
        # Горизонтальные стены: стены ниже каждой ячейки (включая границы) / Горизонтальные Стены: Стены Ниже Каждой Ячейки (Включая Границы)  
        self.horizontal_walls = [[True for _ in range(cols)] for _ in range(rows + 1)]
        
        # Установить стены границ / Установить Стены Границ
        self._set_border_walls()
```

### Методы доступа к стенам

```python
def has_right_wall(self, row: int, col: int) -> bool:
    """Проверить стену справа от ячейки (row, col)"""
    if 0 <= row < self.rows and 0 <= col < self.cols:
        return self.vertical_walls[row][col + 1]
    return True  # Стена границы / Стена Границы

def has_bottom_wall(self, row: int, col: int) -> bool:
    """Проверить стену ниже ячейки (row, col)"""
    if 0 <= row < self.rows and 0 <= col < self.cols:
        return self.horizontal_walls[row + 1][col]
    return True  # Стена границы / Стена Границы

def has_left_wall(self, row: int, col: int) -> bool:
    """Проверить стену слева от ячейки (row, col)"""
    if 0 <= row < self.rows and 0 <= col < self.cols:
        return self.vertical_walls[row][col]
    return True  # Стена границы / Стена Границы

def has_top_wall(self, row: int, col: int) -> bool:
    """Проверить стену выше ячейки (row, col)"""
    if 0 <= row < self.rows and 0 <= col < self.cols:
        return self.horizontal_walls[row][col]
    return True  # Стена границы / Стена Границы
```

## Формат файла

Формат файла лабиринта структурирован как:
```
rows cols
vertical_walls_row_0
vertical_walls_row_1
...
vertical_walls_row_(n-1)
horizontal_walls_row_0
horizontal_walls_row_1
...
horizontal_walls_row_n
```

### Реализация файлового ввода-вывода

```python
def save_to_file(self, filename: str):
    with open(filename, 'w') as f:
        # Записать размеры
        f.write(f"{self.rows} {self.cols}\n\n")
        
        # Записать вертикальные стены
        for i in range(self.rows):
            row = [int(self.vertical_walls[i][j]) for j in range(self.cols + 1)]
            f.write(" ".join(map(str, row)) + "\n")
        f.write("\n")
        
        # Записать горизонтальные стены
        for i in range(self.rows + 1):
            row = [int(self.horizontal_walls[i][j]) for j in range(self.cols)]
            f.write(" ".join(map(str, row)) + "\n")

def load_from_file(self, filename: str) -> 'Maze':
    with open(filename, 'r') as f:
        lines = [line.strip() for line in f if line.strip()]
    
    # Первая строка имеет ряды и столбцы
    rows, cols = map(int, lines[0].split())
    maze = Maze(rows, cols)
    
    # Следующие 'rows' строк представляют вертикальные стены
    idx = 1
    for i in range(rows):
        wall_row = list(map(int, lines[idx].split()))
        for j in range(cols + 1):
            maze.vertical_walls[i][j] = bool(wall_row[j]) if j < len(wall_row) else True
        idx += 1
    
    # Следующие 'rows+1' строк представляют горизонтальные стены
    for i in range(rows + 1):
        wall_row = list(map(int, lines[idx].split()))
        for j in range(cols):
            maze.horizontal_walls[i][j] = bool(wall_row[j]) if j < len(wall_row) else True
        idx += 1
    
    return maze
```

## Проверка совершенного лабиринта

Совершенный лабиринт имеет ровно один путь между любыми двумя точками (без петель и изолированных областей). Мы можем проверить это:

```python
def is_perfect(self) -> bool:
    # 1. Проверить связность (все ячейки достижимы из старта)
    visited = [[False for _ in range(self.cols)] for _ in range(self.rows)]
    stack = [(0, 0)]
    visited[0][0] = True
    count = 1

    directions = [(0, 1), (1, 0), (0, -1), (-1, 0)]
    
    while stack:
        row, col = stack.pop()
        for dr, dc in directions:
            new_row, new_col = row + dr, col + dc
            if (0 <= new_row < self.rows and 0 <= new_col < self.cols and 
                not visited[new_row][new_col]):
                
                # Проверить, можем ли мы переместиться в соседнюю ячейку (без стены)
                can_move = self._can_move(row, col, new_row, new_col)
                if can_move:
                    visited[new_row][new_col] = True
                    stack.append((new_row, new_col))
                    count += 1

    # Все ячейки должны быть достижимы
    if count != self.rows * self.cols:
        return False

    # 2. Проверить правильное количество соединений (n*m - 1 для совершенного лабиринта)
    path_count = self._count_paths()
    return path_count == (self.rows * self.cols - 1)
```

## Ключевые концепции

1. **Модульные стены**: Стены хранятся отдельно от ячеек
2. **Обработка границ**: Стены границ всегда присутствуют
3. **Гибкость**: Структура позволяет легко модифицировать стены
4. **Масштабируемость**: Работает для любого разумного размера лабиринта (≤50x50 по требованиям)
5. **Сохраняемость**: Легко сохранять/загружать с простым файловым форматом 

Эта структура обеспечивает прочный фундамент для алгоритмов генерации, решения и визуализации лабиринтов.