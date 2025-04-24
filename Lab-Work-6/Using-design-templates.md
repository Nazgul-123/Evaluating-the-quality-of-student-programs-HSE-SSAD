# Лабораторная работа №6

## Тема: Использование шаблонов проектирования при разработке информационной системе для автоматизированной оценки кода.

## Цель работы
Получить опыт применения шаблонов проектирования при написании кода программной системы.

---

## Шаблоны проектирования GoF

### Порождающие шаблоны
1. **Factory Method (Фабричный метод)**  
_Описание:_
Определяет интерфейс для создания объекта, но оставляет подклассам решение о том, какой класс инстанцировать. Фабричный метод позволяет классу делегировать инстанцирование подклассам.

![image](https://github.com/user-attachments/assets/9b41026c-e806-4fd7-a493-19c8c81e49f6)

_Применение в системе:_
Этот шаблон можно использовать для создания различных анализаторов кода (статический анализатор, анализатор с LLM).

_Реализация:_
```python
from abc import ABC, abstractmethod

# Базовый класс анализатора
class CodeAnalyzer(ABC):
    @abstractmethod
    def analyze(self, code):
        pass

# Конкретные реализации анализаторов
class StaticAnalyzer(CodeAnalyzer):
    def analyze(self, code):
        return "Статический анализ выполнен"

class LLMBasedAnalyzer(CodeAnalyzer):
    def analyze(self, code):
        return "Оценка LLM завершена"

# Фабрика
class AnalyzerFactory:
    @staticmethod
    def get_analyzer(analyzer_type):
        if analyzer_type == "static":
            return StaticAnalyzer()
        elif analyzer_type == "llm":
            return LLMBasedAnalyzer()
        elif analyzer_type == "composite":
            return CompositeAnalyzer(StaticAnalyzer(), LLMBasedAnalyzer())
        else:
            raise ValueError("Неизвестный тип анализатора")

# Использование фабрики
analyzer = AnalyzerFactory.get_analyzer("static")
print(analyzer.analyze("sample code"))
```
2. **Одиночка (Singleton)**  
_Описание:_
Гарантирует, что в системе существует только один экземпляр класса, и предоставляет глобальную точку доступа к нему.

![image](https://github.com/user-attachments/assets/cd26e7c0-4441-4cd5-b524-3ecee5f1a40e)

_Применение в системе:_
Используется для управления подключением к RabbitMQ, чтобы избежать создания множества подключений.

_Реализация:_
```python
import asyncio
import aiormq

class RabbitMQConnection:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance.connection = None
        return cls._instance

    async def get_connection(self):
        if self.connection is None:
            try:
                self.connection = await aiormq.connect("amqp://rabbitmq")
            except Exception as e:
                print(f"Ошибка подключения к RabbitMQ: {e}")
                raise
        return self.connection

# Использование Singleton
async def main():
    rabbitmq = RabbitMQConnection()
    connection = await rabbitmq.get_connection()
    print("Подключение установлено:", connection)

asyncio.run(main())
```

### Структурные шаблоны
1. **Adapter (Адаптер)**  
_Описание:_
Позволяет объекту с несовместимым интерфейсом работать с другим объектом без изменения его кода. Преобразует интерфейс одного класса в интерфейс другого, который ожидают клиенты.

![image](https://github.com/user-attachments/assets/711eb64f-4044-417a-9086-1ef6fb745b16)

_Применение в системе:_
Используется для адаптации API различных анализаторов (например, статический анализатор и LLM имеют разные методы вызова).

_Реализация:_
```python
class StaticAnalyzer:
    def analyze_static(self, code):
        return "Статический анализ завершен"

class LLMBasedAnalyzer:
    def analyze_llm(self, code):
        return "Оценка LLM завершена"

class AnalyzerAdapter:
    def __init__(self, analyzer):
        self.analyzer = analyzer

    def analyze(self, code):
        if isinstance(self.analyzer, StaticAnalyzer):
            return self.analyzer.analyze_static(code)
        elif isinstance(self.analyzer, LLMBasedAnalyzer):
            return self.analyzer.analyze_llm(code)

# Использование
adapter = AnalyzerAdapter(StaticAnalyzer())
print(adapter.analyze("sample code"))
```

2. **Facade (Фасад)**  
_Описание:_
Предоставляет упрощенный интерфейс для сложной подсистемы, скрывая её детали.

![image](https://github.com/user-attachments/assets/0a0a7b72-a205-4090-8756-b458cf358549)

_Применение в системе:_
Используется для сокрытия сложности взаимодействия с RabbitMQ через упрощенный API.

_Реализация:_
```python
class RabbitMQFacade:
    async def send_message(queue, message):
        connection = await aiormq.connect(RABBITMQ_URL)
        channel = await connection.channel()
        await channel.queue_declare(queue=queue)
        await channel.basic_publish(
            exchange="",
            routing_key=queue,
            body=json.dumps(message).encode()
        )
        await connection.close()
    async def receive_message(self, queue):
        connection = await aiormq.connect(RABBITMQ_URL)
        channel = await connection.channel()
        await channel.queue_declare(queue=queue)
        message = await channel.basic_get(queue=queue, no_ack=True)
        await connection.close()
        return message
```


3. **Proxy (Заместитель)**   
_Описание:_
Предоставляет объект-заместитель для контроля доступа или выполнения дополнительных операций перед обращением к основному объекту.Изменение требований и эволюция системы могут вызвать необходимость внесения серьезных архитектурных изменений. Если на ранних этапах некая операция выполнялась на стороне клиента или же приложение состояло из одного процесса, то со временем исполнение операции может быть перенесено на сервер, а приложение разбито на несколько процессов. В результате возникает задача взаимодействия с удаленным процессом, реализация которой должна быть максимально похожей на локальное взаимодействие. Именно для таких целей предназначен паттерн «Заместитель».

![image](https://github.com/user-attachments/assets/04d8990a-311b-4597-a293-5b414644a031)

Участники  
* Subject: определяет общий интерфейс для Proxy и RealSubject. Поэтому Proxy может использоваться вместо RealSubject  
* RealSubject: представляет реальный объект, для которого создается прокси  
* Proxy: заместитель реального объекта. Хранит ссылку на реальный объект, контролирует к нему доступ, может управлять его созданием и удалением.  
* Client: использует объект Proxy для доступа к объекту RealSubject  

_Применение в системе:_
Используется для проверки прав доступа при запросах к API.

_Реализация:_
```python
class CodeAnalyzerProxy:
    def __init__(self, user_role, analyzer):
        self.user_role = user_role
        self.analyzer = analyzer

    def analyze(self, code):
        if self.user_role not in ["admin", "teacher"]:
            return "Доступ запрещен"
        return self.analyzer.analyze(code)

# Использование
proxy = CodeAnalyzerProxy("user", StaticAnalyzer())
print(proxy.analyze("sample code"))  # Доступ запрещен
```

4. **Composite (Компоновщик)**   
_Описание:_
Позволяет работать с группами объектов так же, как с отдельными объектами. Компонует объекты в древовидные структуры для представления иерархий «часть — целое». Позволяет клиентам единообразно трактовать индивидуальные и составные объекты.

![image](https://github.com/user-attachments/assets/d24e5538-74fc-4713-917b-05733ee7b9bb)

_Применение в системе:_
Позволяет объединять результаты анализа кода в единую структуру.

_Реализация:_
```python
class AnalysisResult:
    def display(self):
        pass

class SingleAnalysis(AnalysisResult):
    def __init__(self, result):
        self.result = result

    def display(self):
        return self.result

class CompositeAnalysis(AnalysisResult):
    def __init__(self):
        self.results = []

    def add(self, result):
        self.results.append(result)

    def display(self):
        return "\n".join(result.display() for result in self.results)

# Использование
composite = CompositeAnalysis()
composite.add(SingleAnalysis("Статический анализ: ОК"))
composite.add(SingleAnalysis("LLM анализ: ОК"))

print(composite.display())
```

### Поведенческие шаблоны
1. **Observer (Наблюдатель)**   
_Описание:_
Позволяет объектам подписываться на события и получать уведомления при их изменении. Определяет зависимость типа «один ко многим» между объектами таким образом, что при изменении состояния одного объекта все зависящие от него оповещаются об этом и автоматически обновляются.

![image](https://github.com/user-attachments/assets/88f8d009-e81b-4310-a4b0-e64ab6730b65)

_Применение в системе:_
Используется для подписки на результаты анализа кода.

_Реализация:_
```python
class Observer:
    def update(self, message):
        pass

class CodeAnalyzer:
    def __init__(self):
        self.subscribers = []

    def subscribe(self, observer):
        self.subscribers.append(observer)

    def notify(self, message):
        for observer in self.subscribers:
            observer.update(message)

class Logger(Observer):
    def update(self, message):
        print(f"Лог: {message}")

# Использование
analyzer = CodeAnalyzer()
logger = Logger()
analyzer.subscribe(logger)
analyzer.notify("Анализ завершен")
```

2. **Command (Команда)**   
_Описание:_
Инкапсулирует запрос как объект, позволяя тем самым задавать параметры клиентов для обработки соответствующих запросов, ставить запросы в очередь или протоколировать их, а также поддерживать отмену операций.
Паттерн «Команда» позволяет спрятать действие в объекте и отвязать источник этого действия от места его исполнения.

![image](https://github.com/user-attachments/assets/8e36c413-de7e-47f5-8c4e-8fd25d4ef6d9)

_Применение в системе:_
Используется для отправки запроса на анализ кода через RabbitMQ.

_Реализация:_
```python
class AnalyzeCommand:
    def __init__(self, analyzer, code):
        self.analyzer = analyzer
        self.code = code

    def execute(self):
        return self.analyzer.analyze(self.code)

# Использование
command = AnalyzeCommand(StaticAnalyzer(), "sample code")
print(command.execute())
```

3. **Chain of Responsibility (Цепочка обязанностей)**   
_Описание:_
позволяет передавать запросы по цепочке обработчиков. Каждый обработчик решает, может ли он обработать запрос, и либо обрабатывает его, либо передает следующему обработчику в цепочке.
  
![image](https://github.com/user-attachments/assets/00436dc8-1c26-40bd-846d-43d87f047836)

_Применение в системе:_
Применяется при проверке кода, передавая его через несколько сервисов анализа.

_Реализация:_
```python
class CodeAnalyzer:
    def __init__(self, next_analyzer=None):
        self.next_analyzer = next_analyzer

    def analyze(self, code):
        result = self._process(code)
        if self.next_analyzer:
            return self.next_analyzer.analyze(code) + "\n" + result
        return result

    def _process(self, code):
        raise NotImplementedError

class StaticAnalyzer(CodeAnalyzer):
    def _process(self, code):
        return "Статический анализ: OK"

class LLMAnalyzer(CodeAnalyzer):
    def _process(self, code):
        return "LLM анализ: OK"

analyzer = StaticAnalyzer(LLMAnalyzer())
result = analyzer.analyze(code)
```


4. **Strategy (Стратегия)**   
_Описание:_
Позволяет определять семейство алгоритмов, инкапсулировать каждый из них и делать их взаимозаменяемыми. Это позволяет изменять алгоритмы независимо от клиента, который их использует.

![image](https://github.com/user-attachments/assets/705ce526-6fd6-4579-b524-813a724498e7)

_Применение в системе:_
Позволяет переключаться между различными алгоритмами анализа кода.

_Реализация:_
```python
class AnalysisStrategy:
    def analyze(self, code):
        pass

class StaticAnalysisStrategy(AnalysisStrategy):
    def analyze(self, code):
        return "Статический анализ: OK"

class LLMAnalysisStrategy(AnalysisStrategy):
    def analyze(self, code):
        return "Оценка LLM: OK"

# Использование стратегии
def analyze_code(code, strategy: AnalysisStrategy):
    return strategy.analyze(code)

code = "sample code"
static_result = analyze_code(code, StaticAnalysisStrategy())
llm_result = analyze_code(code, LLMAnalysisStrategy())


print(static_result)
print(llm_result)
```


5. **Mediator (Посредник)**   
_Описание:_
Позволяет уменьшить связанность между объектами, перенося логику их взаимодействия в отдельный объект-посредник.

![image](https://github.com/user-attachments/assets/67a9bf9f-4a8a-488e-8b0e-6bef3dbfb674)

_Применение в системе:_
Используется для управления взаимодействием между сервисами через брокер сообщений.

_Реализация:_
```python
class Mediator:
    def __init__(self):
        self.components = []

    def register(self, component):
        self.components.append(component)

    def send(self, message, sender):
        for component in self.components:
            if component != sender:
                component.receive(message)

class Component:
    def __init__(self, mediator):
        self.mediator = mediator
        mediator.register(self)

    def send(self, message):
        self.mediator.send(message, self)

    def receive(self, message):
        print(f"Получено сообщение: {message}")

# Использование
mediator = Mediator()
comp1 = Component(mediator)
comp2 = Component(mediator)

comp1.send("Привет, мир!")
```

6. **State (Состояние)**   
_Описание:_
Позволяет объекту изменять свое поведение при изменении его внутреннего состояния. Вместо того чтобы использовать множество условных операторов для управления поведением объекта в зависимости от его состояния, шаблон инкапсулирует каждое состояние в отдельный класс и делегирует выполнение операций объекту текущего состояния. Позволяет объекту варьировать свое поведение в зависимости от внутреннего состояния. Извне создается впечатление, что изменился класс объекта. 
Паттерн «Состояние» предполагает выделение базового класса или интерфейса для всех допустимых операций и наследника для каждого возможного состояния.

![image](https://github.com/user-attachments/assets/4316f9e1-7919-423e-8bc3-ea3ce818e526)

_Применение в системе:_
Может быть использован для управления процессом проверки кода. код может находиться в разных состояниях:
* Ожидание анализа — код только что загружен и ожидает начала анализа.  
* Анализ выполняется — код находится в процессе анализа.  
* Анализ завершен — анализ кода завершен, и результаты готовы.  
* Ошибка анализа — в процессе анализа произошла ошибка.    
Каждое состояние будет иметь свою логику обработки, и система будет переключаться между состояниями в зависимости от текущего этапа проверки.

_Реализация:_
```python
from abc import ABC, abstractmethod

# Базовый класс состояния
class CodeAnalysisState(ABC):
    @abstractmethod
    def handle(self, context):
        pass

# Конкретные состояния
class PendingState(CodeAnalysisState):
    def handle(self, context):
        print("Код ожидает анализа...")
        context.set_state(InProgressState())

class InProgressState(CodeAnalysisState):
    def handle(self, context):
        print("Код анализируется...")
        # Имитация анализа
        if analysis_successful():  # Предположим, что это функция, проверяющая успешность анализа
            context.set_state(CompletedState())
        else:
            context.set_state(ErrorState())

class CompletedState(CodeAnalysisState):
    def handle(self, context):
        print("Анализ кода завершен. Результаты готовы.")
        # Здесь можно добавить логику для отправки результатов

class ErrorState(CodeAnalysisState):
    def handle(self, context):
        print("Ошибка при анализе кода.")
        # Здесь можно добавить логику для обработки ошибки

# Контекст (объект, состояние которого изменяется)
class CodeAnalysisContext:
    def __init__(self):
        self._state = PendingState()

    def set_state(self, state):
        self._state = state

    def request(self):
        self._state.handle(self)

# Вспомогательная функция для имитации успешности анализа
def analysis_successful():
    # В реальной системе здесь будет логика проверки успешности анализа
    return True  # Или False, если анализ завершился ошибкой

# Использование
context = CodeAnalysisContext()
context.request()  # Код ожидает анализа...
context.request()  # Код анализируется...
context.request()  # Анализ кода завершен. Результаты готовы.
```

---

## Шаблоны проектирования GRASP

### Роли (обязанности) классов  
1. **Controller (Контроллер)** - класс чат-бота управляет логикой обработки запросов.
_Проблема:_ Кто должен отвечать за обработку входящих запросов и управление потоком выполнения?

_Решение:_ Класс чат-бота (например, ChatBotController) берет на себя роль контроллера, управляя логикой обработки запросов пользователя и координацией взаимодействия между другими компонентами системы.

_Пример кода:_
```python
class ChatBotController:
    def handle_request(self, user_input):
        analysis_result = self.analyze_code(user_input)
        report = self.generate_report(analysis_result)
        return report
```
_Результаты:_ Упрощается управление логикой приложения, обеспечивается четкое разделение обязанностей.

_Связь с другими паттернами:_ Контроллер часто взаимодействует с Information Expert для получения данных и Creator для создания объектов.

2. **Creator (Создатель)** - управление созданием объектов отчетов и анализа кода.  
_Проблема:_ Кто должен отвечать за создание объектов, таких как отчеты или результаты анализа?

_Решение:_ Класс, отвечающий за генерацию отчетов (например, ReportGenerator), берет на себя роль создателя.

_Пример кода:_
```python
class ReportGenerator:
    def create_report(self, analysis_data):
        return Report(analysis_data)
```
_Результаты:_ Упрощается процесс создания объектов, обеспечивается инкапсуляция логики создания.

_Связь с другими паттернами:_ Creator часто взаимодействует с Information Expert для получения данных, необходимых для создания объектов.

3. **Information Expert (Информационный эксперт)** - класс, обрабатывающий результаты анализа.  
_Проблема:_ Кто должен отвечать за хранение и обработку данных, необходимых для выполнения задачи?

_Решение:_ Класс, который имеет доступ к данным, необходимых для анализа (например, CodeAnalyzer), берет на себя роль информационного эксперта.

_Пример кода:_
```python
class CodeAnalyzer:
    def analyze(self, code):
        # Логика анализа кода
        return analysis_result
```
_Результаты:_ Упрощается доступ к данным, обеспечивается четкое разделение обязанностей.

_Связь с другими паттернами:_ Information Expert часто взаимодействует с Creator для создания объектов на основе данных.

4. **Low Coupling (Слабая связанность)** - обеспечивается за счет разделения сервисов и использования очередей RabbitMQ для минимизации связности.  
_Проблема:_ Как минимизировать зависимости между классами, чтобы изменения в одном классе не влияли на другие?

_Решение:_ Использование очередей RabbitMQ для асинхронного взаимодействия между сервисами, что уменьшает прямые зависимости между ними.

_Пример кода:_
```python
class RabbitMQService:
    def send_message(self, queue_name, message):
        # Логика отправки сообщения в очередь
        pass

    def receive_message(self, queue_name):
        # Логика получения сообщения из очереди
        pass
```
_Результаты:_ Упрощается поддержка и модификация системы, уменьшается риск возникновения ошибок при изменениях.

_Связь с другими паттернами:_ Low Coupling часто достигается через использование Indirection.

5. **High Cohesion (Высокая связность)** - каждое приложение отвечает за конкретную задачу, каждый сервис отвечает за часть решения(работа GitHub, анализ, генерация отчета, работа с ботом).  
_Проблема:_ Как обеспечить, чтобы каждый класс отвечал за одну конкретную задачу?

_Решение:_ Каждый сервис (например, GitHubService, CodeAnalyzer, ReportGenerator) отвечает за конкретную задачу, что обеспечивает высокую связность.

_Пример кода:_
```python
class GitHubService:
    def fetch_repository(self, repo_url):
        # Логика получения данных из GitHub
        pass
```
_Результаты:_ Упрощается понимание и поддержка кода, уменьшается вероятность ошибок.

_Связь с другими паттернами:_ High Cohesion часто достигается через Low Coupling.

### Принципы разработки  
1. **Polymorphism (Полиморфизм)** - позволяет использовать общий интерфейс для различных типов анализа кода.  
_Проблема:_ Как обрабатывать различные типы анализа кода, не изменяя существующий код?

_Решение:_ Использование полиморфизма через интерфейсы или абстрактные классы для анализаторов.

_Пример кода:_
```python
class CodeAnalyzer(ABC):
    @abstractmethod
    def analyze(self, code):
        pass

class PythonAnalyzer(CodeAnalyzer):
    def analyze(self, code):
        # Логика анализа Python кода
        pass
```
_Результаты:_ Упрощается добавление новых типов анализаторов, обеспечивается гибкость системы.

_Связь с другими паттернами:_ Полиморфизм часто используется в сочетании с Protected Variations.

2. **Indirection (Косвенность)** - реализована через посредник RabbitMQ.  
_Проблема:_ Как уменьшить прямые зависимости между компонентами системы?

_Решение:_ Использование посредника (например, RabbitMQ) для взаимодействия между сервисами.

_Пример кода:_
```python
class RabbitMQService:
    def send_message(self, queue_name, message):
        # Логика отправки сообщения в очередь
        pass

    def receive_message(self, queue_name):
        # Логика получения сообщения из очереди
        pass
```
_Результаты:_ Упрощается поддержка и модификация системы, уменьшается риск возникновения ошибок при изменениях.

_Связь с другими паттернами:_ Indirection часто используется для достижения Low Coupling.

3. **Protected Variations (Защищенные вариации)** - обеспечивает гибкость системы за счет использования интерфейсов для анализаторов.  
_Проблема:_ Как защитить систему от изменений в реализации анализаторов?

_Решение:_ Использование интерфейсов для анализаторов, что позволяет изменять реализацию без изменения кода, который их использует.

_Пример кода:_
```python
class CodeAnalyzer(ABC):
    @abstractmethod
    def analyze(self, code):
        pass

class JavaAnalyzer(CodeAnalyzer):
    def analyze(self, code):
        # Логика анализа Java кода
        pass
```
_Результаты:_ Упрощается добавление новых типов анализаторов, обеспечивается гибкость системы.

_Связь с другими паттернами:_ Protected Variations часто используется в сочетании с Polymorphism.

### Свойство программы (цель)  
1. **Increased Reusability (Повышенная повторяемость)** - сервисы можно использовать в разных контекстах, не меняя их код.
_Проблема:_ Как сделать сервисы пригодными для использования в разных контекстах без изменения их кода?

_Решение:_ Использование модульной архитектуры и интерфейсов, что позволяет использовать сервисы в разных контекстах.

_Пример кода:_
```python
class ReportGenerator:
    def generate(self, analysis_data):
        # Логика генерации отчета
        pass
```
_Результаты:_ Упрощается повторное использование кода, уменьшается дублирование.

_Связь с другими паттернами:_ Increased Reusability часто достигается через Low Coupling и High Cohesion.

