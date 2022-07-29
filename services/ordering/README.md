[[_TOC_]]

## Ordering
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

Person(customer, Покупатель, "Совершает покупки")

System_Boundary(boundary, "Ordering") {
' Shop
Container(shop_app, "Shop", "Web, React", "Витрина интернет магазина")
Container_Ext(shop_bff, "Shop BFF", "Api Gateway, Ocelot", "Маршрутизация трафика c web приложения shop, аутентификацяи, авторизация")
Rel(shop_app, shop_bff, "Формирует корзину, делает заказ", "HTTPS")
Rel(customer, shop_app, "Формирует корзину, делает заказ", "HTTPS")

Container(ordering, "Ordering", ".Net, Docker", "Управление процессом оформления заказа")
ContainerDb(ordering_db, "Database", "Postgre SQL", "Корзина, товары и т.п.")
Rel(ordering, ordering_db, "Чтение / Запись", "Sync, TCP")
Rel(shop_bff, ordering, "Формирует корзину, делает заказ", "HTTPS")
}

Container_Ext(warehouse, "Warehouse", ".Net, Docker", "Управление складом")
Rel(ordering, warehouse, "Cоздан новый заказ", "Async, Kafka")

Container_Ext(delivery, "Delivery", ".Net, Docker", "Управление процессом доставки заказа")
Rel(ordering, delivery, "Cоздан новый заказ", "Async, Kafka")

Container_Ext(payment, "Payment", ".Net, Docker", "Управление процессом оплаты")
Rel_R(ordering, payment, "Оплата заказа", "Sync, gRPC")
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

