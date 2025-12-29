# REST API: Регистрация пользователя

Этот документ описывает REST API эндпоинт для регистрации нового пользователя в системе **Book Store**.

## Общая информация

- **Метод**: `POST`
- **URL**: `/api/v1/auth/register`
- **Content-Type**: `application/json`
- **Описание**: Регистрация нового пользователя с проверкой reCAPTCHA и валидацией данных.

## Запрос (Request Body)

| Поле            | Тип    | Обязательное | Ограничения / Валидация                                                                                  |
|-----------------|--------|--------------|----------------------------------------------------------------------------------------------------------|
| `firstName`     | string | Да           | Мин. 2 символа, макс. 50, только буквы, пробел и дефис                                                    |
| `lastName`      | string | Да           | Мин. 2 символа, макс. 50, только буквы, пробел и дефис                                                    |
| `username`      | string | Да           | Мин. 3 символа, макс. 30, только латинские буквы, цифры, подчёркивание (_) и точка (.)                    |
| `password`      | string | Да           | Мин. 8 символов<br>Обязательно:<br>• минимум 1 строчная буква<br>• минимум 1 заглавная буква<br>• минимум 1 цифра<br>• минимум 1 спецсимвол |
| `recaptchaToken`| string | Да           | Действительный токен от Google reCAPTCHA v2 ("I'm not a robot")                                          |

### Пример запроса

```json
{
  "firstName": "Ivan",
  "lastName": "Ivanov",
  "username": "ivan_ivanov",
  "password": "StrongPass123!",
  "recaptchaToken": "03AGdBq27..."
}
```

## Успешный ответ (200 OK)

| Поле       | Тип     | Обязательное | Описание                              |
|------------|---------|--------------|---------------------------------------|
| `userId`   | integer | Да           | ID созданного пользователя            |
| `username` | string  | Да           | Имя пользователя                      |
| `message`  | string  | Да           | Сообщение об успехе                   |

### Пример успешного ответа

```json
{
  "userId": 12345,
  "username": "ivan_ivanov",
  "message": "User registered successfully"
}
```

## Ответы с ошибками

Общий формат ошибки:

```json
{
  "error": "string",                       // Код ошибки
  "message": "string",                     // Человекочитаемое сообщение для пользователя
  "details": {                             // Опционально: детали по полям
    "fieldName": ["error message 1", "error message 2"]
  }
}
```

### Коды ответов и ошибки

| HTTP код | Описание ошибки                              | Пример message                                              | Код ошибки (`error`)       |
|----------|----------------------------------------------|-------------------------------------------------------------|----------------------------|
| 200      | Успешная регистрация                         | "User registered successfully"                              | -                          |
| 400      | Ошибка валидации или неверный формат данных  | "Password must have at least one non alphanumeric character..." | `VALIDATION_ERROR`         |
| 400      | Пустые обязательные поля                     | "First Name is required"                                    | `MISSING_FIELD`            |
| 400      | Username уже занят                           | "User exists!"                                              | `USERNAME_TAKEN`           |
| 400      | reCAPTCHA не пройдена или токен недействителен| "Please verify you are not a robot"                         | `RECAPTCHA_FAILED`         |
| 429      | Слишком много попыток регистрации            | "Too many registration attempts. Try again later."          | `RATE_LIMIT_EXCEEDED`      |
| 500      | Внутренняя ошибка сервера                    | "An unexpected error occurred. Please try again later."     | `INTERNAL_ERROR`           |

### Пример ошибки валидации пароля (400)

```json
{
  "error": "VALIDATION_ERROR",
  "message": "Password must have at least one non alphanumeric character, one digit ('0'-'9'), one uppercase ('A'-'Z'), one lowercase ('a'-'z'), one special character and Password must be eight characters or longer",
  "details": {
    "password": [
      "Password must have at least one non alphanumeric character",
      "Password must have at least one digit ('0'-'9')",
      "Password must have at least one uppercase ('A'-'Z')",
      "Password must have at least one lowercase ('a'-'z')",
      "Password must be eight characters or longer"
    ]
  }
}
```

### Пример ошибки "Пользователь уже существует" (400)

```json
{
  "error": "USERNAME_TAKEN",
  "message": "User exists!",
  "details": {
    "username": ["User exists!"]
  }
}
```

## OpenAPI (Swagger) спецификация

```yaml
paths:
  /api/v1/auth/register:
    post:
      summary: Регистрация нового пользователя
      tags:
        - Authentication
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - firstName
                - lastName
                - username
                - password
                - recaptchaToken
              properties:
                firstName:
                  type: string
                  minLength: 2
                  maxLength: 50
                lastName:
                  type: string
                  minLength: 2
                  maxLength: 50
                username:
                  type: string
                  pattern: '^[a-zA-Z0-9_.]{3,30}$'
                password:
                  type: string
                  minLength: 8
                recaptchaToken:
                  type: string
      responses:
        '200':
          description: Успешная регистрация
          content:
            application/json:
              schema:
                type: object
                properties:
                  userId:
                    type: integer
                  username:
                    type: string
                  message:
                    type: string
        '400':
          description: Ошибка валидации или бизнес-логики
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '429':
          description: Слишком много попыток
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '500':
          description: Внутренняя ошибка сервера
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'

components:
  schemas:
    ErrorResponse:
      type: object
      properties:
        error:
          type: string
        message:
          type: string
        details:
          type: object
          additionalProperties:
            type: array
            items:
              type: string
```
