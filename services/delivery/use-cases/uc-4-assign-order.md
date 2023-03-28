# UC-4 Назначить заказ на курьера

```plantuml
participant "order service" as order_service
participant "courier app" as courier_app
actor Курьер as courier

loop
order_service -> order_service: Поиск свободного курьера
end
order_service -> courier_app: Назначение заказа на курьера
note over courier_app: Web Socket Message
courier_app -> courier: Уведомление курьера
```