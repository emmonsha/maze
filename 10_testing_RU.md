# Тестовая стратегия

## Обзор

Тестирование обеспечивает правильную работу всех компонентов проекта лабиринта и поддерживает качество кода. Комплексная тестовая стратегия включает модульные тесты, интеграционные тесты и проверку сложных алгоритмов.

## Тестовый фреймворк Python: unittest

### Основная структура теста

```python
import unittest
from maze_game.maze.maze import Maze

class TestMaze(unittest.TestCase):
    """Тестовые случаи для класса Maze."""
    
    def test_maze_initialization(self):
        """Проверить, что лабиринт правильно инициализируется со стенами."""
        maze = Maze(5, 5)
        
        # Проверить размеры
        self.assertEqual(maze.rows, 5)
        self.assertEqual(maze.cols, 5)
        
        # Проверить, что границы имеют стены
        # Верхняя граница
        for j in range(5):
            self.assertTrue(maze.horizontal_walls[0][j])

if __name__ == '__main__':
    unittest.main()
```

## Модульное тестирование компонентов

### Тесты класса Maze

```python
class TestMaze(unittest.TestCase):
    def test_has_walls(self):
        """Проверить методы проверки стен."""
        maze = Maze(3, 3)
        
        # Изначально все внутренние стены должны существовать
        self.assertTrue(maze.has_right_wall(0, 0))
        self.assertTrue(maze.has_bottom_wall(0, 0))
        self.assertTrue(maze.has_left_wall(0, 0)) 
        self.assertTrue(maze.has_top_wall(0, 0))
        
        # Граничные стены должны существовать
        self.assertTrue(maze.has_top_wall(0, 0))  # Верхняя граница
        self.assertTrue(maze.has_left_wall(0, 0))  # Левая граница
        self.assertTrue(maze.has_right_wall(0, 2))  # Правая граница
        self.assertTrue(maze.has_bottom_wall(2, 0))  # Нижняя граница
    
    def test_file_io(self):
        """Проверить загрузку и сохранение лабиринта в файл."""
        import tempfile
        import os
        
        # Создать простой лабиринт
        original_maze = Maze(2, 2)
        # Убрать одну стену, чтобы создать простой путь
        original_maze.vertical_walls[0][1] = False
        
        # Создать временный файл
        with tempfile.NamedTemporaryFile(mode='w', delete=False, suffix='.txt') as f:
            temp_filename = f.name
        
        try:
            # Сохранить лабиринт в файл
            original_maze.save_to_file(temp_filename)
            
            # Загрузить лабиринт из файла
            loaded_maze = Maze(1, 1)  # Создать фиктивный для использования метода загрузки / Создать Фиктивный для Использования Метода Загрузки
            loaded_maze = loaded_maze.load_from_file(temp_filename)
            
            # Проверить, что размеры совпадают
            self.assertEqual(loaded_maze.rows, original_maze.rows)
            self.assertEqual(loaded_maze.cols, original_maze.cols)
            
            # Проверить, что стены совпадают
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
            # Очистить временный файл
            os.remove(temp_filename)
    
    def test_is_perfect(self):
        """Проверить метод is_perfect."""
        # Создать простой совершенный лабиринт (все ячейки соединены, нет петель)
        perfect_maze = Maze(2, 2)
        # Создать путь, который соединяет все ячейки
        perfect_maze.vertical_walls[0][1] = False  # (0,0) к (0,1)
        perfect_maze.horizontal_walls[1][1] = False  # (0,1) к (1,1)
        perfect_maze.vertical_walls[1][1] = False  # (1,1) к (1,0)
        
        self.assertTrue(perfect_maze.is_perfect())
```

### Тесты генератора

```python
class TestGenerator(unittest.TestCase):
    def test_generate_basic_maze(self):
        """Проверить генерацию базового лабиринта."""
        maze = generate_maze(5, 5)
        
        # Проверить размеры
        self.assertEqual(maze.rows, 5)
        self.assertEqual(maze.cols, 5)
        
        # Лабиринт должен быть совершенным
        self.assertTrue(maze.is_perfect())
    
    def test_generate_different_sizes(self):
        """Проверить генерацию лабиринтов разных размеров."""
        # Проверить маленький лабиринт
        small_maze = generate_maze(2, 2)
        self.assertEqual(small_maze.rows, 2)
        self.assertEqual(small_maze.cols, 2)
        
        # Проверить большой лабиринт
        large_maze = generate_maze(10, 8)
        self.assertEqual(large_maze.rows, 10)
        self.assertEqual(large_maze.cols, 8)
        
        # Оба должны быть совершенными
        self.assertTrue(small_maze.is_perfect())
        self.assertTrue(large_maze.is_perfect())
```

### Тесты решателя / Тесты Решателя

