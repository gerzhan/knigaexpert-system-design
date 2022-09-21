[[_TOC_]]

## Delivery
Позволяет формировать корзину.
Когда покупатель завершает покупки, то он может добавить адрес доставки,
оплатить и тем самым создать заказ.

## Container diagram
Диаграмма контейнеров показывает высокоуровневую архитектуру программного обеспечения и то, как в ней распределяются обязанности. Она также показывает основные используемые технологии и то, как контейнеры взаимодействуют друг с другом. Это простая схема высокого уровня, ориентированная на технологии, которая одинаково полезна как для разработчиков программного обеспечения, так и для персонала службы поддержки и эксплуатации.

```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

skinparam wrapWidth 200
skinparam maxMessageSize 200
LAYOUT_TOP_DOWN()
LAYOUT_WITH_LEGEND()

!include https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/actors/customer.puml
!include https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/actors/manager.puml
!include https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/actors/courier.puml

System_Boundary(boundary, "Delivery") {
' Shop
!include https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/gateways/shop/shop.puml
!include https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/gateways/shop/gateway.puml
Rel(customer, shop_app, "Получить статус доставки", "HTTPS")

' Backoffice
!include https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/gateways/backoffice/backoffice.puml
!include https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/gateways/backoffice/gateway.puml
Rel(manager, backoffice_app, "Получить статус доставки", "HTTPS")

' Сourier App
!include https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/gateways/courier/courier_app.puml
!include https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/gateways/courier/gateway.puml
Rel(courier, courier_app, "Изменить статус доставки", "HTTPS")

!include https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/services/delivery/normal.puml
!include https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/services/delivery/db.puml
Rel(backoffice_bff, delivery, "Получить статус доставки", "HTTPS")
Rel(shop_bff, delivery, "Получить статус доставки", "HTTPS")
Rel(delivery, courier_bff, "Назначить заказ на исполнителя", "Web Socket")
Rel_R(courier_bff, delivery, "Изменить статус доставки", "HTTPS")
}

!include https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/services/ordering/ext.puml
Rel_R(ordering_ext, delivery, "Cоздан новый заказ", "Async, Kafka")
```

## Component diagram
Диаграмма компонентов показывает, из каких «компонентов» состояит контейнер, что представляет собой каждый из этих компонентов, его обязанности, технологии и детали реализации.

```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Component.puml
LAYOUT_WITH_LEGEND()

Container(spa, "Single Page Application", "javascript and angular", "Provides all the internet banking functionality to customers via their web browser.")
Container(ma, "Mobile App", "Xamarin", "Provides a limited subset ot the internet banking functionality to customers via their mobile mobile device.")
ContainerDb(db, "Database", "Relational Database Schema", "Stores user registration information, hashed authentication credentials, access logs, etc.")
System_Ext(mbs, "Mainframe Banking System", "Stores all of the core banking information about customers, accounts, transactions, etc.")

Container_Boundary(boundary, "Warehouse") {
    Component(sign, "Sign In Controller", "MVC Rest Controlle", "Allows users to sign in to the internet banking system")
    Component(accounts, "Accounts Summary Controller", "MVC Rest Controller", "Provides customers with a summary of their bank accounts")
    Component(security, "Security Component", "Spring Bean", "Provides functionality related to singing in, changing passwords, etc.")
    Component(mbsfacade, "Mainframe Banking System Facade", "Spring Bean", "A facade onto the mainframe banking system.")

    Rel(sign, security, "Uses")
    Rel(accounts, mbsfacade, "Uses")
    Rel(security, db, "Read & write to", "JDBC")
    Rel(mbsfacade, mbs, "Uses", "XML/HTTPS")
}

Rel(spa, sign, "Uses", "JSON/HTTPS")
Rel(spa, accounts, "Uses", "JSON/HTTPS")

Rel(ma, sign, "Uses", "JSON/HTTPS")
Rel(ma, accounts, "Uses", "JSON/HTTPS")
```

## Use case diagram
Диаграмма вариантов использования показывает, какой функционал разрабатываемой программной системы доступен каждой группе пользователей.

```plantuml
left to right direction
skinparam packageStyle rectangle

actor Покупатель as client
actor Менеджер as manager
actor Курьер as courier
actor Ordering as ordering << Система >>

rectangle Delivery {
  usecase (UC-1 Принятие заказа) as UC1
  usecase (UC-2 Изменение статуса заказа на 'Доставлен') as UC2
  usecase (UC-3 Получение статуса доставки) as UC3
  usecase (UC-4 Назначение заказа) as UC4
  

  url of UC1 is [[use-cases/uc-1.md]]
  url of UC2 is [[use-cases/uc-2.md]]
  url of UC3 is [[use-cases/uc-3.md]]
  url of UC4 is [[use-cases/uc-4.md]]
}

ordering --> UC1

UC2 <-- courier
UC4 --> courier

client --> UC3 

UC3 <--manager
```
## Use cases
- [UC-1](use-cases/uc-1.md) Принятие заказа.
- [UC-2](use-cases/uc-2.md) Изменение статуса заказа на 'Доставлен'.
- [UC-3](use-cases/uc-3.md) Получение статуса доставки.
- [UC-4](use-cases/uc-4.md) Назначение заказа.

