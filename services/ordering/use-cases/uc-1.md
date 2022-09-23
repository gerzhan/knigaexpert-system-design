# UC-1 Добавление товара в корзину

```plantuml
actor "Покупатель" as customer
participant "shop" as shop_app
participant "shop BFF" as shop_bff
participant "basket" as basket

customer -> shop_app: Добавляет товар в корзину
shop_app -> shop_bff: Добавляет товар в корзину
note over shop_bff: [POST] /api/v1/carts/current/items/add

shop_bff -> basket: Проксирует запрос
note over basket: [POST] /api/v1/carts/current/items/add

shop_app <--  basket: 200 OK
note over shop_app: Корзина изменена
customer <--  shop_app: Корзина
```