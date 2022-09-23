# UC-2 Удаление товара из корзины

```plantuml
actor "Покупатель" as customer
participant "shop" as shop_app
participant "shop BFF" as shop_bff
participant "basket" as basket

customer -> shop_app: Удаляет товар из корзины
shop_app -> shop_bff: Удаляет товар из корзины
note over shop_bff: [POST] /api/v1/carts/current/items/{id}/delete

shop_bff -> basket: Проксирует запрос
note over basket: [POST] /api/v1/carts/current/items/{id}/delete

shop_app <--  basket: 200 OK
note over shop_app: Корзина изменена
customer <--  shop_app: Корзина
```