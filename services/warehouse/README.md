[[_TOC_]]

## Warehouse
Автоматизирует бизнес процессы склада.
Отвечает за храниение и отгрузку товаров.
Позволяет искать товары, а так же принимать поставки от поставщиков.

## Container diagram
Диаграмма контейнера показывает высокоуровневую форму архитектуры программного обеспечения и то, как в ней распределяются обязанности. Он также показывает основные варианты технологий и то, как контейнеры взаимодействуют друг с другом. Это простая схема высокого уровня, ориентированная на технологии, которая одинаково полезна как для разработчиков программного обеспечения, так и для персонала службы поддержки и эксплуатации.

```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

skinparam wrapWidth 200
skinparam maxMessageSize 200

LAYOUT_TOP_DOWN()
LAYOUT_WITH_LEGEND()

Person(manager, Менеджер, "Принимает поставки")


System_Boundary(boundary, "Warehouse") {

' Backoffice
Container(backoffice_app, "Backoffice", "Web, React", "Панель управления интернет магазином")  
Container_Ext(backoffice_bff, "Backoffice BFF", "Api Gateway, Ocelot", "Маршрутизация трафика, аутентификацяи, авторизация")
Rel(backoffice_app, backoffice_bff, "Принятие поставки", "HTTPS")
Rel(manager, backoffice_app, "Принятие поставки", "HTTPS")


' Services
Container(warehouse, "Warehouse", ".Net, Docker", "Управление складом")
ContainerDb(warehouse_db, "Database", "Postgre SQL", "Схема склада, товары и т.п.")
Rel(warehouse, warehouse_db, "Чтение / Запись", "Sync, TCP")
Rel(backoffice_bff, warehouse, "Принятие поставки", "HTTP")
}

Container_Ext(catalog, "Catalog", ".Net, Docker", "Управление каталогом витрины")
Rel_L(warehouse, catalog, "Добавлен новый продукт", "Async, Kafka")
Rel_L(warehouse, catalog, "Изменены остатки существующего продукта", "Async, Kafka")

Container_Ext(ordering, "Ordering", ".Net, Docker", "Управление процессом оформления заказа")
Rel_L(ordering, warehouse, "Cоздан новый заказ", "Async, Kafka")

```

## Component diagram
Диаграмма компонентов показывает, как контейнер состоит из ряда «компонентов», что представляет собой каждый из этих компонентов, их обязанности и детали технологии/реализации.

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
```plantuml
left to right direction
skinparam packageStyle rectangle

actor Менеджер as manager
actor Ordering as ordering << Система >>

rectangle Warehouse {
  usecase (UC-1 Принятие поставки) as UC1
  usecase (UC-2 Снижение остатков) as UC2

  url of UC1 is [[use-cases/uc-1.md]]
  url of UC2 is [[use-cases/uc-2.md]]
}

manager --> UC1

UC2<-- ordering
```
## Use cases
- [UC-1](use-cases/uc-1.md) Принятие поставки.
- [UC-2](use-cases/uc-2.md) Снижение остатков.

