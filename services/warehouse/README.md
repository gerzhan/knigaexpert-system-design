[[_TOC_]]

## О сервисе
Автоматизирует бизнес процессы склада.
Отвечает за храниение и отгрузку товаров.
Позволяет искать товары, а так же принимать поставки от поставщиков.

## Container diagram
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
Rel(backoffice_bff, warehouse, "Принятие поставки", "HTTPS")
}

Container_Ext(catalog, "Catalog", ".Net, Docker", "Управление каталогом витрины")
Rel_L(warehouse, catalog, "Добавлен новый продукт", "Async, Kafka")
Rel_L(warehouse, catalog, "Изменены остатки существующего продукта", "Async, Kafka")

Container_Ext(ordering, "Ordering", ".Net, Docker", "Управление процессом оформления заказа")
Rel(ordering, warehouse, "Cоздан новый заказ", "Async, Kafka")

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

