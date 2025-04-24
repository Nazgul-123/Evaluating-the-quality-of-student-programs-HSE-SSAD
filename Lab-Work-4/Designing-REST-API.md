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
**URL:** `/api/v1/students`  

**Описание:** Возвращает список студентов.  
- **Параметры запроса:** Отсутствуют  
- **Тело запроса (Body):** Не требуется  
**Пример успешного ответа (200):**  
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
**При отсутствии студентов(200):**  
```json
{
    "info": "Student list is empty"
}
```



#### 2. **Добавление нового студента**
**Метод:** POST  
**URL:** `/api/v1/students`   

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
**Пример успешного ответа (201):**
```json
{
    "id": 3,
    "name": "Петр Петров",
    "email": "petrov@example.com"
}
```
**Пример ответа при ошибке добавления (400 - некорректные данные):**
```json
{
    "error": "Invalid input data"
}
```

#### 3. **Обновление данных студента**
**Метод:** PUT  
**URL:** `/api/v1/students/<student_id>` , например, `/api/v1/students/1`

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
**Пример успешного ответа (200):**  
```json
{
  "id": 1,
  "name": "Иван Иванович",
  "email": "ivanovich@example.com"
}
```
**Пример ответа при отсутствии данных для обновления (400):**  
```json
{
    "error": "Invalid input data"
}
```
**Пример ответа при отсутствии студента, чьи данные хотели обновить (404):**  
```json
{
    "error": "Student not found"
}
```

#### 4. **Удаление студента**
**Метод:** DELETE  
**URL:** `/api/v1/students/<student_id>`, например, `/api/v1/students/2`  

**Описание:** Удаляет студента из базы данных и отчеты по его работам, если такие были.  
- **Параметры запроса:** Отсутствуют  
- **Тело запроса (Body):** Не требуется  
**Пример успешного ответа (200):**  
```json
{
  "message": "Студент и его отчеты успешно удалены"
}
```
**Пример ответа при отсутствии студента, чьи данные хотели удалить (404):**  
```json
{
    "error": "Student not found"
}
```

#### 5. **Получение списка отчетов**
**Метод:** GET  
**URL:** `/api/v1/reports`  

**Описание:** Возвращает все отчеты.  
- **Параметры запроса:** Отсутствуют  
- **Тело запроса (Body):** Не требуется  
**Пример успешного ответа (200):**  
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
**Пример ответа при отсутствии отчетов(200):**  
```json
{
    "info": "Report list is empty"
}
```

#### 6. **Добавление нового отчета**
**Метод:** POST  
**URL:**  `/api/v1/reports`  

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
**Пример успешного ответа при добавлении (201):**  
```json
{
   "code_quality": "in_progress",
    "comments": [],
    "report_id": 2,
    "student_id": 1
}
```
**Пример ответа при ошибки при добавлении, если студент не найден(404):**  
```json
{
   "error" : "Student not found"
}
```
**Пример ответа при ошибки при добавлении, если данные некорректны(400):**  
```json
{
   "error" : "Invalid input data"
}
``` 

### 7. **Получение отчета по ID**
**Метод:** `GET`  
**URL:** `/api/v1/reports/<report_id>`, например, `/api/v1/reports/1`

**Описание:** Возвращает отчет по его идентификатору
- **Параметры запроса:** Отсутствуют  
- **Тело запроса (Body):** Не требуется  
**Пример успешного ответа (200):**  
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
**Пример ответа с ошибкой, если отчет с таким идентификатором не найден(404):**  
```json
{
    "error": "Report not found"
}
```

### 8. **Удаление отчета**
**Метод:** `DELETE`  
**URL:** `/api/v1/reports/<report_id>`, например, `/api/v1/reports/2`

**Описание:** Удаляет отчет из БД по идентификатору
- **Параметры запроса:** Отсутствуют  
- **Тело запроса (Body):** Не требуется  
**Пример успешного ответа (200):**  
```json
{
    "message": "Отчет успешно удален"
}
```
**Пример ответа при ошибке, если отчет с таким идентификатором не найедн (404):**  
```json
{
    {"error": "Report not found"}
}
```


### 9. **Получение студента по ID**
**Метод:** `GET`  
**URL:** `/api/v1/students/<student_id>`, например, `/api/v1/students/1`

**Описание:** Возвращает данные студента по его идентификатору
- **Параметры запроса:** Отсутствуют  
- **Тело запроса (Body):** Не требуется  
**Пример ответа при отсутствии студента с таким идентификатором (404):**  
```json
{
    "error": "Student not found"
}
``` 
**Пример ответа при наличии такого идентификатора (200):**  
```json
{
    "email": "ivanovich@example.com",
    "id": 1,
    "name": "Иван Иванович"
}
```

#### - **Дополнительные обработчики ошибок**
**Описание:** Обработка ошибки, которая возникает в критических сбоях в работе сервера.   
**Пример ответа (500):**  
```json
[
     {"error": "Internal Server Error"}
]
```

---

## Тестирование API  

Тестирование проводилось с использованием **Postman**.
![screenshot from Postman](https://github.com/user-attachments/assets/3091f9dc-421a-4d50-ac3f-15802327adff)

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
    if not students:
        return {"info": "Student list is empty"}, 200
    return jsonify(students), 200

# Создание нового студента
@app.route('/api/v1/students', methods=['POST'])
def create_student():
    data = request.get_json()
    if not data or "name" not in data or "email" not in data:
        return {"error": "Invalid input data"}, 400 #Bad Request («неправильный, некорректный запрос»)
    new_student = {"id": len(students) + 1, "name": data['name'], "email": data['email']}
    students.append(new_student)
    return jsonify(new_student), 201 #Created («создано»)

# Обновление данных студента
@app.route('/api/v1/students/<int:student_id>', methods=['PUT'])
def update_student(student_id):
    data = request.get_json()
    if not data:
        return {"error": "Invalid input data"}, 400  # Bad Request («неправильный, некорректный запрос»)
    for student in students:
        if student['id'] == student_id:
            student.update(data)
            return jsonify(student), 200
    return {"error": "Student not found"}, 404

# Удаление студента
@app.route('/api/v1/students/<int:student_id>', methods=['DELETE'])
def delete_student(student_id):
    global students, reports
    student_exists = any(student["id"] == student_id for student in students)

    if not student_exists:
        return {"error": "Student not found"}, 404

    students = [student for student in students if student['id'] != student_id]
    reports = [report for report in reports if report['student_id'] != student_id]
    return {"message": "Студент и его отчеты успешно удалены"}, 200

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
    if not reports:
        return {"info": "Report list is empty"}, 200
    return jsonify(reports), 200

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
    if not data or "student_id" not in data:
        return {"error": "Invalid input data"}, 400 # Bad Request («неправильный, некорректный запрос»)

    student_exists = any(student["id"] == data["student_id"] for student in students)
    if not student_exists:
        return {"error": "Student not found"}, 404

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
    report_exists = any(report["report_id"] == report_id for report in reports)

    if not report_exists:
        return {"error": "Report not found"}, 404

    reports = [report for report in reports if report['report_id'] != report_id]
    return {"message": "Отчет успешно удален"}, 200

@app.errorhandler(500)
def internal_server_error(error):
    return jsonify({"error": "Internal Server Error"}), 500

if __name__ == '__main__':
    app.run(debug=True)
```
