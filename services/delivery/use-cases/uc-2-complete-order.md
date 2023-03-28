# UC-2 Завершить выполнение заказа

```plantuml
actor Курьер as courier
participant "courier app" as courier_app
participant "courier BFF" as courier_bff
participant "order service" as order_service

courier -> courier_app: Завершить выполнение заказа
courier_app -> courier_bff: Завершить выполнение заказа
note over courier_bff: [POST] /api/v1/orders/{id}/complete

courier_bff -> order_service: Проксирует запрос
note over order_service: [POST] /api/v1/orders/{id}/complete

courier_app <--  order_service: 200 OK
note over courier_app: Сообщение "Статус заказа изменен"
courier <--  courier_app: Статус заказа изменен
```