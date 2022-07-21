[[_TOC_]]

## Catalog
Позволяет отображать каталог товаров.
Так же отвечает за отображение карточки товара.

## Container diagram
```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

skinparam wrapWidth 200
skinparam maxMessageSize 200

LAYOUT_TOP_DOWN()
LAYOUT_WITH_LEGEND()

Person(customer, Покупатель, "Совершает покупки")
Person(manager, Менеджер, "Управляет интернет магазином")

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

## Use case diagram
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

