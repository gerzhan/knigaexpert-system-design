# UC-1 Посмотреть детали продукта
Пользователь может просмотреть карточку продукта. 

## Экраны
![алтернативный текст](../img/three-teams.png)

## Диаграмма последовательностей


```plantuml
actor "Покупатель" as customer
participant "showcase" as showcase_app
participant "showcase BFF" as showcase_bff
participant "basket" as basket

customer -> basket: Запрашивает продукт
note over basket: [GET]/api/v1/products/{id}
alt позитивный сценарий
customer <--  basket: Детали продукта
note over basket
{  
    "title": "Яблоко",
    "price": 7
}
end note
else негативный сценарий
customer <-- basket: Продукт не найден.
note over basket
HTTP Status: 404 Not Found
type: not-found
end note
end
```

