# UC-1 Добавление товара в корзину

```plantuml
actor "Покупатель" as customer
participant "shop" as shop_app
participant "shop BFF" as shop_bff
participant "ordering" as ordering

customer -> shop_app: Добавляет товар в корзину
shop_app -> shop_bff: Добавляет товар в корзину
note over shop_bff: [POST] /api/v1/carts/current/items/add

shop_bff -> ordering: Проксирует запрос
note over ordering: [POST] /api/v1/carts/current/items/add

shop_app <--  ordering: 200 OK
note over shop_app: Корзина изменена
customer <--  shop_app: Корзина
```