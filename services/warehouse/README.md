[[_TOC_]]

## Warehouse
Автоматизирует бизнес процессы склада.
Отвечает за храниение и отгрузку товаров.
Позволяет искать товары, а так же принимать поставки от поставщиков.

## Container diagram
> Диаграмма контейнеров показывает высокоуровневую архитектуру программного обеспечения и то, как в ней распределяются обязанности. Она также показывает основные используемые технологии и то, как контейнеры взаимодействуют друг с другом. Это простая схема высокого уровня, ориентированная на технологии, которая одинаково полезна как для разработчиков программного обеспечения, так и для персонала службы поддержки и эксплуатации.

```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml
skinparam wrapWidth 200
skinparam maxMessageSize 200
LAYOUT_TOP_DOWN()
LAYOUT_WITH_LEGEND()

!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/actors/manager.puml

System_Boundary(boundary, "Warehouse") {
!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/gateways/backoffice/backoffice.puml
!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/gateways/backoffice/gateway.puml
Rel(manager, backoffice_app, "Принять поставку", "HTTPS")

!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/services/warehouse/normal.puml
!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/services/warehouse/db.puml
Rel(backoffice_bff, warehouse, "Принять поставку", "HTTP")
}

!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/services/catalog/ext.puml
Rel_L(warehouse, catalog_ext, "Добавлен новый продукт", "Async, Kafka")
Rel_L(warehouse, catalog_ext, "Изменены остатки существующего продукта", "Async, Kafka")

!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/services/ordering/ext.puml
Rel_L(ordering_ext, warehouse, "Cоздан новый заказ", "Async, Kafka")
```

## Component diagram
> Диаграмма компонентов показывает из каких «компонентов» состояит контейнер, что представляет собой каждый из этих компонентов, его обязанности, технологии и детали реализации.

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

## Code diagram
> Диаграмма классов показывает общую структуру иерархии классов системы, их коопераций, атрибутов, методов, интерфейсов и взаимосвязей между ними.

```plantuml
package "Warehouse Aggregate"  #DDDDDD {
  Class Warehouse <Aggregate>
  {
    - Places[] Places

    + Warehouse()
    + Place FindPlace(Good good)
    + Place GetPlaceByLocation(Location location)
    + Place GetPlaceByCategory(Category category)
    + Load(Category category, Pile pile)
    + TakeOne(Category category)
  }
  
  Class Place <Entity>
  {
    - Warehouse Warehouse
    - Location Location
    - Category Category
    - Pile item

    + Place(Warehouse warehouse, Location location, Category category)
    + SetPile(Pile pile)
  }  

  Class Pile <Value Object>{
    - Good Good
    - int Quantity
    + SubtractOne()
  }

  Class Location <Value Object> {
    - int Row
    - int Shelf 
  }

  Warehouse *- Place
  Place *- Location
  Place *-- Pile
}

package "SharedKernel" #DDDDDD {
  Class Category <Value Object>
  {
    - string Name
    + Category(string name)
  }  
  Place *- Category 

  Class Weight <Value Object>
  {
    - int Gram
    + Weight(int gram)
  }
}

package "Good Aggregate" #DDDDDD {
  Class Good <Aggregate>
  {
    - uuid Id
    - string Title
    - string Description
    - Weight Weight
    - Category Category
    + Good(Guid id, string title, string description, Weight weight, Category category
  }
  Good *-- Weight
  Good *- Category
  Pile *-- Good
}
```

## ER diagram
> Диаграмма отношений сущностей это визуальное представление базы данных, которое показывает, как связаны элементы внутри. Диаграмма ER состоит из двух типов объектов — сущностей и отношений.

```plantuml
entity Warehouses {
  * id : uuid <<PK>>
}

entity Places {
  * id : uuid <<PK>>
  * maximum_available_weight : int
  * location_row : int
  * location_shelf : int
  * id : warehouse_id <<FK>>
  * item_id : uuid <<FK>>
}

entity Piles {
  * id : uuid <<PK>>
  * goodId : uuid <<FK>>
  * weight : int
  * quantity : int
}

entity Goods {
  * id : uuid <<PK>>
  * title : string
  * description : description
  * weight : int
}

Warehouses ||-  Places
Places }o- Piles
Piles }o--|| Goods
```
## Use case diagram
> Диаграмма вариантов использования показывает, какой функционал разрабатываемой программной системы доступен каждой группе пользователей.

```plantuml
left to right direction
skinparam packageStyle rectangle

actor Менеджер as manager
actor Ordering as ordering << Система >>

rectangle Warehouse {
  usecase (UC-1 Снизить остатки) as UC1
  usecase (UC-2 Принять поставку) as UC2
  usecase (UC-3 Получить все товары) as UC3
  usecase (UC-4 Получить информацию о товаре) as UC4
  usecase (UC-5 Списать товар) as UC5
  usecase (UC-6 Переместить товар) as UC6
  usecase (UC-7 Получить схему склада) as UC7
  usecase (UC-8 Найти место хранения товара) as UC8
  
  url of UC1 is [[use-cases/uc-1.md]]
  url of UC2 is [[use-cases/uc-2.md]]
  url of UC3 is [[use-cases/uc-3.md]]
  url of UC4 is [[use-cases/uc-4.md]]
  url of UC5 is [[use-cases/uc-5.md]]
  url of UC6 is [[use-cases/uc-6.md]]
  url of UC7 is [[use-cases/uc-7.md]]
  url of UC8 is [[use-cases/uc-8.md]]
  
}

UC1<-- ordering

manager --> UC1
manager --> UC2
manager --> UC3
manager --> UC4
manager --> UC5
manager --> UC6
manager --> UC7
manager --> UC8
```
**Use cases**
- [UC-1](use-cases/uc-1.md) Снизить остатки.
- [UC-2](use-cases/uc-2.md) Принять поставку.
- [UC-3](use-cases/uc-3.md) Получить все товары.
- [UC-4](use-cases/uc-4.md) Получить информацию о товаре.
- [UC-5](use-cases/uc-5.md) Списать товар.
- [UC-6](use-cases/uc-6.md) Переместить товар.
- [UC-7](use-cases/uc-7.md) Получить схему склада.
- [UC-8](use-cases/uc-8.md) Найти место хранения товара.


