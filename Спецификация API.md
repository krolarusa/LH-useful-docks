# Шаблон спецификации API (расширенная версия)

## 1. Описание ресурса

### 1.1. Общая информация
- **Ресурс**: `[Название ресурса]`
- **Описание**: `[Краткое описание назначения ресурса]`
- **Версия API**: `[v1, v2, ...]`
- **Область видимости**: `[public/internal/admin/system]`
- **Жизненный цикл**: `[stable/deprecated/experimental]`
- **Дата ввода**: `[YYYY-MM-DD]`
- **Дата депрекейшена**: `[YYYY-MM-DD]` (если применимо)

### 1.2. Бизнес-контекст
- **Бизнес-процесс**: `[Описание бизнес-процесса]`
- **Сценарии использования**:
  1. `[Сценарий 1]`
  2. `[Сценарий 2]`
  3. `[Сценарий 3]`

### 1.3. Связанные ресурсы
```
Ссылки на связанные ресурсы:
- GET /api/v1/related-resource - [Описание связи]
- POST /api/v1/other-resource - [Описание связи]
```

## 2. Описание метода и эндпоинта

### 2.1. Основная информация
- **Метод**: `[GET|POST|PUT|PATCH|DELETE]`
- **Эндпоинт**: `/api/v1/[resource]/[sub-resource]`
- **Полный URL**: `{protocol}://{host}:{port}/api/v1/[resource]/[sub-resource]`

### 2.2. Авторизация и доступ
| Уровень доступа | Требуемая роль | Область видимости | Описание |
|----------------|----------------|-------------------|----------|
| `[public/private/internal]` | `[ROLE_NAME]` | `[read/write/admin]` | `[Условия доступа]` |

**Требования к аутентификации**:
- Тип: `[Bearer Token/Basic Auth/OAuth2.0]`
- Токен в заголовке: `Authorization: Bearer {token}`
- Дополнительные заголовки: `[X-API-Key, X-Client-ID, etc]`

### 2.3. Параметры запроса

#### Path Parameters
```json
{
  "id": {
    "type": "integer|string|uuid",
    "required": true,
    "description": "Идентификатор ресурса",
    "constraints": {
      "min": 1,
      "max": 999999,
      "pattern": "[0-9]+"
    },
    "example": 12345
  }
}
```

#### Query Parameters
```yaml
page:
  type: integer
  required: false
  default: 1
  description: "Номер страницы для пагинации"
  constraints:
    min: 1
    max: 1000

size:
  type: integer
  required: false
  default: 20
  description: "Количество элементов на странице"
  constraints:
    min: 1
    max: 100

sort:
  type: string
  required: false
  default: "createdAt,desc"
  description: "Поле и направление сортировки"
  enum: ["createdAt,asc", "createdAt,desc", "name,asc", "name,desc"]

filter:
  type: object
  required: false
  description: "Фильтры в формате RSQL"
  example: "status==ACTIVE;createdAt>=2024-01-01"
```

#### Request Body (для POST/PUT/PATCH)
```json
{
  "type": "object",
  "required": ["field1", "field2"],
  "properties": {
    "field1": {
      "type": "string",
      "description": "Описание поля",
      "minLength": 1,
      "maxLength": 255,
      "pattern": "^[A-Za-z0-9]+$",
      "example": "exampleValue",
      "db_mapping": {
        "table": "table_name",
        "column": "column_name",
        "type": "VARCHAR(255)",
        "nullable": false,
        "constraints": ["UNIQUE"]
      }
    },
    "field2": {
      "type": "integer",
      "description": "Числовое поле",
      "minimum": 0,
      "maximum": 100,
      "default": 50,
      "example": 75,
      "db_mapping": {
        "table": "table_name",
        "column": "column_name",
        "type": "INTEGER",
        "nullable": true,
        "default": "50"
      }
    },
    "nestedObject": {
      "type": "object",
      "description": "Вложенный объект",
      "properties": {
        "subField": {
          "type": "string",
          "example": "nestedValue"
        }
      }
    }
  }
}
```

### 2.4. Заголовки запроса
```
Content-Type: application/json
Accept: application/json
X-Request-ID: {uuid}
X-Correlation-ID: {uuid}
X-Client-Version: 1.0.0
```

## 3. Валидация на бэкенде

### 3.1. Уровни валидации
1. **Синтаксическая валидация** (формат JSON, типы данных)
2. **Семантическая валидация** (бизнес-правила, ограничения)
3. **Бизнес-валидация** (состояние системы, права доступа)

