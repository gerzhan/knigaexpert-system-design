# UC-1 Просмотреть каталог продуктов

```plantuml
actor "Покупатель" as customer
participant "shop" as shop_app
participant "shop BFF" as shop_bff
participant "catalog" as catalog

customer -> shop_app: Запрашивает каталог продуктов
shop_app -> shop_bff: Запрашивает каталог продуктов
note over shop_bff: [GET] /api/v1/products

shop_bff -> catalog: Проксирует запрос
note over catalog: [GET] /api/v1/products

customer <--  catalog: Каталог продуктов
```