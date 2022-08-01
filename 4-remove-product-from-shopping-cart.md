# UC-1 Посмотреть детали продукта
Пользователь может просмотреть карточку продукта. 

## Экраны
![алтернативный текст](../img/three-teams.png)

## Диаграмма последовательностей


```plantuml
actor "Покупатель" as customer
participant "showcase" as showcase_app
participant "showcase BFF" as showcase_bff
participant "ordering" as ordering

customer -> ordering: Запрашивает продукт
note over ordering: [GET]/api/v1/products/{id}
alt позитивный сценарий
customer <--  ordering: Детали продукта
note over ordering
{  
    "title": "Яблоко",
    "price": 7
}
end note
else негативный сценарий
customer <-- ordering: Продукт не найден.
note over ordering
HTTP Status: 404 Not Found
type: not-found
end note
end
```

