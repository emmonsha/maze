# Веб-интерфейс

## Обзор / Обзор

Веб-интерфейс обеспечивает браузерную альтернативу графическому интерфейсу рабочего стола, реализуя те же функциональные возможности с использованием Flask (веб-фреймворка Python) и HTML/CSS/JavaScript. Это позволяет получить доступ к игре лабиринта с любого устройства, имеющего веб-браузер. / Веб-интерфейс обеспечивает браузерную альтернативу графическому интерфейсу рабочего стола, реализуя те же функциональные возможности с использованием Flask (веб-фреймворка Python) и HTML/CSS/JavaScript. Это позволяет получить доступ к игре лабиринта с любого устройства, имеющего веб-браузер.

## Основы Flask / Основы Flask

### Структура приложения Flask / Структура Приложения Flask

```python
from flask import Flask, render_template, request, jsonify

def create_app():
    app = Flask(__name__)
    
    # Глобальное состояние (в реальном приложении используйте сессии или базы данных) / Глобальное Состояние (В Реальном Приложении Используйте Сессии или Базы Данных)
    current_maze = generate_maze(10, 10)
    current_solution = None
    current_start = None
    current_end = None
    
    @app.route('/')
    def index():
        """Главная страница игры лабиринт."""
        return render_template('maze.html')
    
    # Конечные точки API для AJAX вызовов / Конечные Точки API для AJAX Вызовов
    @app.route('/api/maze', methods=['GET'])
    def get_maze():
        # Вернуть текущий лабиринт как JSON / Вернуть Текущий Лабиринт как JSON
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

### Маршруты и конечные точки Flask / Маршруты и Конечные Точки Flask

Flask использует декораторы для определения маршрутов: / Flask использует декораторы для определения маршрутов:

```python
@app.route('/api/maze', methods=['GET'])
def get_maze():
    """Получить текущий лабиринт как JSON."""
    return jsonify({
        'rows': current_maze.rows,
        'cols': current_maze.cols,
        'horizontal_walls': current_maze.horizontal_walls,
        'vertical_walls': current_maze.vertical_walls
    })

@app.route('/api/maze', methods=['POST'])
def create_maze():
    """Сгенерировать новый лабиринт с указанными размерами."""
    data = request.json
    rows = data.get('rows', 10)
    cols = data.get('cols', 10)
    
    # Проверить размер / Проверить Размер
    rows = min(rows, 50)
    cols = min(cols, 50)
    
    global current_maze
    current_maze = generate_maze(rows, cols)
    
    return jsonify({'status': 'success'})
```

## Реализация интерфейса / Реализация Интерфейса

### Структура HTML шаблона / Структура HTML Шаблона

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Игра лабиринт</title>
    <style>
        /* CSS стили / CSS Стили */
        body { font-family: Arial, sans-serif; margin: 20px; }
        #maze-canvas { border: 1px solid #000; }
        .controls { display: inline-block; margin-left: 20px; }
    </style>
</head>
<body>
    <h1>Игра лабиринт</h1>
    
    <div id="maze-container">
        <canvas id="maze-canvas" width="500" height="500"></canvas>
    </div>
    
    <div class="controls">
        <div class="control-group">
            <h3>Сгенерировать лабиринт</h3>
            <label for="rows">Ряды:</label>
            <input type="number" id="rows" value="10" min="5" max="50">
            
            <label for="cols">Колонки:</label>
            <input type="number" id="cols" value="10" min="5" max="50">
            
            <button onclick="generateMaze()">Сгенерировать новый лабиринт</button>
        </div>
        <!-- Больше элементов управления / Больше Элементов Управления -->
    </div>

    <script>
        // JavaScript код / JavaScript Код
    </script>
</body>
</html>
```

