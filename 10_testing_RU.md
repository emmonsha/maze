# Тестовая стратегия

## Обзор / Обзор

Тестирование обеспечивает правильную работу всех компонентов проекта лабиринта и поддерживает качество кода. Комплексная тестовая стратегия включает модульные тесты, интеграционные тесты и проверку сложных алгоритмов. / Тестирование обеспечивает правильную работу всех компонентов проекта лабиринта и поддерживает качество кода. Комплексная тестовая стратегия включает модульные тесты, интеграционные тесты и проверку сложных алгоритмов.

## Тестовый фреймворк Python: unittest / Тестовый Фреймворк Python: unittest

### Основная структура теста / Основная Структура Теста

```python
import unittest
from maze_game.maze.maze import Maze

class TestMaze(unittest.TestCase):
    """Тестовые случаи для класса Maze."""
    
    def test_maze_initialization(self):
        """Проверить, что лабиринт правильно инициализируется со стенами."""
        maze = Maze(5, 5)
        
        # Проверить размеры / Проверить Размеры
        self.assertEqual(maze.rows, 5)
        self.assertEqual(maze.cols, 5)
        
        # Проверить, что границы имеют стены / Проверить, что Границы Имеют Стены
        # Верхняя граница / Верхняя Граница
        for j in range(5):
            self.assertTrue(maze.horizontal_walls[0][j])

if __name__ == '__main__':
    unittest.main()
```

## Модульное тестирование компонентов / Модульное Тестирование Компонентов

### Тесты класса Maze / Тесты Класса Maze

```python
class TestMaze(unittest.TestCase):
    def test_has_walls(self):
        """Проверить методы проверки стен."""
        maze = Maze(3, 3)
        
        # Изначально все внутренние стены должны существовать / Изначально Все Внутренние Стены Должны Существовать
        self.assertTrue(maze.has_right_wall(0, 0))
        self.assertTrue(maze.has_bottom_wall(0, 0))
        self.assertTrue(maze.has_left_wall(0, 0)) 
        self.assertTrue(maze.has_top_wall(0, 0))
        
        # Граничные стены должны существовать / Граничные Стены Должны Существовать
        self.assertTrue(maze.has_top_wall(0, 0))  # Верхняя граница / Верхняя Граница
        self.assertTrue(maze.has_left_wall(0, 0))  # Левая граница / Левая Граница
        self.assertTrue(maze.has_right_wall(0, 2))  # Правая граница / Правая Граница
        self.assertTrue(maze.has_bottom_wall(2, 0))  # Нижняя граница / Нижняя Граница
    
    def test_file_io(self):
        """Проверить загрузку и сохранение лабиринта в файл."""
        import tempfile
        import os
        
        # Создать простой лабиринт / Создать Простой Лабиринт
        original_maze = Maze(2, 2)
        # Убрать одну стену, чтобы создать простой путь / Убрать Одну Стену, Чтобы Создать Простой Путь
        original_maze.vertical_walls[0][1] = False
        
        # Создать временный файл / Создать Временный Файл
        with tempfile.NamedTemporaryFile(mode='w', delete=False, suffix='.txt') as f:
            temp_filename = f.name
        
        try:
            # Сохранить лабиринт в файл / Сохранить Лабиринт в Файл
            original_maze.save_to_file(temp_filename)
            
            # Загрузить лабиринт из файла / Загрузить Лабиринт из Файла
            loaded_maze = Maze(1, 1)  # Создать фиктивный для использования метода загрузки / Создать Фиктивный для Использования Метода Загрузки
            loaded_maze = loaded_maze.load_from_file(temp_filename)
            
            # Проверить, что размеры совпадают / Проверить, что Размеры Совпадают
            self.assertEqual(loaded_maze.rows, original_maze.rows)
            self.assertEqual(loaded_maze.cols, original_maze.cols)
            
            # Проверить, что стены совпадают / Проверить, что Стены Совпадают
            for i in range(loaded_maze.rows + 1):
                for j in range(loaded_maze.cols):
                    self.assertEqual(
                        loaded_maze.horizontal_walls[i][j],
                        original_maze.horizontal_walls[i][j]
                    )
            
            for i in range(loaded_maze.rows):
                for j in range(loaded_maze.cols + 1):
                    self.assertEqual(
                        loaded_maze.vertical_walls[i][j],
                        original_maze.vertical_walls[i][j]
                    )
        finally:
            # Очистить временный файл / Очистить Временный Файл
            os.remove(temp_filename)
    
    def test_is_perfect(self):
        """Проверить метод is_perfect."""
        # Создать простой совершенный лабиринт (все ячейки соединены, нет петель) / Создать Простой Совершенный Лабиринт (Все Ячейки Соединены, Нет Петель)
        perfect_maze = Maze(2, 2)
        # Создать путь, который соединяет все ячейки / Создать Путь, который Соединяет Все Ячейки
        perfect_maze.vertical_walls[0][1] = False  # (0,0) к (0,1) / (0,0) к (0,1)
        perfect_maze.horizontal_walls[1][1] = False  # (0,1) к (1,1) / (0,1) к (1,1) 
        perfect_maze.vertical_walls[1][1] = False  # (1,1) к (1,0) / (1,1) к (1,0)
        
        self.assertTrue(perfect_maze.is_perfect())
```

