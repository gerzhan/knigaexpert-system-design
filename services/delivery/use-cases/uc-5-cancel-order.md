# UC-1 Отменить заказ

```plantuml
actor Менеджер as manager
participant "backoffice" as backoffice_web_app
participant "backoffice BFF" as backoffice_bff
participant "order service" as order_service

manager -> backoffice_web_app: Отменяет заказ
backoffice_web_app -> backoffice_bff: Отменяет заказ
note over backoffice_bff: [POST] /api/v1/orders/{id}/cancel

backoffice_bff -> order_service: Проксирует запрос
note over order_service: [POST] /api/v1/orders/{id}/cancel

backoffice_web_app <--  order_service: 200 OK
note over backoffice_web_app: Экран "Заказ отменен"
manager <--  backoffice_web_app: Заказ отменен
```
```