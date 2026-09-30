# RestAPI_JPA_H2

Простой REST API на базе **Spring Boot**, **Spring Data JPA** и встроенной **H2 Database**.

Проект демонстрирует базовый подход к созданию REST API для работы с реляционными данными через JPA и Hibernate, а также разделение приложения на контроллеры, сервисы и репозитории.

## Используемые технологии

* Java 17
* Spring Boot 3.4.1
* Spring Web
* Spring Data JPA
* Hibernate
* H2 Database
* SpringDoc OpenAPI
* Maven

## Структура проекта

```text
src/
├── main/
│   ├── java/
│   │   └── com/crosska/jpa/JPAWorker/
│   │       ├── controller/
│   │       │   ├── BankController.java
│   │       │   └── PersonController.java
│   │       │
│   │       ├── entity/
│   │       │   ├── Bank.java
│   │       │   └── Person.java
│   │       │
│   │       ├── repository/
│   │       │   ├── BankRepository.java
│   │       │   └── PersonRepository.java
│   │       │
│   │       ├── service/
│   │       │   ├── BankService.java
│   │       │   └── PersonService.java
│   │       │
│   │       └── JpaWorkerApplication.java
│   │
│   └── resources/
│       └── application.properties
│
└── test/
    └── java/
        └── com/crosska/jpa/JPAWorker/
            └── JpaWorkerApplicationTests.java
```

В проекте используется классическая многослойная структура:

```text
HTTP-запрос
     │
     ▼
Controller
     │
     ▼
Service
     │
     ▼
Repository
     │
     ▼
JPA / Hibernate
     │
     ▼
H2 Database
```

## Модель базы данных

В приложении используются две основные сущности: `Bank` и `Person`.

Один банк может содержать несколько пользователей, при этом каждый `Person` связан с одним `Bank`.

```text
Bank
 ├── id
 ├── name
 ├── address
 ├── city
 ├── country
 │
 └── persons
       │
       ├── Person
       ├── Person
       └── ...
```

### Bank

Сущность `Bank` соответствует таблице `banks`.

Основные поля:

* `id` - автоматически генерируемый первичный ключ
* `name` - уникальное название банка
* `address` - уникальный адрес банка
* `city` - город
* `country` - страна
* `persons` - список связанных пользователей

Связь с `Person` реализована через `@OneToMany`:

```java
@OneToMany(
    mappedBy = "bank",
    cascade = CascadeType.ALL,
    orphanRemoval = true
)
private List<Person> persons;
```

### Person

Сущность `Person` соответствует таблице `persons`.

Основные поля:

* `id` - автоматически генерируемый первичный ключ
* `name` - имя
* `surname` - фамилия
* `email` - адрес электронной почты
* `age` - возраст
* `bank` - связанный банк

Связь с `Bank` реализована через `@ManyToOne`:

```java
@ManyToOne
@JoinColumn(name = "banks_id", nullable = false)
private Bank bank;
```

Таким образом, используется классическая связь:

```text
Bank 1 ─────────── N Person
```

## REST API

Все REST endpoints используют базовый путь `/h2`.

### Person

#### Создание пользователя

```http
POST /h2/person
```

Пример запроса:

```json
{
    "name": "John",
    "surname": "Smith",
    "email": "john.smith@example.com",
    "age": 30
}
```

При создании пользователя выполняется проверка возраста. Допустимый диапазон: от `1` до `119`.

Пример ответа:

```json
{
    "id": 1,
    "name": "John",
    "surname": "Smith",
    "email": "john.smith@example.com",
    "age": 30
}
```

#### Получение всех пользователей

```http
GET /h2/person
```

Возвращает список всех пользователей, находящихся в базе данных.

#### Получение пользователя по ID

```http
GET /h2/person/{id}
```

Пример:

```http
GET /h2/person/1
```

Если пользователь найден, API возвращает объект `Person`.

Если пользователь отсутствует:

```http
404 Not Found
```

#### Обновление пользователя

```http
PUT /h2/person/{id}
```

Пример:

```http
PUT /h2/person/1
```

Тело запроса:

```json
{
    "name": "John",
    "surname": "Johnson",
    "email": "john.johnson@example.com",
    "age": 31
}
```

#### Удаление пользователя

```http
DELETE /h2/person/{id}
```

Пример:

```http
DELETE /h2/person/1
```

Удаляет пользователя с указанным ID.

### Bank

#### Создание банка