### 3.2. Правила валидации
```yaml
validations:
  - field: "email"
    rules:
      - required: true
      - type: string
      - pattern: "^[\\w-\\.]+@([\\w-]+\\.)+[\\w-]{2,4}$"
      - unique: true
        check_in_table: "users"
        column: "email"
    error_messages:
      required: "Email обязателен для заполнения"
      pattern: "Некорректный формат email"
      unique: "Пользователь с таким email уже существует"
      
  - field: "age"
    rules:
      - type: integer
      - min: 18
      - max: 120
    error_messages:
      min: "Возраст должен быть не менее 18 лет"
      max: "Возраст не может превышать 120 лет"
```

### 3.3. Исключения и коды ошибок
| HTTP Status | Код ошибки | Сообщение | Условие возникновения |
|-------------|------------|-----------|----------------------|
| 400 | `VALIDATION_ERROR` | `[Описание]` | Ошибка валидации входных данных |
| 401 | `UNAUTHORIZED` | `[Описание]` | Отсутствует или невалидный токен |
| 403 | `FORBIDDEN` | `[Описание]` | Недостаточно прав |
| 404 | `NOT_FOUND` | `[Описание]` | Ресурс не найден |
| 409 | `CONFLICT` | `[Описание]` | Конфликт данных (уникальность, состояние) |
| 422 | `BUSINESS_RULE_VIOLATION` | `[Описание]` | Нарушение бизнес-правил |
| 429 | `RATE_LIMIT_EXCEEDED` | `[Описание]` | Превышен лимит запросов |
| 500 | `INTERNAL_ERROR` | `[Описание]` | Внутренняя ошибка сервера |

### 3.4. Формат ответа с ошибкой
```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "status": 400,
  "error": "Bad Request",
  "code": "VALIDATION_ERROR",
  "message": "Ошибка валидации входных данных",
  "details": [
    {
      "field": "email",
      "message": "Некорректный формат email",
      "rejectedValue": "invalid-email"
    }
  ],
  "path": "/api/v1/users",
  "requestId": "req-123456789",
  "documentationUrl": "https://docs.example.com/errors/VALIDATION_ERROR"
}
```

## 4. Ответ

### 4.1. Успешный ответ
**HTTP Status**: `[200|201|204]`

**Заголовки ответа**:
```
Content-Type: application/json
X-Request-ID: {uuid}
X-Rate-Limit-Limit: 100
X-Rate-Limit-Remaining: 99
X-Rate-Limit-Reset: 1673780400
```

**Структура тела ответа**:
```json
{
  "success": true,
  "data": {
    // Основные данные
  },
  "metadata": {
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "req-123456789",
    "version": "1.0.0"
  },
  "pagination": {
    "page": 1,
    "size": 20,
    "totalElements": 150,
    "totalPages": 8,
    "first": true,
    "last": false
  },
  "links": {
    "self": "/api/v1/resource?page=1&size=20",
    "next": "/api/v1/resource?page=2&size=20",
    "prev": null,
    "first": "/api/v1/resource?page=1&size=20",
    "last": "/api/v1/resource?page=8&size=20"
  }
}
```

### 4.2. Структура данных
```json
{
  "id": {
    "type": "integer",
    "description": "Уникальный идентификатор",
    "example": 12345
  },
  "createdAt": {
    "type": "string",
    "format": "date-time",
    "description": "Дата и время создания",
    "example": "2024-01-15T10:30:00Z"
  },
  "updatedAt": {
    "type": "string",
    "format": "date-time",
    "description": "Дата и время последнего обновления",
    "example": "2024-01-15T11:45:00Z"
  },
  "status": {
    "type": "string",
    "enum": ["ACTIVE", "INACTIVE", "PENDING", "DELETED"],
    "description": "Статус ресурса",
    "example": "ACTIVE"
  }
}
```

### 4.3. Кэширование
```yaml
caching:
  enabled: true
  strategy: "public"
  max_age: 3600
  stale_while_revalidate: 86400
  vary: ["Accept-Encoding", "Authorization"]
  cache_control: "public, max-age=3600, stale-while-revalidate=86400"
```

## 5. Примеры запрос-ответ

### 5.1. Пример 1: Получение списка ресурсов
**Запрос**:
```http
GET /api/v1/users?page=1&size=10&sort=name,asc&filter=status==ACTIVE
Accept: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
X-Request-ID: 550e8400-e29b-41d4-a716-446655440000
```

