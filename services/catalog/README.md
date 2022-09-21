[[_TOC_]]

## Catalog
Позволяет отображать каталог товаров.
Так же отвечает за отображение карточки товара.

## Container diagram
Диаграмма контейнеров показывает высокоуровневую архитектуру программного обеспечения и то, как в ней распределяются обязанности. Она также показывает основные используемые технологии и то, как контейнеры взаимодействуют друг с другом. Это простая схема высокого уровня, ориентированная на технологии, которая одинаково полезна как для разработчиков программного обеспечения, так и для персонала службы поддержки и эксплуатации.

```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml
' Components
!define actors https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/actors
!define frontends https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/frontends  
!define services https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/services

skinparam wrapWidth 200
skinparam maxMessageSize 200
LAYOUT_TOP_DOWN()
LAYOUT_WITH_LEGEND()

!include actors/customer.puml
!include actors/manager.puml

!include frontends/shop/shop.puml
!include frontends/shop/gateway.puml

!include frontends/backoffice/backoffice.puml
!include frontends/backoffice/gateway.puml

!include services/warehouse/ext.puml

System_Boundary(boundary, "Catalog") {
Rel(customer, shop_app, "Просматривает каталог, карточку товара", "HTTPS")
Rel(manager, backoffice_app, "Изменение цены, описания продукта", "HTTPS")

' Service
!include services/catalog/normal.puml
!include services/catalog/db.puml

Rel_L(warehouse_ext, catalog, "Добавлен новый продукт", "Async, Kafka")
Rel_L(warehouse_ext, catalog, "Изменены остатки существующего продукта", "Async, Kafka")
Rel(shop_bff, catalog, "Просматривает каталог, карточку товара", "HTTP")
Rel(backoffice_bff, catalog, "Изменение цены, описания продукта", "HTTP")
}
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
actor Warehouse as warehouse << Система >>

rectangle Catalog {
  usecase (UC-1 Просмотр каталога продуктов) as UC1
  usecase (UC-2 Посмотр карточки продукта) as UC2
  usecase (UC-3 Изменение цены, описания продукта) as UC3
  usecase (UC-4 Добавление нового продукта) as UC4
  usecase (UC-5 Изменение остатков существующего продукта) as UC5

  url of UC1 is [[use-cases/uc-1.md]]
  url of UC2 is [[use-cases/uc-2.md]]
  url of UC3 is [[use-cases/uc-3.md]]
  url of UC4 is [[use-cases/uc-4.md]]
  url of UC5 is [[use-cases/uc-5.md]]
}

client --> UC1
client --> UC2

UC1 <-- manager
UC2 <-- manager
UC3 <-- manager

UC4<-- warehouse
UC5<-- warehouse
```
## Use cases
- [UC-1](use-cases/uc-1.md) Просмотр каталога продуктов.
- [UC-2](use-cases/uc-2.md) Посмотр карточки продукта.
- [UC-3](use-cases/uc-3.md) Изменение цены, описания продукта.
- [UC-4](use-cases/uc-4.md) Добавление нового продукта.
- [UC-5](use-cases/uc-5.md) Изменение остатков существующего продукта.

