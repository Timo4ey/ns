### Регистрация и авторизация клиента и адаптера
```mermaid
sequenceDiagram
    actor Admin
    participant Client
    participant Adapter
    participant NS
    participant DB

    Note over Admin, NS: Регистрация клиента в NS
    Admin ->> NS: POST /client(token)
    NS ->> DB: User.create_one(token)

    Note over Admin, NS: Регистрация клиента в NS
    Admin ->> NS: POST /adapter(token)
    NS ->> DB: Adapter.create_one(token)
```
___
### Каждый клиент может получить список адаптеров, доступных для уведомлений
```mermaid
sequenceDiagram
    participant Client
    participant NS
    participant DB

    Note over Client, DB: Получение списка адаптеров
    Client ->> NS: GET /adapter
    NS -->> DB: Adapter.get_list

    Note over Client, DB: Создание заявки на уведомление
    Client ->> NS: POST /job
    NS -->> DB: Job.create_one
    DB -->> NS: job_id
    NS -->> Client: job_id

    alt Клиента хочет подписаться на результат
        Note over Client, DB: Подписка
        Client ->> NS: POST /notification
        NS -->> DB: Notification.create_one()
    end

    alt Клиента хочет получить статус задачи
        Note over Client, DB: Чтение задачи
        Client ->> NS: GET /job/<job_id>
        NS -->> DB: Notification.get_one(job_id)
    end

    alt Клиента хочет отменить задачу
        Note over Client, DB: Удаление задачи
        Client ->> NS: DELETE /job/<job_id>
        NS -->> DB: Notification.delete_one(job_id)
    end
```
___

### Взаимеодействие с адаптерами
```mermaid

sequenceDiagram
    participant Client
    participant NS
    participant Adapter
    actor User
    actor System

    alt если client создал задачу
        Note over Client, Adapter: уведомляем адаптер
        Client ->> NS: POST /job
        NS ->> Adapter: POST /job
    end

    loop Every minute
        Note over Client, Adapter: запрашиваем раз в минуту
        Adapter ->> NS: GET /job

        alt уведомляем по sms
            Adapter -->> User: send_sms
        else уведомляем по http
            Adapter -->> System: POST /some_resource
        end
    end

    Adapter -->> User: 

    Adapter -->> NS: PUT /job
```
