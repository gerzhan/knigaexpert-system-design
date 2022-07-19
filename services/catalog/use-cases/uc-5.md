# UC-5 Изменение остатков существующего продукта

```plantuml
participant "warehouse" as warehouse
queue "stocks" as stocks_queue
participant "catalog" as catalog

alt Успешный случай
warehouse -> stocks_queue: изменены остатки для продукта
note over stocks_queue: stocks_changed {id,stocks}

stocks_queue -> catalog: изменены остатки для продукта
end
```