### Тесты генератора / Тесты Генератора

```python
class TestGenerator(unittest.TestCase):
    def test_generate_basic_maze(self):
        """Проверить генерацию базового лабиринта."""
        maze = generate_maze(5, 5)
        
        # Проверить размеры / Проверить Размеры
        self.assertEqual(maze.rows, 5)
        self.assertEqual(maze.cols, 5)
        
        # Лабиринт должен быть совершенным / Лабиринт Должен Быть Совершенным
        self.assertTrue(maze.is_perfect())
    
    def test_generate_different_sizes(self):
        """Проверить генерацию лабиринтов разных размеров."""
        # Проверить маленький лабиринт / Проверить Маленький Лабиринт
        small_maze = generate_maze(2, 2)
        self.assertEqual(small_maze.rows, 2)
        self.assertEqual(small_maze.cols, 2)
        
        # Проверить большой лабиринт / Проверить Большой Лабиринт
        large_maze = generate_maze(10, 8)
        self.assertEqual(large_maze.rows, 10)
        self.assertEqual(large_maze.cols, 8)
        
        # Оба должны быть совершенными / Оба Должны Быть Совершенными
        self.assertTrue(small_maze.is_perfect())
        self.assertTrue(large_maze.is_perfect())
```

### Тесты решателя / Тесты Решателя

```python
class TestSolver(unittest.TestCase):
    def test_solve_simple_path(self):
        """Проверить решение простого прямого пути."""
        maze = Maze(3, 3)
        # Создать простой путь от (0,0) к (0,2) / Создать Простой Путь От (0,0) К (0,2)
        maze.vertical_walls[0][1] = False  # (0,0) к (0,1) / (0,0) к (0,1)
        maze.vertical_walls[0][2] = False  # (0,1) к (0,2) / (0,1) к (0,2)
        
        start = (0, 0)
        end = (0, 2)
        path = solve_maze(maze, start, end)
        
        self.assertIsNotNone(path)
        self.assertGreater(len(path), 0)
        self.assertEqual(path[0], start)
        self.assertEqual(path[-1], end)
    
    def test_solve_no_path(self):
        """Проверить решение, когда путь не существует."""
        maze = Maze(3, 3)
        # Оставить все стены, поэтому путь от (0,0) к (2,2) не существует / Оставить Все Стены, Поэтому Путь От (0,0) К (2,2) Не Существует
        
        start = (0, 0)
        end = (2, 2)
        path = solve_maze(maze, start, end)
        
        self.assertIsNone(path)
    
    def test_solve_same_start_end(self):
        """Проверить решение, когда начало и конец совпадают."""
        maze = Maze(3, 3)
        start = (1, 1)
        end = (1, 1)
        path = solve_maze(maze, start, end)
        
        self.assertEqual(path, [start])
```

### Тесты пещеры / Тесты Пещеры

```python
class TestCave(unittest.TestCase):
    def test_cave_initialization(self):
        """Проверить, что пещера правильно инициализируется."""
        rows, cols = 10, 10
        cave = Cave(rows, cols, initial_chance=0.45)
        
        self.assertEqual(cave.rows, rows)
        self.assertEqual(cave.cols, cols)
    
    def test_cave_simulation_step(self):
        """Проверить один шаг симуляции пещеры."""
        cave = Cave(5, 5, initial_chance=0.5)
        initial_state = [row[:] for row in cave.grid]  # Копировать сетку / Копировать Сетку
        
        changed = cave.step_simulation(birth_limit=4, death_limit=3)
        
        # Сетка должна была измениться в большинстве случаев / Сетка Должна Была Измениться в Большинстве Случаев
        # (случайная инициализация может создать стабильное состояние) / (Случайная Инициализация Может Создать Стабильное Состояние)
        self.assertIsInstance(changed, bool)
    
    def test_cave_simulation_run(self):
        """Проверить выполнение нескольких шагов симуляции пещеры."""
        cave = Cave(10, 10, initial_chance=0.45)
        steps_performed = cave.run_simulation(birth_limit=4, death_limit=3, steps=5)
        
        self.assertGreater(steps_performed, 0)
        self.assertLessEqual(steps_performed, 5)
```

