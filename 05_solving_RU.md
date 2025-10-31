# Решение лабиринтов

## Обзор

Решение лабиринта включает нахождение пути от начальной точки до конечной точки. Наиболее распространенный подход для нахождения кратчайшего пути в невзвешенном графе, как лабиринт, - это поиск в ширину (BFS).

## Алгоритм поиска в ширину (BFS)

### Почему BFS?

BFS идеально подходит для решения лабиринтов, потому что:
- Он находит кратчайший путь (минимальное количество шагов)
- Он исследует все узлы на текущей глубине перед переходом к следующей
- Он гарантированно находит решение, если оно существует
- Он прост в реализации и понимании

### Шаги алгоритма

1. **Инициализация**: Добавить начальную позицию в очередь, отметить как посещенную
2. **Обработка**: Взять первую позицию из очереди
3. **Проверка**: Если это конец, мы нашли решение
4. **Исследование**: Добавить всех непосещенных соседей в очередь
5. **Повторение**: Пока очередь не пуста или решение не найдено

### Реализация

```python
from collections import deque

def solve_maze(maze: Maze, start: Tuple[int, int], end: Tuple[int, int]) -> Optional[List[Tuple[int, int]]]:
    start_row, start_col = start
    end_row, end_col = end

    # Проверить начальную и конечную точки
    if not (0 <= start_row < maze.rows and 0 <= start_col < maze.cols):
        return None
    if not (0 <= end_row < maze.rows and 0 <= end_col < maze.cols):
        return None

    # Если начало и конец совпадают
    if start == end:
        return [start]

    # Настройка BFS
    queue = deque([start])
    visited = [[False for _ in range(maze.cols)] for _ in range(maze.rows)]
    visited[start_row][start_col] = True
    predecessors = {}  # Для реконструкции пути

    # Направления: вправо, вниз, влево, вверх
    directions = [(0, 1), (1, 0), (0, -1), (-1, 0)]

    while queue:
        curr_row, curr_col = queue.popleft()

        # Проверить, достигли ли мы пункта назначения
        if (curr_row, curr_col) == end:
            break

        # Исследовать соседей
        for dr, dc in directions:
            new_row, new_col = curr_row + dr, curr_col + dc

            # Проверить, что новая позиция действительна и не посещена
            if (0 <= new_row < maze.rows and 0 <= new_col < maze.cols and
                    not visited[new_row][new_col]):

                # Проверить, можем ли мы переместиться в этом направлении (без стены)
                can_move = False
                if dr == 0 and dc == 1:  # Движение вправо
                    can_move = not maze.has_right_wall(curr_row, curr_col)
                elif dr == 1 and dc == 0:  # Движение вниз
                    can_move = not maze.has_bottom_wall(curr_row, curr_col)
                elif dr == 0 and dc == -1:  # Движение влево
                    can_move = not maze.has_left_wall(curr_row, curr_col)
                elif dr == -1 and dc == 0:  # Движение вверх
                    can_move = not maze.has_top_wall(curr_row, curr_col)

                if can_move:
                    visited[new_row][new_col] = True
                    queue.append((new_row, new_col))
                    predecessors[(new_row, new_col)] = (curr_row, curr_col)

    # Если мы не достигли конца
    if not visited[end_row][end_col]:
        return None

    # Реконструировать путь
    path = []
    current = end
    while current != start:
        path.append(current)
        current = predecessors[current]
    path.append(start)
    path.reverse()

    return path

def _can_move_in_direction(maze: Maze, row: int, col: int, dr: int, dc: int) -> bool:
    """Проверить, возможно ли движение в заданном направлении (без стены)"""
    if dr == 0 and dc == 1:  # Движение вправо
        return not maze.has_right_wall(row, col)
    elif dr == 1 and dc == 0:  # Движение вниз
        return not maze.has_bottom_wall(row, col)
    elif dr == 0 and dc == -1:  # Движение влево
        return not maze.has_left_wall(row, col)
    elif dr == -1 and dc == 0:  # Движение вверх
        return not maze.has_top_wall(row, col)
    return False
```

## Альтернативные подходы

### Поиск в глубину (DFS)

- Использует стек вместо очереди
- Может найти решение быстрее, но не обязательно кратчайшее
- Использует меньше памяти в некоторых случаях
- Реализация аналогична, но использует `append()` и `pop()` (LIFO)

```python
def solve_maze_dfs(maze: Maze, start: Tuple[int, int], end: Tuple[int, int]) -> Optional[List[Tuple[int, int]]]:
    stack = [start]
    visited = [[False for _ in range(maze.cols)] for _ in range(maze.rows)]
    predecessors = {}
    visited[start[0]][start[1]] = True
    
    # Похоже на BFS, но с операциями стека
    # ... детали реализации
```

### Алгоритм A*

- Более сложный, но может быть быстрее для больших лабиринтов
- Использует эвристику для направления поиска к цели
- Гарантирует кратчайший путь, если эвристика допустима
- Больше вычислений на шаг, но потенциально меньше общих шагов

## Реконструкция пути

Функция `_reconstruct_path` критична для получения фактического пути:

```python
def _reconstruct_path(predecessors, start, end):
    path = []
    current = end
    while current != start:
        path.append(current)
        current = predecessors[current]
    path.append(start)
    path.reverse()
    return path
```

## Соображения производительности

### Временная сложность
- **BFS**: O(V + E) где V - вершины (ячейки) и E - ребра (возможные ходы)
- Для лабиринта: O(rows × cols)

### Пространственная сложность
- **BFS**: O(V) для очереди и массива посещенных
- Для лабиринта: O(rows × cols)

### Советы по оптимизации

1. **Раннее прекращение**: Остановиться, как только достигнут конец
2. **Двунаправленный поиск**: Искать одновременно от начала и конца
3. **Предварительная обработка**: Проверить, находятся ли начало и конец в одной и той же связной компоненте

## Интеграция с визуализацией

В контексте GUI путь должен быть визуализирован:

```python
def draw_solution_path(screen, path, cell_width, cell_height):
    if not path:
        return
        
    # Нарисовать линию через центр каждой ячейки в пути
    for i in range(len(path) - 1):
        row1, col1 = path[i]
        row2, col2 = path[i + 1]
        
        x1 = col1 * cell_width + cell_width // 2
        y1 = row1 * cell_height + cell_height // 2
        x2 = col2 * cell_width + cell_width // 2  
        y2 = row2 * cell_height + cell_height // 2
        
        pygame.draw.line(screen, (255, 0, 0), (x1, y1), (x2, y2), 2)  # Красный, 2px толщиной
```

## Обработка ошибок

Решатель должен корректно обрабатывать:
- Недействительные начальные/конечные позиции
- Отсутствие возможного пути
- Начало и конец совпадают
- Начало и конец находятся в изолированных секциях лабиринта

BFS - это оптимальный выбор для этого проекта, потому что он гарантирует кратчайший путь, сохраняя при этом простоту и надежность.