```http
POST /h2/bank
```

Пример запроса:

```json
{
    "name": "Example Bank",
    "address": "Main Street 1",
    "city": "Berlin",
    "country": "Germany"
}
```

При успешном создании возвращается HTTP-код `201 Created`.

#### Получение всех банков

```http
GET /h2/bank
```

Возвращает список всех банков из базы данных.

#### Получение банка по ID

```http
GET /h2/bank/{id}
```

Пример:

```http
GET /h2/bank/1
```

Возвращает банк с указанным ID.

Если банк не найден:

```http
404 Not Found
```

#### Обновление банка

```http
PUT /h2/bank/{id}
```

Обновляет существующий объект `Bank`.

#### Удаление банка

```http
DELETE /h2/bank/{id}
```

Удаляет банк с указанным ID.

## H2 Database

В проекте используется H2 Database в режиме хранения данных в памяти.

Настройки базы данных:

```properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=password
```

Hibernate автоматически создаёт структуру базы данных при запуске приложения:

```properties
spring.jpa.hibernate.ddl-auto=create-drop
```

Это означает, что база данных существует только во время работы приложения. После остановки приложения данные удаляются.

Для просмотра SQL-запросов Hibernate включён параметр:

```properties
spring.jpa.show-sql=true
```

## OpenAPI / Swagger

Для автоматической документации REST API используется **SpringDoc OpenAPI**.

После запуска приложения Swagger UI доступен по адресу:

```text
http://localhost:8080/swagger-docs.html
```

OpenAPI specification:

```text
http://localhost:8080/api-docs
```

Swagger UI позволяет просматривать доступные endpoints и выполнять HTTP-запросы непосредственно из браузера.

## Запуск проекта

### Требования

Перед запуском необходимо установить:

* Java 17 или новее
* Git

Отдельно устанавливать Maven не требуется, поскольку проект содержит Maven Wrapper.

### Клонирование репозитория

```bash
git clone https://github.com/foxhound-official/RestAPI_JPA_H2.git
cd RestAPI_JPA_H2
```

### Запуск через Maven Wrapper

Windows:

```cmd
mvnw.cmd spring-boot:run
```

Linux / macOS:

```bash
./mvnw spring-boot:run
```

После запуска приложение будет доступно по адресу:

```text
http://localhost:8080
```

### Сборка проекта

Linux / macOS:

```bash
./mvnw clean package
```

Windows:

```cmd
mvnw.cmd clean package
```

После успешной сборки JAR-файл будет находиться в директории:

```text
target/
```

Запуск собранного приложения:

```bash
java -jar target/JPAWorker-0.0.1-SNAPSHOT.jar
```

## Тестирование

В проекте присутствует базовый тест Spring Boot:

```text
src/test/java/com/crosska/jpa/JPAWorker/JpaWorkerApplicationTests.java
```

Запуск тестов:

Linux / macOS:

```bash
./mvnw test
```

Windows:

```cmd
mvnw.cmd test
```

## Пример работы с API

Типичный сценарий работы с приложением может выглядеть следующим образом.

Сначала создаём банк:

```http
POST /h2/bank
Content-Type: application/json
```

```json
{
    "name": "Example Bank",
    "address": "Main Street 1",
    "city": "Berlin",
    "country": "Germany"
}
```

После этого можно создавать и получать данные через соответствующие REST endpoints.

Получение списка банков:

```http
GET /h2/bank
```

Получение списка пользователей:

```http
GET /h2/person
```

Получение конкретного объекта:

```http
GET /h2/person/1
```

Обновление:

```http
PUT /h2/person/1
```

Удаление:

```http
DELETE /h2/person/1
```

## Назначение проекта

Проект представляет собой небольшой пример создания REST API с использованием Spring Boot и JPA.

Основные задачи проекта:

* создание REST-контроллеров;
* работа с HTTP GET, POST, PUT и DELETE;
* разделение приложения на Controller, Service и Repository;
* использование Spring Data JPA;
* создание JPA-сущностей;
* настройка связей `@OneToMany` и `@ManyToOne`;
* работа с H2 Database;
* автоматическое создание структуры базы данных через Hibernate;
* документирование API через OpenAPI и Swagger.

Проект может использоваться как учебный пример для изучения взаимодействия Spring Boot, REST API, JPA, Hibernate и реляционной базы данных.

## Лицензия

В репозитории на данный момент не указана отдельная лицензия.
