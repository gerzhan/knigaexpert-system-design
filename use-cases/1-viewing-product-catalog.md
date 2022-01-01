# UC-1 Просмотреть каталог продуктов

## Экраны
![алтернативный текст](../img/three-teams.png)

## Диаграмма последовательностей
```plantuml
actor "Покупатель" as customer
participant "showcase" as showcase_app
participant "showcase BFF" as showcase_bff
participant "ordering" as ordering

customer -> ordering: Запрашивает каталог продуктов
note over ordering: [GET]/api/v1/products
customer <--  ordering: Каталог продуктов
```