### Интеграция JavaScript AJAX / Интеграция JavaScript AJAX

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
        updateStatus('Лабиринт загружен. Установите начальную и конечную точки для решения.');
    })
    .catch(error => {
        console.error('Ошибка загрузки лабиринта:', error);
        updateStatus('Ошибка загрузки лабиринта');
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
        console.error('Ошибка генерации лабиринта:', error);
        updateStatus('Ошибка генерации лабиринта');
    });
}
```

### Рисование Canvas с JavaScript / Рисование Canvas с JavaScript

```javascript
function drawMaze() {
    if (!currentMaze) return;
    
    const canvas = document.getElementById('maze-canvas');
    const ctx = canvas.getContext('2d');
    
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    const cellWidth = canvas.width / currentMaze.cols;
    const cellHeight = canvas.height / currentMaze.rows;
    
    // Нарисовать ячейки и стены / Нарисовать Ячейки и Стены
    for (let row = 0; row < currentMaze.rows; row++) {
        for (let col = 0; col < currentMaze.cols; col++) {
            // Нарисовать фон ячейки / Нарисовать Фон Ячейки
            ctx.fillStyle = '#ffffff';
            ctx.fillRect(col * cellWidth, row * cellHeight, cellWidth, cellHeight);
            
            // Нарисовать правую стену / Нарисовать Правую Стену
            if (currentMaze.vertical_walls[row][col + 1]) {
                ctx.beginPath();
                ctx.moveTo((col + 1) * cellWidth, row * cellHeight);
                ctx.lineTo((col + 1) * cellWidth, (row + 1) * cellHeight);
                ctx.strokeStyle = '#000000';
                ctx.lineWidth = 2;
                ctx.stroke();
            }
            
            // Нарисовать нижнюю стену / Нарисовать Нижнюю Стену
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
    
    // Нарисовать границы / Нарисовать Границы
    ctx.strokeRect(0, 0, canvas.width, canvas.height);
    
    // Нарисовать путь решения, если он существует / Нарисовать Путь Решения, Если Он Существует
    drawSolution(ctx, cellWidth, cellHeight);
    drawPoints(ctx, cellWidth, cellHeight);
}

function drawSolution(ctx, cellWidth, cellHeight) {
    if (!solution) return;
    
    ctx.strokeStyle = '#ff0000';
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

### Взаимодействие мыши / Взаимодействие Мыши

```javascript
const canvas = document.getElementById('maze-canvas');

canvas.addEventListener('click', function(e) {
    const rect = canvas.getBoundingClientRect();
    const x = e.clientX - rect.left;
    const y = e.clientY - rect.top;
    
    if (!currentMaze) {
        updateStatus('Пожалуйста, сначала сгенерируйте лабиринт');
        return;
    }
    
    // Рассчитать, какая ячейка была кликнута / Рассчитать, Какая Ячейка Была Кликнута
    const cellWidth = canvas.width / currentMaze.cols;
    const cellHeight = canvas.height / currentMaze.rows;
    const col = Math.floor(x / cellWidth);
    const row = Math.floor(y / cellHeight);
    
    if (mode === 'start') {
        start = [row, col];
        mode = 'end';
        updateStatus(`Начало установлено в (${row}, ${col}). Теперь кликните для конечной точки.`);
        drawMaze();
    } else if (mode === 'end') {
        end = [row, col];
        mode = 'none';
        updateStatus(`Конец установлен в (${row}, ${col}). Решение лабиринта...`);
        
        // Решить лабиринт / Решить Лабиринт
        solveMaze();
    }
});
```

## Расширенные функции / Расширенные Функции

### Реализация режима пещеры / Реализация Режима Пещеры

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
        drawCurrent(); // Переключиться на drawCave(); / Переключиться на drawCave();
        updateStatus(`Пещера сгенерирована: ${data.rows}x${data.cols}`);
    })
    .catch(error => {
        console.error('Ошибка генерации пещеры:', error);
        updateStatus('Ошибка генерации пещеры');
    });
}

function drawCave() {
    if (!currentCave) return;
    
    const canvas = document.getElementById('maze-canvas');
    const ctx = canvas.getContext('2d');
    
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    const cellWidth = canvas.width / currentCave.cols;
    const cellHeight = canvas.height / currentCave.rows;
    
    // Нарисовать ячейки пещеры / Нарисовать Ячейки Пещеры
    for (let row = 0; row < currentCave.rows; row++) {
        for (let col = 0; col < currentCave.cols; col++) {
            const x = col * cellWidth;
            const y = row * cellHeight;
            
            // Нарисовать живые (стеновые) ячейки коричневыми, мертвые (пустые) ячейки светло-серыми / Нарисовать Живые (Стеновые) Ячейки Коричневыми, Мертвые (Пустые) Ячейки Светло-Серыми
            if (currentCave.grid[row][col]) {
                ctx.fillStyle = '#8B4513'; // Коричневый / Коричневый
                ctx.fillRect(x, y, cellWidth, cellHeight);
            } else {
                ctx.fillStyle = '#f0f0f0'; // Светло-серый / Светло-Серый
                ctx.fillRect(x, y, cellWidth, cellHeight);
            }
            
            // Нарисовать границу для видимости / Нарисовать Границу для Видимости
            ctx.strokeStyle = '#cccccc';
            ctx.lineWidth = 1;
            ctx.strokeRect(x, y, cellWidth, cellHeight);
        }
    }
    
    // Нарисовать границы / Нарисовать Границы
    ctx.strokeRect(0, 0, canvas.width, canvas.height);
}
```

## Интеграция API Flask / Интеграция API Flask

### Управление состоянием / Управление Состоянием

```python
def create_app():
    # Хранить текущее состояние / Хранить Текущее Состояние
    current_maze = generate_maze(10, 10)
    current_solution = None
    current_start = None
    current_end = None
    
    @app.route('/api/solve', methods=['POST'])
    def solve():
        """Решить лабиринт с заданными начальной и конечной точками."""
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

