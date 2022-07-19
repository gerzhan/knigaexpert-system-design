[[_TOC_]]

## Delivery
Позволяет формировать корзину.
Когда покупатель завершает покупки, то он может добавить адрес доставки,
оплатить и тем самым создать заказ.

## Container diagram
```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

skinparam wrapWidth 200
skinparam maxMessageSize 200

LAYOUT_TOP_DOWN()
LAYOUT_WITH_LEGEND()

Person(customer, Покупатель, "Совершает покупки")
Person(manager, Менеджер, "Управляет интернет магазином")
Person(courier, Курьер, "Доставляет заказ")

Container_Ext(ordering, "Ordering", ".Net, Docker", "Управление процессом оформления заказа")


System_Boundary(boundary, "Delivery") {
' Shop
Container(shop_app, "Shop", "Web, React", "Витрина интернет магазина")
Container_Ext(shop_bff, "Shop BFF", "Api Gateway, Ocelot", "Маршрутизация трафика c web приложения shop, аутентификацяи, авторизация")
Rel(shop_app, shop_bff, "Получить статус доставки", "HTTPS")
Rel(customer, shop_app, "Получить статус доставки", "HTTPS")

' Backoffice
Container(backoffice_app, "Backoffice", "Web, React", "Панель управления интернет магазином")  
Container_Ext(backoffice_bff, "Backoffice BFF", "Api Gateway, Ocelot", "Маршрутизация трафика, аутентификацяи, авторизация")
Rel(backoffice_app, backoffice_bff, "Получить статус доставки", "HTTPS")
Rel(manager, backoffice_app, "Получить статус доставки", "HTTPS")

' Сourier App
Container(courier_app, "Courier App", "Mobile, React Native", "Приложение курьера")  
Container_Ext(courier_bff, "Courier BFF", "Api Gateway, Ocelot", "Маршрутизация трафика, аутентификацяи, авторизация")
Rel_U(courier_app, courier_bff, "Изменить статус доставки", "HTTPS")
Rel_U(courier, courier_app, "Изменить статус доставки", "HTTPS")

Container(delivery, "Delivery", ".Net, Docker", "Управление процессом доставки заказа")
Rel_R(ordering, delivery, "Cоздан новый заказ", "Async, Kafka")
Rel(backoffice_bff, delivery, "Получить статус доставки", "HTTPS")
Rel(shop_bff, delivery, "Получить статус доставки", "HTTPS")
Rel(delivery, courier_app, "Назначить заказ на исполнителя", "Web Socket")
Rel_U(courier_bff, delivery, "Изменить статус доставки", "HTTPS")
}
```

## Use case diagram
```plantuml
left to right direction
skinparam packageStyle rectangle

actor Покупатель as client
actor Менеджер as manager
actor Курьер as courier
actor Ordering as ordering << Система >>

rectangle Delivery {
  usecase (UC-1 Принятие заказа) as UC2
  usecase (UC-2 Изменение статуса заказа на 'Доставлен') as UC4
  usecase (UC-3 Получение статуса доставки) as UC1
  usecase (UC-4 Назначение заказа) as UC3
  

  url of UC1 is [[use-cases/uc-1.md]]
  url of UC2 is [[use-cases/uc-2.md]]
  url of UC3 is [[use-cases/uc-3.md]]
  url of UC4 is [[use-cases/uc-4.md]]
}

ordering -->UC2

UC1 <-- client 
UC1 <--manager
UC2 --> UC3

UC3 -->courier
UC4 <--courier
```
## Use cases
- [UC-1](use-cases/uc-1.md) Принятие заказа.
- [UC-2](use-cases/uc-2.md) Изменение статуса заказа на 'Доставлен'.
- [UC-3](use-cases/uc-3.md) Получение статуса доставки.
- [UC-4](use-cases/uc-4.md) Назначение заказа.

