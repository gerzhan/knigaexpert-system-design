[[_TOC_]]

## Catalog
Позволяет отображать каталог товаров.
Так же отвечает за отображение карточки товара.

## Container diagram
Диаграмма контейнеров показывает высокоуровневую архитектуру программного обеспечения и то, как в ней распределяются обязанности. Она также показывает основные используемые технологии и то, как контейнеры взаимодействуют друг с другом. Это простая схема высокого уровня, ориентированная на технологии, которая одинаково полезна как для разработчиков программного обеспечения, так и для персонала службы поддержки и эксплуатации.

```plantuml
' !include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/actors/customer.puml
!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/actors/manager.puml

skinparam wrapWidth 200
skinparam maxMessageSize 200

LAYOUT_TOP_DOWN()
LAYOUT_WITH_LEGEND()

Container_Ext(warehouse, "Warehouse", ".Net, Docker", "Управление складом")

System_Boundary(boundary, "Catalog") {
' Shop
Container(shop_app, "Shop", "Web, React", "Витрина интернет магазина")
Container_Ext(shop_bff, "Shop BFF", "Api Gateway, Ocelot", "Маршрутизация трафика c web приложения shop, аутентификацяи, авторизация")
Rel(shop_app, shop_bff, "Просматривает каталог, карточку товара", "HTTPS")
Rel(customer, shop_app, "Просматривает каталог, карточку товара", "HTTPS")

' Backoffice
Container(backoffice_app, "Backoffice", "Web, React", "Панель управления интернет магазином")  
Container_Ext(backoffice_bff, "Backoffice BFF", "Api Gateway, Ocelot", "Маршрутизация трафика, аутентификацяи, авторизация")
Rel(backoffice_app, backoffice_bff, "Изменение цены, описания продукта", "HTTPS")
Rel(manager, backoffice_app, "Изменение цены, описания продукта", "HTTPS")


Container(catalog, "Catalog", ".Net, Docker", "Управление каталогом витрины")
ContainerDb(catalog_db, "Database", "Postgre SQL", "Категории, товары и т.п.")
Rel(catalog, catalog_db, "Чтение / Запись", "Sync, TCP")
Rel_L(warehouse, catalog, "Добавлен новый продукт", "Async, Kafka")
Rel_L(warehouse, catalog, "Изменены остатки существующего продукта", "Async, Kafka")
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

