# Лабораторная работа №4  
### Тема: Проектирование REST API  
### Цель работы: Получить опыт проектирования программного интерфейса.

---

## Документация по API  

**Описание сервиса**  
Сервис предназначен для оценки программного кода студентов по дисциплине "Программирование". API реализует функции взаимодействия с пользователями (студенты и преподаватели), анализа кода, генерации отчетов и получения данных из GitHub.

### Проектные решения  

1. **Архитектурный стиль**: REST API для обеспечения стандартизации и удобства интеграции.  
2. **Методы HTTP**: Используются методы GET, POST, PUT, DELETE для реализации CRUD-операций.  
3. **Идентификация пользователей**: Авторизация через токены.  
4. **Формат данных**: Передача данных в формате JSON для удобства работы с клиентами.  
5. **Обработка ошибок**: Реализована единая структура ошибок с кодами и описанием.  
6. **Структура URL**: Четкое разделение по ресурсам, например, `/students`, `/teachers`, `/reports`.  
7. **Статус-коды HTTP**: Использование стандартных кодов для ответа сервера (200, 201, 400, 404, 500).  
8. **Версионирование**: API имеет версию в URL (`/api/v1/`), чтобы обеспечить обратную совместимость.  

---

### Эндпоинты API  

#### 1. **Получение списка студентов**
**Метод:** GET  
**URL:** `http://127.0.0.1:5000/api/v1/students`  
**Описание:** Возвращает список студентов.  
- **Параметры запроса:** Отсутствуют  
- **Тело запроса (Body):** Не требуется  
**Пример ответа:**  
```json
[
  {
    "id": 1,
    "name": "Иван Иванов",
    "email": "ivanov@example.com"
  },
  {
    "id": 2,
    "name": "Мария Петрова",
    "email": "petrova@example.com"
  }
]
```

#### 2. **Добавление нового студента**
**Метод:** POST  
**URL:** `http://127.0.0.1:5000/api/v1/students`   
**Описание:** Добавляет нового студента в базу данных.  
- **Заголовки (Headers):**
  - Content-Type: `application/json`
- **Тело запроса (Body):** (в формате JSON)
**Тело запроса:**  
```json
{
  "name": "Петр Петров",
  "email": "petrov@example.com"
}
```
**Пример ответа:**
```json
{
    "id": 3,
    "name": "Петр Петров",
    "email": "petrov@example.com"
}
```
#### 3. **Обновление данных студента**
**Метод:** PUT  
**URL:** `http://127.0.0.1:5000/api/v1/students/<student_id>` , например, `http://127.0.0.1:5000/api/v1/students/1`
**Описание:** Обновляет данные о студенте по его идентификатору.  
- **Заголовки (Headers):**
  - Content-Type: `application/json`
- **Тело запроса (Body):** (в формате JSON)
**Тело запроса:**  
```json
{
  "name": "Иван Иванович",
  "email": "ivanovich@example.com"
}
```
**Пример ответа:**  
```json
{
  "id": 1,
  "name": "Иван Иванович",
  "email": "ivanovich@example.com"
}
```
#### 4. **Удаление студента**
**Метод:** DELETE  
**URL:** `http://127.0.0.1:5000/api/v1/students/<student_id>`, например, `http://127.0.0.1:5000/api/v1/students/2`  
**Описание:** Удаляет студента из базы данных и отчеты по его работам, если такие были.  
- **Параметры запроса:** Отсутствуют  
- **Тело запроса (Body):** Не требуется  
**Пример ответа:**  
```json
{
  "message": "Студент и его отчеты успешно удалены"
}
```
#### 5. **Получение списка отчетов**
**Метод:** GET  
**URL:** `http://127.0.0.1:5000/api/v1/reports`  
**Описание:** Возвращает все отчеты.  
- **Параметры запроса:** Отсутствуют  
- **Тело запроса (Body):** Не требуется  
**Пример ответа:**  
```json
[
{
  "student_id": 1,
  "code_quality": "excellent",
  "comments": [
    "Код хорошо структурирован",
    "Следует добавить больше комментариев"]
}
]
```
#### 6. **Добавление нового отчета**
**Метод:** POST  
**URL:**  `http://127.0.0.1:5000/api/v1/reports`  
**Описание:** Создает отчет по загруженному коду.  
- **Заголовки (Headers):**
  - Content-Type: `application/json`
