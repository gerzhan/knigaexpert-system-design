# UC-3 Получить информацию о заказе

### Покупателем
```plantuml
actor "Покупатель" as customer
participant "shop (web or mobile)" as shop_app
participant "shop BFF" as shop_bff
participant "order service" as order_service

customer -> shop_app: Получить информацию о заказе
shop_app -> shop_bff: Получить информацию о заказе
note over shop_bff: [GET] /api/v1/orders/{id}/status

shop_bff -> order_service: Проксирует запрос
note over order_service: [GET] /api/v1/orders/{id}/status

shop_app <--  order_service: 200 OK
note over shop_app: Экран "Статус заказа"
customer <--  shop_app: Статус заказа
```

### Менеджером
```plantuml
actor Менеджер as manager
participant "backoffice (web)" as backoffice_web_app
participant "backoffice BFF" as backoffice_bff
participant "order_service" as order_service

manager -> backoffice_web_app: Получить информацию о заказе
backoffice_web_app -> backoffice_bff: Получить информацию о заказе
note over backoffice_bff: [GET] /api/v1/orders/{id}/status

backoffice_bff -> order_service: Проксирует запрос
note over order_service: [GET] /api/v1/orders/{id}/status

backoffice_web_app <--  order_service: 200 OK
note over backoffice_web_app: Экран "Статус заказа"
manager <--  backoffice_web_app: Статус заказа
```