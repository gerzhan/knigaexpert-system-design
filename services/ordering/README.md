[[_TOC_]]

## Ordering
Позволяет формировать корзину.
Когда покупатель завершает покупки, то он может добавить адрес доставки,
оплатить и тем самым создать заказ.

## Container diagram
Диаграмма контейнеров показывает высокоуровневую архитектуру программного обеспечения и то, как в ней распределяются обязанности. Она также показывает основные используемые технологии и то, как контейнеры взаимодействуют друг с другом. Это простая схема высокого уровня, ориентированная на технологии, которая одинаково полезна как для разработчиков программного обеспечения, так и для персонала службы поддержки и эксплуатации.

```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/actors/customer.puml

skinparam wrapWidth 200
skinparam maxMessageSize 200
LAYOUT_TOP_DOWN()
LAYOUT_WITH_LEGEND()

System_Boundary(boundary, "Ordering") {
!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/gateways/shop/shop.puml
!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/gateways/shop/gateway.puml
!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/services/ordering/normal.puml
!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/services/ordering/db.puml

Rel(customer, shop_app, "Формирует корзину, делает заказ", "HTTPS")
Rel(shop_bff, ordering, "Формирует корзину, делает заказ", "HTTPS")
}

!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/services/warehouse/ext.puml
!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/services/delivery/ext.puml
!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/services/payment/ext.puml

Rel(ordering, warehouse_ext, "Cоздан новый заказ", "Async, Kafka")
Rel(ordering, delivery_ext, "Cоздан новый заказ", "Async, Kafka")
Rel_R(ordering, payment_ext, "Оплата заказа", "Sync, gRPC")
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

rectangle Ordering {
  usecase (UC-1 Добавление товара в корзину) as UC1
  usecase (UC-2 Удаление товара из корзины) as UC2
  usecase (UC-3 Просмотр списка товаров в корзине) as UC3
  usecase (UC-4 Добавление адреса доставки) as UC4
  usecase (UC-5 Оплата заказа) as UC5

  url of UC1 is [[use-cases/uc-1.md]]
  url of UC2 is [[use-cases/uc-2.md]]
  url of UC3 is [[use-cases/uc-3.md]]
  url of UC4 is [[use-cases/uc-4.md]]
  url of UC5 is [[use-cases/uc-5.md]]
}

client --> UC1
client --> UC2
client --> UC3
client --> UC4
client --> UC5
```
## Use cases
- [UC-1](use-cases/uc-1.md) Добавление товара в корзину.
- [UC-2](use-cases/uc-2.md) Удаление товара из корзины.
- [UC-3](use-cases/uc-3.md) Просмотр списка товаров в корзине.
- [UC-4](use-cases/uc-4.md) Добавление адреса доставки.
- [UC-5](use-cases/uc-5.md) Оплата заказа.

