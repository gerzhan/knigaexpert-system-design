# UC-4 Добавление адреса доставки

```plantuml
actor "Покупатель" as customer
participant "shop" as shop_app
participant "shop BFF" as shop_bff
participant "basket" as basket

customer -> shop_app: Добавляет адрес доставки
shop_app -> shop_bff: Добавляет адрес доставки
note over shop_bff: [POST] /api/v1/carts/current/address/add

shop_bff -> basket: Проксирует запрос
note over basket: [POST] /api/v1/carts/current/address/add

shop_app <--  basket: 200 OK
note over shop_app: Адрес добавлен к заказу
customer <--  shop_app: Заказ
```