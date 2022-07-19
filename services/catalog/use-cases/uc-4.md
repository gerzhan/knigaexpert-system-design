# UC-4 Добавление нового продукта

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