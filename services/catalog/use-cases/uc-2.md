# UC-2 Посмотр карточки продукта

```plantuml
actor "Покупатель" as customer
participant "shop" as shop_app
participant "shop BFF" as shop_bff
participant "catalog" as catalog
alt Успешный случай
customer -> shop_app: Запрашивает карточку продукта
shop_app -> shop_bff: Запрашивает карточку продукта
note over shop_bff: [GET] /api/v1/products/{id}

shop_bff -> catalog: Проксирует запрос
note over catalog: [GET] /api/v1/products/{id}

customer <--  catalog: Карточка продукта
end
alt Продукт не существует
shop_app <--  catalog: 404 Not Found
note over shop_app: Ошибка "Продукт не существует"
customer <--  shop_app: Экран с ошибкой
end
```