- **Тело запроса (Body):** (в формате JSON)
**Тело запроса:**  
```json
{
  "student_id": 1
}
```
**Пример ответа:**  
```json
{
   "code_quality": "in_progress",
    "comments": [],
    "report_id": 2,
    "student_id": 1
}
``` 
### 7. **Получение отчета по ID**
**Метод:** `GET`  
**URL:** `http://127.0.0.1:5000/api/v1/reports/<report_id>`, например, `http://127.0.0.1:5000/api/v1/reports/1`
**Описание:** Возвращает отчет по его идентификатору
- **Параметры запроса:** Отсутствуют  
- **Тело запроса (Body):** Не требуется  
**Пример ответа:**  
```json
{
    "code_quality": "good",
    "comments": [
        "Хорошо структурированный код",
        "Добавьте больше комментариев"
    ],
    "report_id": 1,
    "student_id": 1
}
``` 
### 8. **Удаление отчета**
**Метод:** `DELETE`  
**URL:** `http://127.0.0.1:5000/api/v1/reports/<report_id>`, например, `http://127.0.0.1:5000/api/v1/reports/2`
**Описание:** Удаляет отчет из БД по идентификатору
- **Параметры запроса:** Отсутствуют  
- **Тело запроса (Body):** Не требуется  
**Пример ответа:**  
```json
{
    "message": "Отчет успешно удален"
}
``` 
### 9. **Получение студента по ID**
**Метод:** `GET`  
**URL:** `http://127.0.0.1:5000/api/v1/students/<student_id>`, например, `http://127.0.0.1:5000/api/v1/students/1`
**Описание:** Возвращает данные студента по его идентификатору
- **Параметры запроса:** Отсутствуют  
- **Тело запроса (Body):** Не требуется  
**Пример ответа при отсутствии такого идентификатора:**  
```json
{
    "error": "Student not found"
}
``` 
**Пример ответа при наличии такого идентификатора:**  
```json
{
    "email": "ivanovich@example.com",
    "id": 1,
    "name": "Иван Иванович"
}
``` 
---

## Тестирование API  

Тестирование проводилось с использованием **Postman**.

---

## Реализация API  

Для реализации использовался Python (Flask).  

**Пример кода реализации**:  
```
from flask import Flask, request, jsonify

app = Flask(__name__)

# Данные студентов
students = [
    {"id": 1, "name": "Иван Иванов", "email": "ivanov@example.com"}
]

# Данные отчетов
reports = [
    {"report_id": 1, "student_id": 1, "code_quality": "good", "comments": ["Хорошо структурированный код", "Добавьте больше комментариев"]}
]

# Получение списка студентов
@app.route('/api/v1/students', methods=['GET'])
def get_students():
    return jsonify(students)

# Создание нового студента
@app.route('/api/v1/students', methods=['POST'])
def create_student():
    data = request.get_json()
    new_student = {"id": len(students) + 1, "name": data['name'], "email": data['email']}
    students.append(new_student)
    return jsonify(new_student), 201

# Обновление данных студента
@app.route('/api/v1/students/<int:student_id>', methods=['PUT'])
def update_student(student_id):
    data = request.get_json()
    for student in students:
        if student['id'] == student_id:
            student.update(data)
            return jsonify(student)
    return {"error": "Student not found"}, 404

# Удаление студента
@app.route('/api/v1/students/<int:student_id>', methods=['DELETE'])
def delete_student(student_id):
    global students, reports
    students = [student for student in students if student['id'] != student_id]
    reports = [report for report in reports if report['student_id'] != student_id]
    return {"message": "Студент и его отчеты успешно удалены"}

# Получение студента по ID
@app.route('/api/v1/students/<int:student_id>', methods=['GET'])
def get_student_by_id(student_id):
    for student in students:
        if student['id'] == student_id:
            return jsonify(student)
    return {"error": "Student not found"}, 404

# Получение списка отчетов
@app.route('/api/v1/reports', methods=['GET'])
def get_reports():
    return jsonify(reports)

# Получение отчета по ID
@app.route('/api/v1/reports/<int:report_id>', methods=['GET'])
def get_report(report_id):
    for report in reports:
        if report['report_id'] == report_id:
            return jsonify(report)
    return {"error": "Report not found"}, 404

# Добавление нового отчета
@app.route('/api/v1/reports', methods=['POST'])
def create_report():
    data = request.get_json()
    new_report = {
        "report_id": len(reports) + 1,
        "student_id": data['student_id'],
        "code_quality": "in_progress",
        "comments": []
    }
    reports.append(new_report)
    return jsonify(new_report), 201

# Удаление отчета
@app.route('/api/v1/reports/<int:report_id>', methods=['DELETE'])
def delete_report(report_id):
    global reports
    reports = [report for report in reports if report['report_id'] != report_id]
    return {"message": "Отчет успешно удален"}

if __name__ == '__main__':
    app.run(debug=True)
```
