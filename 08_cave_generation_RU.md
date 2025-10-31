# Генерация пещер с клеточными автоматами

## Обзор

Генерация пещер использует клеточные автоматы для создания органических, пещероподобных структур. Клеточный автомат - это сетка ячеек, которая развивается в соответствии с простыми правилами, основанными на состояниях соседних ячеек.

## Основы клеточных автоматов

### Основные концепции

1. **Сетка**: 2D массив ячеек
2. **Состояния**: Каждая ячейка либо живая (стена), либо мертвая (пустое пространство)
3. **Соседи**: Обычно 8 смежных ячеек (окрестность Мура)
4. **Правила**: Простые условия, определяющие следующее состояние
5. **Итерации**: Повторное применение правил

### Алгоритм

Алгоритм генерации пещер имеет две фазы:
1. **Инициализация**: Случайное заполнение сетки
2. **Симуляция**: Применение клеточных правил многократно для развития узора

## Реализация

### Базовый класс пещеры

```python
class Cave:
    def __init__(self, rows: int, cols: int, initial_chance: float = 0.45):
        self.rows = rows
        self.cols = cols
        # True = живая/стена, False = мертвая/пустое
        self.grid = [[False for _ in range(cols)] for _ in range(rows)]
        self.initial_chance = initial_chance
        self._initialize_randomly()
    
    def _initialize_randomly(self):
        """Инициализировать сетку пещеры случайно на основе начального шанса."""
        for i in range(self.rows):
            for j in range(self.cols):
                self.grid[i][j] = random.random() < self.initial_chance
```

### Правила симуляции / Правила Симуляции

```python
def step_simulation(self, birth_limit: int = 4, death_limit: int = 3) -> bool:
    """
    Выполнить один шаг симуляции клеточного автомата.
    
    Правила:
    - Если мертвая ячейка имеет > birth_limit живых соседей, она становится живой
    - Если живая ячейка имеет < death_limit живых соседей, она умирает
    """
    new_grid = [[False for _ in range(self.cols)] for _ in range(self.rows)]
    changed = False
    
    for i in range(self.rows):
        for j in range(self.cols):
            # Подсчитать живых соседей
            neighbors = self._count_living_neighbors(i, j)
            
            # Применить правила
            if self.grid[i][j]:  # Если ячейка жива
                if neighbors < death_limit:
                    # Умирает из-за недостаточной популяции
                    new_grid[i][j] = False
                    changed = True
                else:
                    # Выживает
                    new_grid[i][j] = True
            else:  # Если ячейка мертва
                if neighbors > birth_limit:
                    # Становится живой из-за воспроизводства
                    new_grid[i][j] = True
                    changed = True
                else:
                    # Остается мертвой
                    new_grid[i][j] = False
    
    # Обновить сетку
    self.grid = new_grid
    return changed

def _count_living_neighbors(self, row: int, col: int) -> int:
    """Подсчитать живых соседей вокруг ячейки (окрестность Мура)."""
    neighbors = 0
    
    # Проверить все 8 направлений: вверх, вниз, влево, вправо и диагонали
    for di in [-1, 0, 1]:
        for dj in [-1, 0, 1]:
            if di == 0 and dj == 0:
                continue  # Пропустить саму ячейку
            
            ni, nj = row + di, col + dj
            
            # Ячейки вне пещеры рассматриваются как живые (стены)
            if 0 <= ni < self.rows and 0 <= nj < self.cols:
                if self.grid[ni][nj]:
                    neighbors += 1
            else:
                # Рассматривать вне границ как живые (создает границу)
                neighbors += 1
    
    return neighbors
```

### Запуск нескольких шагов

```python
def run_simulation(self, birth_limit: int = 4, death_limit: int = 3, steps: int = 5) -> int:
    """
    Запустить несколько шагов симуляции клеточного автомата.
    Возвращает количество выполненных шагов до стабилизации.
    """
    for step in range(steps):
        changed = self.step_simulation(birth_limit, death_limit)
        if not changed:
            # Если изменений не произошло, пещера стабилизировалась
            return step + 1
    
    return steps
```

## Управление параметрами

### Пределы рождения и смерти

- **Предел рождения**: Минимальное количество живых соседей для того, чтобы мертвая ячейка стала живой
- **Предел смерти**: Максимальное количество живых соседей для того, чтобы живая ячейка осталась живой

```text
Низкий предел рождения (например, 2-3): Больше ячеек становятся живыми → более плотные пещеры
Высокий предел рождения (например, 5-6): Меньше ячеек становятся живыми → более разреженные пещеры
Низкий предел смерти (например, 1-2): Больше ячеек умирает → менее связанные пещеры
Высокий предел смерти (например, 4-5): Больше ячеек выживает → более связанные пещеры
```

### Начальный шанс

Начальный шанс определяет, сколько ячеек начинают как живые:

```text
Низкий начальный шанс (0.3-0.4): Меньше начальных стен → более открытые пещеры
Высокий начальный шанс (0.5-0.7): Больше начальных стен → более закрытые пещеры
```

## Реализация файлового ввода-вывода

