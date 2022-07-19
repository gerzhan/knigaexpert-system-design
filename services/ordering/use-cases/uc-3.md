# UC-3 Просмотр списка товаров в корзине

```plantuml
actor "Покупатель" as customer
participant "shop" as shop_app
participant "shop BFF" as shop_bff
participant "ordering" as ordering

customer -> shop_app: Просматривает корзину
shop_app -> shop_bff: Просматривает корзину
note over shop_bff: [GET] /api/v1/carts/current

shop_bff -> ordering: Проксирует запрос
note over ordering: [GET] /api/v1/carts/current

shop_app <--  ordering: 200 OK
note over shop_app: Товары в корзине
customer <--  shop_app: Корзина
```