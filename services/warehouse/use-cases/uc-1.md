# UC-1 Принять поставку
Диаграмма последовательности показывает жизненный цикл объекта на единой временной оси, взаимодействие с другими обьектами и актерами информационной системы в рамках одного варианта использования.

### Продукт добавляется впервые
```plantuml
actor Менеджер as manager
participant "warehouse" as warehouse
queue "products" as products_queue
participant "catalog" as catalog

alt Успешный случай
manager -> warehouse: новая поставка

warehouse -> products_queue: добавлен новый продукт
note over products_queue: new_product_added {id,name,stocks}

products_queue -> catalog: добавлен новый продукт
end
```

### Продукт существует
```plantuml
actor Менеджер as manager
participant "warehouse" as warehouse
queue "stocks" as stocks_queue
participant "catalog" as catalog

alt Успешный случай
manager -> warehouse: новая поставка
warehouse -> stocks_queue: изменены остатки для продукта
note over stocks_queue: stocks_changed {id,stocks}

stocks_queue -> catalog: изменены остатки для продукта
end
```