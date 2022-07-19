# UC-3 Получение статуса доставки

### Покупателем
```plantuml
actor "Покупатель" as customer
participant "shop" as shop_app
participant "shop BFF" as shop_bff
participant "delivery" as delivery

customer -> shop_app: Запрашивает статус доставки
shop_app -> shop_bff: Запрашивает статус доставки
note over shop_bff: [GET] /api/v1/orders/{id}/status

shop_bff -> delivery: Проксирует запрос
note over delivery: [GET] /api/v1/orders/{id}/status

shop_app <--  delivery: 200 OK
note over shop_app: Экран "Статус доставки"
customer <--  shop_app: Статус доставки
```

### Менеджером
```plantuml
actor Менеджер as manager
participant "backoffice" as backoffice_app
participant "backoffice BFF" as backoffice_bff
participant "delivery" as delivery

manager -> backoffice_app: Запрашивает статус доставки
backoffice_app -> backoffice_bff: Запрашивает статус доставки
note over backoffice_bff: [GET] /api/v1/orders/{id}/status

backoffice_bff -> delivery: Проксирует запрос
note over delivery: [GET] /api/v1/orders/{id}/status

backoffice_app <--  delivery: 200 OK
note over backoffice_app: Экран "Статус доставки"
manager <--  backoffice_app: Статус доставки
```