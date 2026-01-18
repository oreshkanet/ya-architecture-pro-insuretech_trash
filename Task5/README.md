# Проектирование GraphQL API

## Анализ RestAPI

Ресурсы:

- `Client` - основная сущность клиента;
- `Document` - список документов клиента;
- `Relatives` - информация о родственниках клиента.

Операции:

- `GET /clients/{id}` - получение информации о клиенте;
- `GET /clients/{id}/documents` - получение списка документов клиента;
- `GET /clients/{id}/relatives` - получение информации о родственниках клиента.

Все операции являются Read-only, без создания/изменения/удаления. Получение данных по каждому из ресурсов требует отдельного HTTP-запроса.

## GraphQL

Цель GraphQL — объединить все данные в один гибкий запрос, избегая дублирования и множественных запросов для получения полных данных.

Ресурсы:

```graphql
type Client {
    id: ID!
    name: String!
    age: Int!

    documents: [Document!]!
    relatives: [Relative!]!
}

type Document {
    id: ID!
    type: String!
    number: String!
    issueDate: String!
    expiryDate: String!
}

type Relative {
    id: ID!
    relationType: String!
    name: String!
    age: Int!
}
```

Операции:

```graphql
type Query {
    client(id: ID!): Client
}
```

## Запросы

Запрос базовой информации о пользователе:

```graphql
query {
    client(id: "123") {
        id
        name
        age
    }
}
```

Запрос информации о пользователе и документах:
```graphql
query {
    client(id: "123") {
        name
        document {
            id
            type
            number
            issueDate
            expiryDate
        }
    }
}
```

Запрос полной информации о пользователе (вместо 3х запросов RestAPI):

```graphql
query {
    client(id: "123") {
        id
        name
        age
        document {
            id
            type
            number
            issueDate
            expiryDate
        }
        type Relative {
            id
            relationType
            name
            age
        }
    }
}
```

## Преимущества GraphQL

GraphQL-схема полностью покрывает функциональность текущего REST API, при этом:

- Устраняет необходимость в нескольких запросах.
- Позволяет клиентам запрашивать только нужные поля.
- Готова к расширению (например, поиск по ФИО, фильтрация документов и т.п.).

Если в будущем REST API будет расширен (например, POST/PUT), схему можно дополнить мутациями (Mutation), но на текущий момент они не требуются.

