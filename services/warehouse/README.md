[[_TOC_]]

# Warehouse
Автоматизирует бизнес процессы склада.
Отвечает за храниение и отгрузку товаров.
Позволяет искать товары, а так же принимать поставки от поставщиков.

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

!include actors/manager.puml

System_Boundary(boundary, "Warehouse") {
!include frontends/backoffice/web_app.puml
!include frontends/backoffice/gateway.puml
Rel(manager, backoffice_web_app, "Принять поставку", "HTTPS")

!include services/warehouse/normal.puml
!include services/warehouse/db.puml
Rel(backoffice_bff, warehouse, "Принять поставку", "HTTP")
}

!include services/catalog/ext.puml
Rel_L(warehouse, catalog_ext, "Добавлен новый продукт", "Async, Kafka")
Rel_L(warehouse, catalog_ext, "Изменены остатки существующего продукта", "Async, Kafka")

!include services/basket/ext.puml
Rel_L(basket_ext, warehouse, "Cоздан новый заказ", "Async, Kafka")
```

## Component diagram
Диаграмма компонентов показывает, из каких «компонентов» состояит контейнер, что представляет собой каждый из этих компонентов, его обязанности, технологии и детали реализации.

```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Component.puml
LAYOUT_WITH_LEGEND()

Container_Ext(api_client, "API Client", " HTTP REST", "Внешний потребитель API")

Container_Ext(message_bus, "Message Bus", "Kafka", "Сообщения")

Container_Boundary(warehouse_service, "Warehouse") {
    Container_Boundary(api_layer, "API Layer") {
    Component(warehouse_handler, "WarehouseHandler", "HTTP Handler", "Обрабатывает HTTP запросы, извлекает параметры из них")
    
    Component(basket_consumer, "BasketConsumer", "Kafka Consumer", "Обрабатывает Message из Kafka")
    }

    Container_Boundary(application_layer, "Application Layer") {
      Container_Boundary(commands, "Commands") {
        Component(uc4_take_one_good_command, "UC-4 TakeOneGood", "", "Снизить остатки")
        Component(uc2_load_goods_command, "UC-2 LoadGoods", "", "Принять поставку")
      }

      Container_Boundary(queries, "Queries") {
        Component(uc3_get_all_goods_query, "UC-3 GetAllGoods", "", "Получить все виды товаров")

        Component(uc4_get_good_by_id_query, "UC-4 GetGoodById", "", "Получить информацию о товаре")

        Component(uc5_get_all_places_query, "UC-5 GetAllPlaces", "", "Получить схему склада")
      }
    }

    Container_Boundary(domain_layer, "Domain Layer") {
      Component(warehouse_aggregate, "Warehouse", "Aggregate", "Склад")
      Component(good_aggregate, "Good", "Aggregate", "Описание товара, остатки")
    }

    Container_Boundary(infrastructure_layer, "Infrastructure Layer") {
      Component(warehouse_aggregate_repository, "WarehouseRepository", "", "Сохранение/восстановление аггрегата")
      Component(good_aggregate_repository, "GoodRepository", "", "Сохранение/восстановление аггрегата")
    }

    Rel(message_bus, basket_consumer, "Оформлен новый заказ, куплены такие-то продукты", "Async, Kafka")
    
    Rel(api_client, warehouse_handler, "Просматривает каталог, карточку товара", "HTTP")
    Rel(api_client, warehouse_handler, "Изменение цены, описания продукта", "HTTP")

    Rel_D(api_layer, application_layer, "Uses")
    
    Rel_D(commands, domain_layer, "Uses")

    Rel_R(warehouse_aggregate, good_aggregate, "Uses")

    Rel_D(commands, infrastructure_layer, "Uses")
}

ContainerDb(db, "WarehouseDb", "Postgres", "Хранит информацию о складе и товаров в нем")
Rel_D(infrastructure_layer, db, "Uses")
Rel_D(queries, db, "Uses")
```

## Code diagram
Диаграмма классов показывает общую структуру иерархии классов системы, их коопераций, атрибутов, методов, интерфейсов и взаимосвязей между ними.

```plantuml
package "Warehouse Aggregate"  #DDDDDD {
  Class Warehouse <Aggregate>
  {
    - uuid Id
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
    - uuid Id
    - Warehouse Warehouse
    - Location Location
    - Category Category
    - Pile item

    + Place(Warehouse warehouse, Location location, Category category)
    + SetPile(Pile pile)
  }

  Class Category <Entity>
  {
    - uuid Id
    - string Name
    + Category(string name)
  }  
  Place *-- Category 

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

package "Good Aggregate" #DDDDDD {
  Class Good <Aggregate>
  {
    - uuid Id
    - string Title
    - string Description
    - Weight Weight
    + Good(Guid id, string title, string description, Weight weight)
  }
  Good *-- Weight
  Pile *-- Good

  Class Weight <Value Object>
  {
    - int Gram
    + Weight(int gram)
  }

}
```

## ER diagram
Диаграмма отношений сущностей это визуальное представление базы данных, которое показывает, как связаны элементы внутри. Диаграмма ER состоит из двух типов объектов — сущностей и отношений.

```plantuml
entity Warehouses {
  * id : uuid <<PK>>
}

entity Places {
  * id : uuid <<PK>>
  * warehouse_id <<FK>>
  * location_row : int
  * location_shelf : int
  * category_id_ : uuid
  * pile_good_id : uuid
  * pile_quantity : string
}

entity Categories {
  * id : uuid <<PK>>
  * category_name : string
}

entity Goods {
  * id : uuid <<PK>>
  * title : string
  * description : description
  * weight_gram : int
}

Places ||--  Categories
Warehouses ||-  Places
Places }o--|| Goods
```
## Use case diagram
Диаграмма вариантов использования показывает, какой функционал разрабатываемой программной системы доступен каждой группе пользователей.

```plantuml
left to right direction
skinparam packageStyle rectangle

actor Менеджер as manager
actor Basket as basket << Система >>

rectangle Warehouse {
  usecase (UC-1 Снизить остатки) as UC1
  usecase (UC-2 Принять поставку) as UC2
  usecase (UC-3 Получить все виды товаров) as UC3
  usecase (UC-4 Получить информацию о товаре) as UC4
  usecase (UC-5 Получить схему склада) as UC5
  
  url of UC1 is [[use-cases/uc-1.md]]
  url of UC2 is [[use-cases/uc-2.md]]
  url of UC3 is [[use-cases/uc-3.md]]
  url of UC4 is [[use-cases/uc-4.md]]
  url of UC5 is [[use-cases/uc-5.md]]
}

UC1<-- basket

manager --> UC1
manager --> UC2
manager --> UC3
manager --> UC4
manager --> UC5
```
**Use cases**
- [UC-1](use-cases/uc-1.md) Снизить остатки.
- [UC-2](use-cases/uc-2.md) Принять поставку.
- [UC-3](use-cases/uc-3.md) Получить все виды товаров.
- [UC-4](use-cases/uc-4.md) Получить информацию о товаре.
- [UC-6](use-cases/uc-5.md) Получить схему склада.