### Визуализация шаблонов с Jinja2 / Визуализация Шаблонов с Jinja2

Шаблонизатор Flask (Jinja2) позволяет динамическую генерацию HTML: / Шаблонизатор Flask (Jinja2) позволяет динамическую генерацию HTML:

```html
<!-- В maze.html / В maze.html -->
<div class="status" id="status">
    {% if current_maze %}
        Лабиринт загружен с размерами {{ current_maze.rows }} x {{ current_maze.cols }}
    {% else %}
        Нажмите "Сгенерировать новый лабиринт" для начала
    {% endif %}
</div>
```

## Соображения развертывания / Соображения Развертывания

### Запуск сервера / Запуск Сервера

```bash
# Прямой запуск / Прямой Запуск
python -m maze_game.web.app

# Использование Make / Использование Make
make web
```

### Конфигурация для производства / Конфигурация Для Производства

```python
def create_app():
    app = Flask(__name__)
    
    # Конфигурация для производства / Конфигурация Для Производства
    if not app.debug:
        app.config['SECRET_KEY'] = os.environ.get('SECRET_KEY')
    
    # ... маршруты / ... Маршруты
    return app
```

## Распространенные проблемы и решения / Распространенные Проблемы и Решения

### 1. Управление состоянием / 1. Управление Состоянием
- **Проблема**: Веб-запросы не имеют состояния / **Проблема**: Веб-запросы не имеют состояния
- **Решение**: Использовать серверное хранение сессий или глобальные переменные для разработки / **Решение**: Использовать серверное хранение сессий или глобальные переменные для разработки

### 2. Обновления в реальном времени / 2. Обновления в Реальном Времени
- **Проблема**: Необходимо обновлять дисплей без перезагрузки страницы / **Проблема**: Необходимо обновлять дисплей без перезагрузки страницы
- **Решение**: Использовать AJAX для вызовов API и JavaScript для динамических обновлений / **Решение**: Использовать AJAX для вызовов API и JavaScript для динамических обновлений

### 3. Проблемы кросс-оригина / 3. Проблемы Кросс-Оригина
- **Проблема**: Интерфейс и бэкенд могут быть на разных портах / **Проблема**: Интерфейс и бэкенд могут быть на разных портах
- **Решение**: Соответствующие заголовки CORS (не нужны при обслуживании из того же приложения Flask) / **Решение**: Соответствующие заголовки CORS (не нужны при обслуживании из того же приложения Flask)

### 4. Загрузка файлов / 4. Загрузка Файлов
- **Проблема**: Загрузка файлов лабиринтов через веб-интерфейс / **Проблема**: Загрузка файлов лабиринтов через веб-интерфейс
- **Решение**: Использовать HTML файловый ввод с XMLHttpRequest или fetch API / **Решение**: Использовать HTML файловый ввод с XMLHttpRequest или fetch API

## Лучшие практики / Лучшие Практики

1. **Разделение ответственности**: Держать Flask, HTML, CSS и JavaScript организованными / 1. **Разделение Ответственности**: Держать Flask, HTML, CSS и JavaScript организованными
2. **Обработка ошибок**: Предоставлять четкие сообщения об ошибках для сбоев API / 2. **Обработка Ошибок**: Предоставлять четкие сообщения об ошибках для сбоев API
3. **Безопасность**: Проверять весь ввод на стороне сервера / 3. **Безопасность**: Проверять весь ввод на стороне сервера
4. **Производительность**: Оптимизировать рисование и избегать ненужных вызовов API / 4. **Производительность**: Оптимизировать рисование и избегать ненужных вызовов API
5. **Пользовательский опыт**: Предоставлять индикаторы загрузки и четкую обратную связь / 5. **Пользовательский Опыт**: Предоставлять индикаторы загрузки и четкую обратную связь

Flask обеспечивает отличную платформу для создания веб-интерфейсов, которые могут дополнять настольные приложения, предлагая доступность и независимость от платформы. / Flask обеспечивает отличную платформу для создания веб-интерфейсов, которые могут дополнять настольные приложения, предлагая доступность и независимость от платформы.