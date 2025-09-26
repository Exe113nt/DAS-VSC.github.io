# DAS (Decomposed API Specification) Framework

DAS Framework — это подход к модульной разработке OpenAPI/AsyncAPI спецификаций, позволяющий разбивать монолитные документы на логические модули и слои.

Основная идея заключается в том, чтобы один раз описывать модели сущностей и переиспользуемые атрибуты (например, идентификаторы или даты), а затем многократно ссылаться на них в разных частях спецификации. 
Такой подход значительно повышает читаемость и снижает количество дублируемых определений, минимизируя рассинхронизацию.

Дополнительное преимущество подхода заключается в том, что при добавлении нового интерфейса (например, WebSocket, Kafka и др.) нет необходимости заново описывать модели и сообщения. Достаточно подключить уже существующие определения, что многократно сокращает время разработки спецификации нового интерфейса.

## Преимущества

📂 Модульность — каждая сущность и её части описываются один раз.

🔄 Переиспользование — атрибуты и модели повторно применяются в разных модулях.

📖 Читаемость — структура спецификации становится более прозрачной.

🛡 Качество — минимизация дублирования снижает риск ошибок и рассинхронизации.

## 📂 Шаблон структуры файлов

```bash
SomeService/
  └──controllers/
    └── EntityController/ <-- контроллер
          ├── models/
          │     └── Entity.yaml <-- модель
          ├── schemas.yaml <-- DTO
          ├── parameters.yaml <-- query параметры запросов
          └── paths.yaml <-- эндпоинты
    └── utils/
          ├── responses.yaml <-- общие оболочки ответов, структура сообщений об ошибках
          └── attributes.yaml <-- переиспользуемые в сервисе/проекте атрибуты
    └── openapi.yaml <-- корневой файл openapi спецификации
    └── asyncapi.yaml <-- корневой файл asyncapi спецификации
  └──out/ 
    └──SomeService.yaml <- итоговая openapi спецификация
```

## 🧩 Связи модулей спецификации

Данный пример демонстрирует, что благодаря разбиению спецификации на отдельные файлы и слои, каждый компонент остаётся изолированным и легко читаемым. Это упрощает навигацию и позволяет разрабатывать новые эндпоинты, контракты и сущности с минимальными затратами времени.

<details><summary>Корневой файл спецификации</summary>
  
```yaml
#openapi.yaml

openapi: 3.1.0
info:
  version: 1.0.0
  title: SomeService API

tags:
  - name: EntityController
    description: Entity Controller

paths:
  /api/v1/entities/:
    $ref: './controllers/EntityController/paths.yaml#/Entites'
```
</details>

<details><summary>Эндпоинты контроллера Entity</summary>

```yaml
#controllers/EntityController/paths.yaml

Entities:
  get:
    tags:
      - EntityController
    summary: Get list of Entity objects
    operationId: GetEntityObjects
    description: |
      Description of how the endpoint works
    responses:
      200 :
        description: List of Entity objects
        content:
          application/json:
            schema:
              $ref: 'schemas.yaml#/GetEntityListResponseDto'
      400:
        $ref: '../../utils/responses.yaml#/ValidationError'
      401:
        $ref: '../../utils/responses.yaml#/UnauthorizedError'
      403:
        $ref: '../../utils/responses.yaml#/ForbiddenError'
      500:
        $ref: '../../utils/responses.yaml#/InternalServerError'
```
</details>

<details><summary>DTO контроллера</summary>

```yaml
#controllers/EntityController/schemas.yaml

GetEntityListResponseDto:
  type: array
  items: 
    $ref: './models/Entity.yaml#/EntityModel'
```
</details>

<details><summary>Модель Entity</summary>

```yaml
#controllers/EntityController/models/Entity.yaml

EntityModel:
  type: object
  required:
    - id
  properties:
    id:
      $ref: '#/EntityId'

EntityId:
  type: string
  format: uuid
  description: An unique Entity identifier
```
</details>

<details><summary>Итоговая спецификация</summary>

```yaml
openapi: 3.1.0
info:
  version: 1.0.0
  title: SomeService API

tags:
  - name: EntityController
    description: Entity Controller

paths:
  /api/v1/entities/:
    get:
      tags:
        - EntityController
      summary: Get list of Entity objects
      operationId: GetEntityObjects
      description: |
        Description of how the endpoint works
      responses:
        200 :
          description: List of Entity objects
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/GetEntityListResponseDto'
        400:
          $ref: '#/components/schemas/ValidationError'
        401:
          $ref: '#/components/schemas/UnauthorizedError'
        403:
          $ref: '#/components/schemas/ForbiddenError'
        500:
          $ref: '#/components/schemas/InternalServerError'

components:
  schemas:
    EntityModel:
      type: object
      required:
        - id
      properties:
        id:
          $ref: '#/components/schemas/EntityId'
    EntityId:
      type: string
      format: uuid
      description: An unique Entity identifier
    GetEntityListResponseDto:
      type: array
      items: 
        $ref: '#/components/schemas/EntityModel'
    ...
```
</details>

## 💻 Текущая реализация

Сейчас DAS Framework развернут как закрытая веб-версия VS Code с авторизацией и преднастроенной средой для работы с API-спецификациями. Это не просто редактор, а полноценный инструмент командной разработки:

🔍 Предпросмотр спецификации в реальном времени - фоновый демон отслеживает изменения в файловой структуре и автоматически пересобирает итоговую спецификацию. Это значит, что команда всегда видит актуальный OpenAPI/AsyncAPI документ для предпросмотра и публикации без ручной сборки.

🧹 Линтер и контроль качества - встроенные проверки гарантируют, что спецификация строго соответствует правилам OpenAPI. Ошибки выявляются сразу, ещё на этапе разработки, а не в процессе интеграции.

📝 Типизированные комментарии - поддержка аннотаций и структурированных комментариев позволяет создавать понятную документацию, которая одновременно служит и для разработчиков, и для аналитиков.

🤝 Совместная работа аналитиков и разработчиков - вся команда работает в единой среде и в единой структуре файлов. Спецификация становится единым источником правды, а не разрозненными документами и таблицами в Confluence.

🔗 Интеграция с Confluence - готовая спецификация подключается к Confluence через плагин и отображается в виде полноценной документации. Больше никаких мучительных таблиц с методами и ручного форматирования — все данные подгружаются из «живой» спецификации.

Таким образом, DAS превращает процесс описания API из рутинной и дублирующейся работы в управляемый, прозрачный и командный процесс, где вся спецификация поддерживается в актуальном виде и всегда готова к публикации.
