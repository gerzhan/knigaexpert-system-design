# UC-1 Принятие заказа

```plantuml
participant "basket" as basket
queue "orders" as orders_queue
participant "delivery" as delivery

basket -> orders_queue: Cоздан новый заказ
note over orders_queue: order_created {id,items[{id, quantity},{id, quantity}]}
orders_queue -> delivery: Cоздан новый заказ
delivery -> delivery: Заказ сохранен
```