# UC-4 Назначение заказа

```plantuml
participant "delivery" as delivery
participant "courier app" as courier_app
actor Курьер as courier

loop
delivery -> delivery: Поиск свободного исполнителя
end
delivery -> courier_app: Назначение заказа на исполнителя
note over courier_app: Web Socket Message
courier_app -> courier: Уведомление исполнителя
```