**Ответ (200 OK)**:
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "email": "user1@example.com",
      "name": "Иван Иванов",
      "status": "ACTIVE",
      "createdAt": "2024-01-10T14:30:00Z",
      "updatedAt": "2024-01-12T09:15:00Z"
    },
    {
      "id": 2,
      "email": "user2@example.com",
      "name": "Петр Петров",
      "status": "ACTIVE",
      "createdAt": "2024-01-11T10:45:00Z",
      "updatedAt": "2024-01-14T16:20:00Z"
    }
  ],
  "metadata": {
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "550e8400-e29b-41d4-a716-446655440000",
    "version": "1.0.0"
  },
  "pagination": {
    "page": 1,
    "size": 10,
    "totalElements": 45,
    "totalPages": 5,
    "first": true,
    "last": false
  },
  "links": {
    "self": "/api/v1/users?page=1&size=10",
    "next": "/api/v1/users?page=2&size=10",
    "prev": null,
    "first": "/api/v1/users?page=1&size=10",
    "last": "/api/v1/users?page=5&size=10"
  }
}
```

### 5.2. Пример 2: Создание ресурса
**Запрос**:
```http
POST /api/v1/users
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

{
  "email": "newuser@example.com",
  "name": "Новый пользователь",
  "password": "SecurePass123!",
  "age": 25
}
```

**Ответ (201 Created)**:
```json
{
  "success": true,
  "data": {
    "id": 46,
    "email": "newuser@example.com",
    "name": "Новый пользователь",
    "status": "PENDING",
    "createdAt": "2024-01-15T10:30:00Z",
    "updatedAt": "2024-01-15T10:30:00Z"
  },
  "metadata": {
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "550e8400-e29b-41d4-a716-446655440001",
    "version": "1.0.0"
  },
  "links": {
    "self": "/api/v1/users/46",
    "activate": "/api/v1/users/46/activate"
  }
}
```

**Ответ с ошибкой (400 Bad Request)**:
```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "status": 400,
  "error": "Bad Request",
  "code": "VALIDATION_ERROR",
  "message": "Ошибка валидации входных данных",
  "details": [
    {
      "field": "email",
      "message": "Пользователь с таким email уже существует",
      "rejectedValue": "existing@example.com"
    },
    {
      "field": "password",
      "message": "Пароль должен содержать не менее 8 символов",
      "rejectedValue": "123"
    }
  ],
  "path": "/api/v1/users",
  "requestId": "550e8400-e29b-41d4-a716-446655440001"
}
```

### 5.3. Пример 3: Частичное обновление (PATCH)
**Запрос**:
```http
PATCH /api/v1/users/46
Content-Type: application/json-patch+json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

[
  { "op": "replace", "path": "/name", "value": "Обновленное имя" },
  { "op": "replace", "path": "/status", "value": "ACTIVE" }
]
```

**Ответ (200 OK)**:
```json
{
  "success": true,
  "data": {
    "id": 46,
    "email": "newuser@example.com",
    "name": "Обновленное имя",
    "status": "ACTIVE",
    "createdAt": "2024-01-15T10:30:00Z",
    "updatedAt": "2024-01-15T11:45:00Z"
  }
}
```

## 6. Дополнительная информация

### 6.1. Лимиты и квоты
```yaml
rate_limits:
  user:
    requests_per_minute: 60
    burst_limit: 100
  
  api_key:
    requests_per_minute: 1000
    burst_limit: 1500

payload_limits:
  max_request_size: "10MB"
  max_response_size: "50MB"
  max_items_in_array: 1000
```

### 6.2. Версионирование
- **Стратегия**: URI versioning (`/api/v1/resource`)
- **Поддержка версий**: одновременно поддерживаются последние 2 мажорные версии
- **Депрекейшн**: за 6 месяцев до удаления endpoint помечается как deprecated

### 6.3. Мониторинг и метрики
```yaml
monitoring:
  metrics:
    - api.request.count
    - api.request.duration
    - api.error.rate
    - api.response.size
    
  alerts:
    - error_rate_above_5_percent
    - p99_latency_above_2s
    - availability_below_99_9
```

### 6.4. Спецификация OpenAPI (Swagger)
```yaml
openapi: 3.0.0
info:
  title: "User Management API"
  version: "1.0.0"
  description: "API для управления пользователями"

paths:
  /api/v1/users:
    get:
      summary: "Получить список пользователей"
      operationId: "getUsers"
      # ... остальная спецификация
```
