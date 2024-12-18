# Лабораторная работа №3

## Тема: Использование принципов проектирования на уровне методов и классов

### Цель работы
Получить опыт проектирования и реализации модулей с использованием принципов KISS, YAGNI, DRY, SOLID и др.

---

### 1. Диаграмма контейнеров
![C4-container](https://github.com/user-attachments/assets/80b072ba-c653-46d8-9bdd-65366f85e942)

**Контейнеры системы:**
1. Чат-бот (Telegram Bot):
  - Основной интерфейс для взаимодействия преподавателей и студентов с системой.
  - Обрабатывает пользовательские запросы, передает их серверу и отправляет пользователю ответы.

2. Сервер приложения:
  - Обрабатывает бизнес-логику.
  - Взаимодействует с моделью машинного обучения для анализа и классификации кода.
  - Генерирует отчеты и комментарии по каждому критерию оценки.
  - Обеспечивает интеграцию с GitHub для автоматической выгрузки кода по запросу преподавателя.

3. База данных:
  - Хранит информацию о преподавателях (ID телеграм пользователя, корпоративная почта).

4. Система контроля версий (GitHub):
  - Обеспечивает доступ к исходному коду студентов.
  - Используется для загрузки и хранения проектов.

### 2. Диаграмма компонентов
![C4-Компоненты(Система оценки кода)](https://github.com/user-attachments/assets/7c46fc64-7abc-4e81-9396-19e691fd3bae)

▎**Компоненты системы**
1. **Telegram-бот**
   - Интерфейс для преподавателей: Позволяет авторизованным преподавателям запрашивать отчеты по коду студентов и получать результаты в текстовом формате.
   - Интерфейс для студентов: Позволяет студентам загружать свои работы и получать отчеты с комментариями по критериям.
   - Поддерживает авторизацию преподавателей и студентов.
   - Передает запросы на сервер и возвращает результаты пользователям.

2. **Система оценки кода**
   - Модуль анализа кода: Выполняет автоматическую проверку загруженного кода на основе заданных критериев. Формирует краткие комментарии по каждому критерию.
   - Модуль классификации:  Присваивает проекту класс ("плохой", "средний", "отличный").
   - Модуль генерации отчетов: Формирует отчеты с результатами анализа, комментариями и рекомендациями.

3. **База данных**
   - Хранение данных о преподавателях: Содержит информацию о преподавателях, включая ID Telegram пользователя и корпоративную почту.

4. **Система контроля версий (GitHub)**
   - Хранит исходный код студентов.
   - Обеспечивает доступ к исходному коду студентов по запросу.

▎**Взаимодействие компонентов**
- Telegram-бот взаимодействует с системой оценки кода, получая запросы на анализ кода от студентов и преподавателей.
- Модель для анализа кода обрабатывает загруженный код, формирует комментарии на основе заранее заданных критериев для оценки кода и предоставляет результаты анализа модулю генерации отчетов для формирования итогового документа.
- Модель для классификации кода обрабатывает загруженный код, результаты анализа модели анализа кода, классифицирует код и предоставляет результаты модулю генерации отчетов для формирования итогового документа.
- База данных хранит информацию о преподавателях, необходимую для авторизации и отправки отчетов.
- GitHub по запросу через бот загружает код студентов для дальнейшего анализа с помощью моделей машинного обучения.

### 3. Диаграмма последовательностей
![Диаграмма последовательностей (папс)](https://github.com/user-attachments/assets/73d1af59-5a26-4e7f-94f2-03b460ea5ca0)


### 4. Модель БД
![Диаграмма классов БД (папс)](https://github.com/user-attachments/assets/2b69b54d-5d87-4c55-a17c-e8f0a7015e0a)

### 5. Применение основных принципов разработки
*KISS (Keep It Simple, Stupid)*
Логика Telegram-бота минимизирована: он только пересылает запросы на сервер.
Генерация отчетов осуществляется с использованием заранее подготовленных шаблонов.
```
class TelegramBot:
    def __init__(self, api_token, server_url):
        self.api_token = api_token
        self.server_url = server_url

    def handle_request(self, user_input):
        # Простая логика: бот только пересылает запрос на сервер
        response = self.send_to_server(user_input)
        return response

    def send_to_server(self, user_input):
        # Простая передача данных на сервер
        import requests
        payload = {'query': user_input}
        response = requests.post(self.server_url, json=payload)
        return response.json()
```
Чат-бот выполняет только одну задачу — пересылку запросов, избегая излишних функций.

*YAGNI (You Ain’t Gonna Need It)*
Реализованы только функции анализа, классификации и генерации отчетов.
Расширение функционала возможно, но добавление ненужных на данный момент функций исключено.
```
class ReportGenerator:
    def __init__(self, analysis_result):
        self.analysis_result = analysis_result

    def generate(self):
        # Минимально необходимый функционал: генерация текста отчета
        report = f"Код проекта: {self.analysis_result['project_name']}\n"
        report += f"Классификация: {self.analysis_result['classification']}\n"
        report += "Комментарии:\n"
        for comment in self.analysis_result['comments']:
            report += f"- {comment}\n"
        return report
```
Генерация отчета ограничивается текстовым форматом. Поддержка графиков или других сложных форматов не реализуется, так как это не требуется сейчас.

*DRY (Don’t Repeat Yourself)*
Анализ кода вынесен в отдельный модуль, который переиспользуется различными компонентами.
Взаимодействие с базой данных централизовано через ORM.
```
class Database:
    def __init__(self, connection_string):
        self.connection_string = connection_string

    def execute_query(self, query, params=None):
        # Универсальный метод для выполнения запросов
        import sqlite3
        with sqlite3.connect(self.connection_string) as conn:
            cursor = conn.cursor()
            if params:
                cursor.execute(query, params)
            else:
                cursor.execute(query)
            conn.commit()
            return cursor.fetchall()

class UserRepository(Database):
    def get_user_by_id(self, user_id):
        # Повторное использование execute_query
        query = "SELECT * FROM users WHERE id = ?"
        return self.execute_query(query, (user_id,))

class ProjectRepository(Database):
    def get_project_by_id(self, project_id):
        # Повторное использование execute_query
        query = "SELECT * FROM projects WHERE id = ?"
        return self.execute_query(query, (project_id,))
```
Логика выполнения запросов реализована один раз в базовом классе Database и переиспользуется в специализированных классах.

*SOLID*
* S (Single Responsibility): Каждый модуль отвечает только за свою задачу.
```
class CodeAnalyzer:
    def analyze_code(self, code):
        # Выполняет только анализ кода
        issues = []
        if "eval" in code:
            issues.append("Использование eval небезопасно.")
        if len(code.splitlines()) > 100:
            issues.append("Код слишком длинный.")
        return {"issues": issues, "line_count": len(code.splitlines())}
```
Модуль для анализа кода выполняет только анализ, оставляя генерацию отчетов и другие задачи другим модулям.  

* O (Open/Closed): Модули анализа кода можно расширять, добавляя новые критерии проверки, без изменения существующего кода.
``` 
class BaseRule:
    def check(self, code):
        raise NotImplementedError

class EvalRule(BaseRule):
    def check(self, code):
        if "eval" in code:
            return "Использование eval небезопасно."
        return None

class LengthRule(BaseRule):
    def check(self, code):
        if len(code.splitlines()) > 100:
            return "Код слишком длинный."
        return None
class CodeAnalyzer:
    def init(self):
        self.rules = [EvalRule(), LengthRule()]  # Легко добавить новые правила

    def analyze_code(self, code):
        results = []
        for rule in self.rules:
            result = rule.check(code)
            if result:
                results.append(result)
        return results
```        
Модуль расширяем для добавления новых критериев проверки без изменения существующего кода.  

* L (Liskov Substitution):  Различные БД могут заменять друг друга.
```
from abc import ABC, abstractmethod
from typing import Dict, Any

# L (Liskov Substitution)
class DatabaseInterface(ABC):
    @abstractmethod
    def save(self, data: Dict[str, Any]) -> None:
        """Абстрактный метод сохранения данных"""
        pass

class SQLiteDatabase(DatabaseInterface):
    def save(self, data: Dict[str, Any]) -> None:
        """Конкретная реализация сохранения в SQLite"""
        print(f"Сохранено в SQLite: {data}")

class PostgreSQLDatabase(DatabaseInterface):
    def save(self, data: Dict[str, Any]) -> None:
        """Альтернативная реализация для PostgreSQL"""
        print(f"Сохранено в PostgreSQL: {data}")

class DatabaseManager:
    def __init__(self, database: DatabaseInterface):
        self._database = database
    
    def persist_data(self, data: Dict[str, Any]) -> None:
        """Метод для сохранения данных через абстракцию"""
        self._database.save(data)
```
Интерфейс для базы данных позволяет подменять реализацию без изменения логики.  
 
* I (Interface Segregation): Раздельные действия для студентов и преподавателей
```
class StudentAction(ABC):
    @abstractmethod
    def upload_assignment(self, content: str) -> None:
        """Действия студента"""
        pass

class TeacherAction(ABC):
    @abstractmethod
    def review_assignment(self, assignment_id: str) -> None:
        """Действия преподавателя"""
        pass

class LearningPlatform:
    def __init__(self, student_actions: StudentAction, 
                 teacher_actions: TeacherAction):
        self.student = student_actions
        self.teacher = teacher_actions

class TelegramLearningBot(StudentAction, TeacherAction):
    def upload_assignment(self, content: str) -> None:
        print(f"Студент загрузил задание: {content}")
    
    def review_assignment(self, assignment_id: str) -> None:
        print(f"Преподаватель проверил задание {assignment_id}")
```

Разделение интерфейсов для студентов и преподавателей.  

* D (Dependency Inversion): Внедрение различных анализаторов кода
```
class CodeAnalyzer(ABC):
    @abstractmethod
    def analyze(self, code: str) -> str:
        """Абстрактный анализатор кода"""
        pass

class StaticCodeAnalyzer(CodeAnalyzer):
    def analyze(self, code: str) -> str:
        """Конкретная реализация статического анализа"""
        return "Выполнен статический анализ кода"

class DynamicCodeAnalyzer(CodeAnalyzer):
    def analyze(self, code: str) -> str:
        """Конкретная реализация динамического анализа"""
        return "Выполнен динамический анализ кода"

class ReportGenerator:
    def __init__(self, analyzer: CodeAnalyzer):
        self._analyzer = analyzer
    
    def generate_report(self, code: str) -> str:
        """Генерация отчета с использованием внедренного анализатора"""
        analysis_result = self._analyzer.analyze(code)
        return f"Отчет: {analysis_result}"
```
Использование инверсии зависимостей для анализа кода.

### 6. Дополнительные принципы разработки
*1. BDUF (Big Design Up Front/Масштабное проектирование прежде всего)*
Применимость: Частично ограничена

Обоснование для использования:
- Четкая архитектура системы с явно определенными компонентами
- Необходимость интеграции нескольких сложных подсистем (Telegram-бот, ML-модели, GitHub)

Причины ограничения:
- Высокая вероятность изменения требований в процессе разработки
- Риск "переусложнения" архитектуры на ранних этапах
- Потребность в гибкости при разработке ML-компонентов

Решение: Гибридный подход - укрупненное проектирование с итеративной детализацией

*2. SoC (Separation of Concerns/Принцип разделения ответственности)*
Применимость: Полностью применим

Обоснование:
- Четкое разделение компонентов в архитектуре:
- Telegram-бот (интерфейс)
- Система оценки кода (бизнес-логика)
- Модули анализа и классификации (ML-логика)
- База данных (хранение)
- Интеграция с GitHub (внешнее взаимодействие)

Преимущества:
- Независимое развитие и тестирование каждого компонента
- Возможность замены/модернизации отдельных модулей
- Упрощение поддержки и масштабирования системы

*3. PoC (Proof of Concept/Доказательство концепции)*
Применимость: Рекомендуется

Обоснование:
- Проверка feasibility ML-моделей анализа кода
- Валидация интеграционных сценариев (Telegram + GitHub)
- Оценка производительности и точности системы

Ключевые области для PoC:
- Точность ML-модели анализа кода на С#
- Корректность интеграции с GitHub API
- Производительность обработки кода
- User Experience в Telegram-боте

Решение:
- Создание прототипа с минимальным функционалом
- Пилотное тестирование на ограниченной выборке
- Итеративная доработка based on feedback