```python
def save_to_file(self, filename: str):
    """Сохранить пещеру в файл."""
    with open(filename, 'w') as f:
        # Записать размеры
        f.write(f"{self.rows} {self.cols}\n")
        
        # Записать сетку (1 для живой, 0 для мертвой)
        for i in range(self.rows):
            row = [int(self.grid[i][j]) for j in range(self.cols)]
            f.write(" ".join(map(str, row)) + "\n")

def load_from_file(self, filename: str) -> 'Cave':
    """Загрузить пещеру из файла."""
    with open(filename, 'r') as f:
        lines = [line.strip() for line in f if line.strip()]
    
    # Первая строка содержит ряды и колонки
    rows, cols = map(int, lines[0].split())
    cave = Cave(rows, cols)
    
    # Остальные строки представляют сетку пещеры
    for i in range(rows):
        row_data = list(map(int, lines[i + 1].split()))
        for j in range(cols):
            cave.grid[i][j] = bool(row_data[j]) if j < len(row_data) else False
    
    return cave
```

## Визуализация в GUI

### Рисование Pygame

```python
def draw_cave(self):
    """Нарисовать пещеру на экране pygame."""
    cell_width, cell_height = self.calculate_cell_size()
    self.screen.fill(self.BG_COLOR)
    
    # Нарисовать ячейки пещеры
    for row in range(self.cave.rows):
        for col in range(self.cave.cols):
            x = col * cell_width
            y = row * cell_height
            
            # Нарисовать живые (стеновые) ячейки коричневым, мертвые (пустые) ячейки светло-серым
            if self.cave.grid[row][col]:
                # Стена пещеры
                pygame.draw.rect(self.screen, self.CAVE_WALL_COLOR, 
                                (x, y, cell_width, cell_height))
                # Нарисовать границу для видимости
                pygame.draw.rect(self.screen, (101, 67, 33), 
                                (x, y, cell_width, cell_height), 1)
            else:
                # Пустое пространство пещеры
                pygame.draw.rect(self.screen, self.BG_COLOR, 
                                (x, y, cell_width, cell_height))
                # Нарисовать границу для видимости
                pygame.draw.rect(self.screen, (200, 200, 200), 
                                (x, y, cell_width, cell_height), 1)
```

### Рисование веб-интерфейса

```javascript
function drawCave() {
    if (!currentCave) return;
    
    const canvas = document.getElementById('maze-canvas');
    const ctx = canvas.getContext('2d');
    
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    const cellWidth = canvas.width / currentCave.cols;
    const cellHeight = canvas.height / currentCave.rows;
    
    // Нарисовать ячейки пещеры
    for (let row = 0; row < currentCave.rows; row++) {
        for (let col = 0; col < currentCave.cols; col++) {
            const x = col * cellWidth;
            const y = row * cellHeight;
            
            if (currentCave.grid[row][col]) {
                // Нарисовать стену коричневой
                ctx.fillStyle = '#8B4513'; // Коричневый
                ctx.fillRect(x, y, cellWidth, cellHeight);
            } else {
                // Нарисовать пустое пространство светло-серым
                ctx.fillStyle = '#f0f0f0'; // Светло-серый
                ctx.fillRect(x, y, cellWidth, cellHeight);
            }
            
            // Нарисовать границу ячейки
            ctx.strokeStyle = '#cccccc';
            ctx.lineWidth = 1;
            ctx.strokeRect(x, y, cellWidth, cellHeight);
        }
    }
}
```

## Расширенные методы

### Сглаживание итераций

Несколько шагов симуляции уточняют структуру пещеры:

```python
def smooth_cave(self, iterations: int = 5):
    """Применить несколько шагов симуляции для сглаживания пещеры."""
    for _ in range(iterations):
        self.step_simulation(birth_limit=4, death_limit=3)
```

### Обработка краев

Решить, как обращаться с ячейками вне границ пещеры:

1. **Всегда живые (стены)**: Создает естественные границы
2. **Случайные**: Добавляет случайность к краям
3. **На основе среднего**: Сглаживает края

### Формирование туннелей

Чтобы создать более туннельноподобные структуры, можно:
- Использовать разные правила рождения/смерти для разных областей
- Применить постобработку для создания туннелей между большими открытыми пространствами 
- Использовать разные начальные распределения

## Настройка параметров

### Для открытых пещер
- Низкий начальный шанс (0.35-0.45)
- Низкий предел рождения (3-4)
- Высокий предел смерти (4-5)

### Для плотных пещер
- Высокий начальный шанс (0.55-0.65)
- Высокий предел рождения (5-6)
- Низкий предел смерти (2-3)

### Для естественно выглядящих пещер
- Начальный шанс: 0.40-0.50
- Предел рождения: 4 (по умолчанию)
- Предел смерти: 3 (по умолчанию)

## Интеграция с другими компонентами

Класс пещеры может быть интегрирован с:
- Системой файлового ввода-вывода для загрузки/сохранения
- GUI для визуализации
- Веб-интерфейсом для доступа через браузер
- Элементами управления параметрами для экспериментов

Клеточные автоматы обеспечивают элегантный и эффективный способ генерации органически выглядящих пещероподобных структур с помощью всего нескольких простых правил и параметров.