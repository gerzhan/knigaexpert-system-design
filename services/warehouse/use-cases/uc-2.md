# UC-2 Снижение остатков

```plantuml
participant "ordering" as ordering
queue "orders" as orders_queue
participant "warehouse" as warehouse

alt Успешный случай
ordering -> orders_queue: создан новый заказ
note over orders_queue: order_created {id,items[{id, quantity},{id, quantity}]}
end
```