# UC-4 Добавление нового продукта

```plantuml
participant "warehouse" as warehouse
queue "products" as products_queue
participant "catalog" as catalog

alt Успешный случай
warehouse -> products_queue: добавлен новый продукт
note over products_queue: new_product_added {id,name,stocks}

products_queue -> catalog: добавлен новый продукт
end
```