```python
class TestSolver(unittest.TestCase):
    def test_solve_simple_path(self):
        """Проверить решение простого прямого пути."""
        maze = Maze(3, 3)
        # Создать простой путь от (0,0) к (0,2)
        maze.vertical_walls[0][1] = False  # (0,0) к (0,1)
        maze.vertical_walls[0][2] = False  # (0,1) к (0,2)
        
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
        # Оставить все стены, поэтому путь от (0,0) к (2,2) не существует
        
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
        initial_state = [row[:] for row in cave.grid]  # Копировать сетку
        
        changed = cave.step_simulation(birth_limit=4, death_limit=3)
        
        # Сетка должна была измениться в большинстве случаев
        # (случайная инициализация может создать стабильное состояние)
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
        
        # Проверить, что Q-таблица имеет правильную структуру
        self.assertIn((0, 0), agent.q_table)
        self.assertIn(0, agent.q_table[(0, 0)])  # Действие 0 (вверх)
        self.assertIn(1, agent.q_table[(0, 0)])  # Действие 1 (вправо)
        self.assertIn(2, agent.q_table[(0, 0)])  # Действие 2 (вниз)
        self.assertIn(3, agent.q_table[(0, 0)])  # Действие 3 (влево)
    
    def test_get_reward(self):
        """Проверить функцию награды."""
        maze = Maze(3, 3)
        end_pos = (2, 2)
        agent = QLearningAgent(maze, end_pos)
        
        # Награда в конечной позиции должна быть высокой
        end_reward = agent.get_reward(end_pos)
        self.assertGreater(end_reward, 50)  # Должна быть 100
        
        # Другие позиции должны иметь отрицательную награду
        other_reward = agent.get_reward((0, 0))
        self.assertLess(other_reward, 0)  # Должна быть -1
    
    def test_get_possible_actions(self):
        """Проверить получение возможных действий из позиции."""
        maze = Maze(3, 3)
        # Добавить некоторые стены, чтобы ограничить движение
        maze.horizontal_walls[1][0] = True  # Блокировать движение вниз от (0,0)
        
        end_pos = (2, 2)
        agent = QLearningAgent(maze, end_pos)
        
        actions = agent.get_possible_actions((0, 0))
        # Не должно включать действие 2 (вниз) из-за стены
        self.assertNotIn(2, actions)
    
    def test_agent_training(self):
        """Проверить, что агент может быть обучен без ошибок."""
        maze = Maze(3, 3)
        end_pos = (2, 2)
        agent = QLearningAgent(maze, end_pos)
        
        # Обучение не должно вызывать исключений
        try:
            agent.train(episodes=10)  # Малое число для тестирования
        except Exception as e:
            self.fail(f"Обучение неудачно с исключением: {e}")
```

## Тестовые стратегии

### Разработка через тестирование (TDD)

```python
# Пример: Написать тест сначала, затем реализацию
def test_new_feature(self):
    """Проверить функцию, которая еще не существует."""
    maze = Maze(5, 5)
    # Мы ожидаем, что этот метод будет существовать и работать должным образом
    complexity = maze.calculate_complexity()
    self.assertGreater(complexity, 0)
```

### Граничное тестирование

```python
def test_boundary_conditions(self):
    """Проверить граничные значения."""
    # Проверить самый маленький возможный лабиринт
    small_maze = Maze(1, 1)
    self.assertEqual(small_maze.rows, 1)
    self.assertEqual(small_maze.cols, 1)
    
    # Проверить самый большой допустимый лабиринт
    large_maze = Maze(50, 50)  # Максимальный размер по требованиям
    self.assertEqual(large_maze.rows, 50)
    self.assertEqual(large_maze.cols, 50)
```

### Крайние случаи / Крайние Случаи

```python
def test_edge_cases(self):
    """Проверить крайние случаи."""
    # Пустой лабиринт (недействительный, но должен обрабатываться корректно)
    with self.assertRaises(Exception):  # Или любая соответствующая обработка
        invalid_maze = Maze(0, 0)
    
    # Лабиринт с одной ячейкой
    single_cell = Maze(1, 1)
    self.assertTrue(single_cell.has_top_wall(0, 0))  # Граничная стена
```

## Запуск тестов

### Использование unittest

```bash
# Запустить все тесты
python -m unittest discover maze_game/tests -v

# Запустить конкретный тестовый файл
python -m unittest maze_game.tests.test_maze -v

# Запустить конкретный тестовый класс
python -m unittest maze_game.tests.test_maze.TestMaze -v

# Запустить один тестовый метод
python -m unittest maze_game.tests.test_maze.TestMaze.test_maze_initialization -v
```

### Использование Makefile

```makefile
tests:
	@echo "Запуск тестов..."
	$(PYTHON) -m unittest discover maze_game/tests -v
```

## Метрики качества

### Покрытие тестов

Хотя в этом проекте не используются инструменты покрытия, хорошие практики включают: / Хотя в этом проекте не используются инструменты покрытия, хорошие практики включают:

- **100% публичных методов** должны иметь тесты
- **Крайние случаи** должны быть покрыты
- **Ошибочные условия** должны быть протестированы
- **Граничные значения** должны быть протестированы

### Изоляция тестов

Каждый тест должен:

- Быть независимым от других тестов
- Не полагаться на глобальное состояние, когда возможно
- Самостоятельно очищаться
- Иметь четкие, описательные имена

## Интеграционное тестирование

Проверить, как компоненты работают вместе:

```python
def test_maze_generation_to_solving(self):
    """Проверить генерацию лабиринта и затем его решение."""
    maze = generate_maze(10, 10)
    
    # Должен быть решаем от (0,0) до (9,9)
    start = (0, 0)
    end = (maze.rows - 1, maze.cols - 1)
    path = solve_maze(maze, start, end)
    
    # Путь должен существовать в совершенном лабиринте
    self.assertIsNotNone(path)
    self.assertGreater(len(path), 0)
```

## Имитация и заглушки

Для сложных зависимостей используйте имитацию:

```python
from unittest.mock import Mock

def test_with_mock_dependencies(self):
    """Проверить с имитированными зависимостями."""
    # Создать имитационный лабиринт с заранее определенным поведением
    mock_maze = Mock()
    mock_maze.rows = 5
    mock_maze.cols = 5
    mock_maze.has_right_wall.return_value = False  # Всегда разрешать движение вправо
    
    # Проверить поведение решателя с имитационным лабиринтом
    path = solve_maze(mock_maze, (0, 0), (0, 4))
    # Проверить, что решатель работает с имитационным лабиринтом
```

Комплексная тестовая стратегия гарантирует, что все компоненты работают должным образом и дает уверенность при внесении изменений в кодовую базу.