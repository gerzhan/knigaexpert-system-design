# UC-2 Изменение статуса заказа на 'Доставлен'

```plantuml
actor Курьер as courier
participant "courier app" as courier_app
participant "courier BFF" as courier_bff
participant "delivery" as delivery

courier -> courier_app: Изменяет статус доставки на "Доставлен"
courier_app -> courier_bff: Изменяет статус доставки на "Доставлен"
note over courier_bff: [POST] /api/v1/orders/{id}/status

courier_bff -> delivery: Проксирует запрос
note over delivery: [POST] /api/v1/orders/{id}/status

courier_app <--  delivery: 200 OK
note over courier_app: Сообщение "Статус доставки изменен"
courier <--  courier_app: Статус доставки изменен
```