### Тесты Q-Learning / Тесты Q-Learning

```python
class TestQLearning(unittest.TestCase):
    def test_agent_initialization(self):
        """Проверить, что агент Q-learning правильно инициализируется."""
        maze = Maze(3, 3)
        end_pos = (2, 2)
        agent = QLearningAgent(maze, end_pos)
        
        # Проверить, что Q-таблица имеет правильную структуру / Проверить, что Q-таблица Имеет Правильную Структуру
        self.assertIn((0, 0), agent.q_table)
        self.assertIn(0, agent.q_table[(0, 0)])  # Действие 0 (вверх) / Действие 0 (Вверх)
        self.assertIn(1, agent.q_table[(0, 0)])  # Действие 1 (вправо) / Действие 1 (Вправо)
        self.assertIn(2, agent.q_table[(0, 0)])  # Действие 2 (вниз) / Действие 2 (Вниз)
        self.assertIn(3, agent.q_table[(0, 0)])  # Действие 3 (влево) / Действие 3 (Влево)
    
    def test_get_reward(self):
        """Проверить функцию награды."""
        maze = Maze(3, 3)
        end_pos = (2, 2)
        agent = QLearningAgent(maze, end_pos)
        
        # Награда в конечной позиции должна быть высокой / Награда в Конечной Позиции Должна Быть Высокой
        end_reward = agent.get_reward(end_pos)
        self.assertGreater(end_reward, 50)  # Должна быть 100 / Должна Быть 100
        
        # Другие позиции должны иметь отрицательную награду / Другие Позиции Должны Иметь Отрицательную Награду
        other_reward = agent.get_reward((0, 0))
        self.assertLess(other_reward, 0)  # Должна быть -1 / Должна Быть -1
    
    def test_get_possible_actions(self):
        """Проверить получение возможных действий из позиции."""
        maze = Maze(3, 3)
        # Добавить некоторые стены, чтобы ограничить движение / Добавить Некоторые Стены, Чтобы Ограничить Движение
        maze.horizontal_walls[1][0] = True  # Блокировать движение вниз от (0,0) / Блокировать Движение Вниз От (0,0)
        
        end_pos = (2, 2)
        agent = QLearningAgent(maze, end_pos)
        
        actions = agent.get_possible_actions((0, 0))
        # Не должно включать действие 2 (вниз) из-за стены / Не Должно Включать Действие 2 (Вниз) Из-за Стены
        self.assertNotIn(2, actions)
    
    def test_agent_training(self):
        """Проверить, что агент может быть обучен без ошибок."""
        maze = Maze(3, 3)
        end_pos = (2, 2)
        agent = QLearningAgent(maze, end_pos)
        
        # Обучение не должно вызывать исключений / Обучение Не Должно Вызывать Исключений
        try:
            agent.train(episodes=10)  # Малое число для тестирования / Малое Число для Тестирования
        except Exception as e:
            self.fail(f"Обучение неудачно с исключением: {e}")
```

## Тестовые стратегии / Тестовые Стратегии

### Разработка через тестирование (TDD) / Разработка Через Тестирование (TDD)

```python
# Пример: Написать тест сначала, затем реализацию / Пример: Написать Тест Сначала, Затем Реализацию
def test_new_feature(self):
    """Проверить функцию, которая еще не существует."""
    maze = Maze(5, 5)
    # Мы ожидаем, что этот метод будет существовать и работать должным образом / Мы Ожидаем, что Этот Метод Будет Существовать и Работать Должным Образом
    complexity = maze.calculate_complexity()
    self.assertGreater(complexity, 0)
```

### Граничное тестирование / Граничное Тестирование

```python
def test_boundary_conditions(self):
    """Проверить граничные значения."""
    # Проверить самый маленький возможный лабиринт / Проверить Самый Маленький Возможный Лабиринт
    small_maze = Maze(1, 1)
    self.assertEqual(small_maze.rows, 1)
    self.assertEqual(small_maze.cols, 1)
    
    # Проверить самый большой допустимый лабиринт / Проверить Самый Большой Допустимый Лабиринт
    large_maze = Maze(50, 50)  # Максимальный размер по требованиям / Максимальный Размер по Требованиям
    self.assertEqual(large_maze.rows, 50)
    self.assertEqual(large_maze.cols, 50)
```

### Крайние случаи / Крайние Случаи

