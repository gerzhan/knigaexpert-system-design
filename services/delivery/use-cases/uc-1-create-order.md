# UC-1 Создать заказ

```plantuml
participant "basket" as basket
queue "basket_confirmed_topic" as basket_confirmed_topic
participant "order service" as order_service

basket -> basket_confirmed_topic: Cоздан новый заказ
note over basket_confirmed_topic: basket_confirmed {id,items[{id, quantity},{id, quantity}]}
basket_confirmed_topic -> order_service: Cоздан новый заказ
order_service -> order_service: Заказ сохранен
```