# UC-1 Снизить остатки
Диаграмма последовательности показывает жизненный цикл объекта на единой временной оси, взаимодействие с другими обьектами и актерами информационной системы в рамках одного варианта использования.

```plantuml
participant "basket" as basket
queue "orders" as orders_queue
participant "warehouse" as warehouse

alt Успешный случай
basket -> orders_queue: Cоздан новый заказ
note over orders_queue: order_created {id,items[{id, quantity},{id, quantity}]}
end
```