```python
def test_edge_cases(self):
    """Проверить крайние случаи."""
    # Пустой лабиринт (недействительный, но должен обрабатываться корректно) / Пустой Лабиринт (Недействительный, но Должен Обрабатываться Корректно)
    with self.assertRaises(Exception):  # Или любая соответствующая обработка / Или Любая Соответствующая Обработка
        invalid_maze = Maze(0, 0)
    
    # Лабиринт с одной ячейкой / Лабиринт с Одной Ячейкой
    single_cell = Maze(1, 1)
    self.assertTrue(single_cell.has_top_wall(0, 0))  # Граничная стена / Граничная Стена
```

## Запуск тестов / Запуск Тестов

### Использование unittest / Использование unittest

```bash
# Запустить все тесты / Запустить Все Тесты
python -m unittest discover maze_game/tests -v

# Запустить конкретный тестовый файл / Запустить Конкретный Тестовый Файл
python -m unittest maze_game.tests.test_maze -v

# Запустить конкретный тестовый класс / Запустить Конкретный Тестовый Класс
python -m unittest maze_game.tests.test_maze.TestMaze -v

# Запустить один тестовый метод / Запустить Один Тестовый Метод
python -m unittest maze_game.tests.test_maze.TestMaze.test_maze_initialization -v
```

### Использование Makefile / Использование Makefile

```makefile
tests:
	@echo "Запуск тестов..."
	$(PYTHON) -m unittest discover maze_game/tests -v
```

## Метрики качества / Метрики Качества

### Покрытие тестов / Покрытие Тестов

Хотя в этом проекте не используются инструменты покрытия, хорошие практики включают: / Хотя в этом проекте не используются инструменты покрытия, хорошие практики включают:

- **100% публичных методов** должны иметь тесты / **100% Публичных Методов** Должны Иметь Тесты
- **Крайние случаи** должны быть покрыты / **Крайние Случаи** Должны Быть Покрыты
- **Ошибочные условия** должны быть протестированы / **Ошибочные Условия** Должны Быть Протестированы
- **Граничные значения** должны быть протестированы / **Граничные Значения** Должны Быть Протестированы

### Изоляция тестов / Изоляция Тестов

Каждый тест должен: / Каждый Тест Должен:

- Быть независимым от других тестов / Быть Независимым от Других Тестов
- Не полагаться на глобальное состояние, когда возможно / Не Полагаться на Глобальное Состояние, Когда Возможно
- Самостоятельно очищаться / Самостоятельно Очищаться
- Иметь четкие, описательные имена / Иметь Четкие, Описательные Имена

## Интеграционное тестирование / Интеграционное Тестирование

Проверить, как компоненты работают вместе: / Проверить, Как Компоненты Работают Вместе:

```python
def test_maze_generation_to_solving(self):
    """Проверить генерацию лабиринта и затем его решение."""
    maze = generate_maze(10, 10)
    
    # Должен быть решаем от (0,0) до (9,9) / Должен Быть Решаем От (0,0) До (9,9)
    start = (0, 0)
    end = (maze.rows - 1, maze.cols - 1)
    path = solve_maze(maze, start, end)
    
    # Путь должен существовать в совершенном лабиринте / Путь Должен Существовать в Совершенном Лабиринте
    self.assertIsNotNone(path)
    self.assertGreater(len(path), 0)
```

## Имитация и заглушки / Имитация и Заглушки

Для сложных зависимостей используйте имитацию: / Для Сложных Зависимостей Используйте Имитацию:

```python
from unittest.mock import Mock

def test_with_mock_dependencies(self):
    """Проверить с имитированными зависимостями."""
    # Создать имитационный лабиринт с заранее определенным поведением / Создать Имитационный Лабиринт с Заранее Определенным Поведением
    mock_maze = Mock()
    mock_maze.rows = 5
    mock_maze.cols = 5
    mock_maze.has_right_wall.return_value = False  # Всегда разрешать движение вправо / Всегда Разрешать Движение Вправо
    
    # Проверить поведение решателя с имитационным лабиринтом / Проверить Поведение Решателя с Имитационным Лабиринтом
    path = solve_maze(mock_maze, (0, 0), (0, 4))
    # Проверить, что решатель работает с имитационным лабиринтом / Проверить, что Решатель Работает с Имитационным Лабиринтом
```

Комплексная тестовая стратегия гарантирует, что все компоненты работают должным образом и дает уверенность при внесении изменений в кодовую базу. / Комплексная тестовая стратегия гарантирует, что все компоненты работают должным образом и дает уверенность при внесении изменений в кодовую базу.