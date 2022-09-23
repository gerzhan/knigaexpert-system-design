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

!include frontends/shop/web_app.puml
!include frontends/shop/gateway.puml

!include frontends/backoffice/web_app.puml
!include frontends/backoffice/gateway.puml

!include services/warehouse/ext.puml

System_Boundary(boundary, "Catalog") {
Rel(customer, shop_web_app, "Просматривает каталог, карточку товара", "HTTPS")
Rel(manager, backoffice_web_app, "Изменение цены, описания продукта", "HTTPS")

' Service
!include services/catalog/normal.puml
!include services/catalog/db.puml

Rel_L(warehouse_ext, catalog, "Добавлен новый продукт/изменены остатки существющего продукта", "Async, Kafka")
Rel(shop_bff, catalog, "Просматривает каталог, карточку товара", "HTTP")
Rel(backoffice_bff, catalog, "Изменение цены, описания продукта", "HTTP")
}
```

## Component diagram
Диаграмма компонентов показывает, из каких «компонентов» состояит контейнер, что представляет собой каждый из этих компонентов, его обязанности, технологии и детали реализации.

```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Component.puml
LAYOUT_WITH_LEGEND()

Container_Ext(api_client, "API Client", " HTTP REST", "Внешний потребитель API")

Container_Ext(message_bus, "Message Bus", "Kafka", "Transport for business events")

Container_Boundary(catalog_service, "Catalog") {
    Container_Boundary(api_layer, "API Layer") {
    Component(catalog_controller, "CatalogController", "Web API Controller", "Обрабатывает HTTP запросы, извлекает параметры из них")
    
    Component(goods_consumer, "GoodsConsumer", "Kafka Consumer", "Обрабатывает Message из Kafka")
    }

    Container_Boundary(application_layer, "Application Layer") {
      Container_Boundary(commands, "Commands") {
        Component(uc4_upload_good_command, "UC-4 UploadGoodCommand", "", "Добавление/изменение остатков продукта")
        Component(uc3_edit_good_command, "UC-3 EditGoodCommand", "", "Изменение цены, описания продукта")
      }

      Container_Boundary(queries, "Queries") {
        Component(uc1_get_catalog_query, "UC-1 GetCatalogQuery", "", "Просмотр каталога продуктов")

        Component(uc2_get_good_details_query, "UC-2 GetGoodDetailsQuery", "", "Посмотр карточки продукта")
      }
    }

    Container_Boundary(domain_layer, "Domain Layer") {
      Component(catalog_aggregate, "Catalog", "Aggregate", "Категории и товары в них")
      Component(good_aggregate, "Good", "Aggregate", "Описание товара, цена, остатки")
    }

    Container_Boundary(infrastructure_layer, "Infrastructure Layer") {
      Component(catalog_aggregate_repository, "CatalogRepository", "", "Provides functionality related to singing in, changing passwords, etc.")
      Component(good_aggregate_repository, "GoodRepository", "", "A facade onto the mainframe banking system.")
    }

    Rel(message_bus, goods_consumer, "Добавлен новый продукт/изменены остатки существющего продукта", "Async, Kafka")
    
    Rel(api_client, catalog_controller, "Просматривает каталог, карточку товара", "HTTP")
    Rel(api_client, catalog_controller, "Изменение цены, описания продукта", "HTTP")

    Rel_D(api_layer, application_layer, "Uses")
    
    Rel_D(commands, domain_layer, "Uses")

    Rel_R(catalog_aggregate, good_aggregate, "Uses")

    Rel_D(commands, infrastructure_layer, "Uses")
}

ContainerDb(db, "CatalogDb", "Postgres", "Хранит категории и товары в них")
Rel_D(infrastructure_layer, db, "Uses")
Rel_D(queries, db, "Uses")


```

## Code diagram
Диаграмма классов показывает общую структуру иерархии классов системы, их коопераций, атрибутов, методов, интерфейсов и взаимосвязей между ними.

```plantuml
package "Catalog Aggregate"  #DDDDDD {
  Class Catalog <Aggregate>
  {
    - Category[] Categories 
    
    + Catalog()
  }
  
  Class Category <Entity>
  {
    - uuid Id
    - string Title 
    - Good[] Goods

    + Category(string title)
    + AddGood(uuid goodId)
    + RemoveGood(uuid goodId)
  }  
}

package "SharedKernel" #DDDDDD {
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
    - string imageUrl
    - Weight Weight    
    - int Quantity
    - int Price
    + Buy(int quantity)
    + Upload(int quantity)
    + Good(Guid id, string title, string description, Weight weight, Category category
  }

  Catalog *- Category
  Good *-- Weight
  Category *-- Good
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
  * category_name : string
  * pile_good_id : string
  * pile_quantity : string
}

entity Goods {
  * id : uuid <<PK>>
  * title : string
  * description : description
  * category_name : string
  * weight_gram : int
}

Warehouses ||-  Places
Places }o--|| Goods
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
  usecase (UC-4 Добавление/изменение остатков продукта) as UC4

  url of UC1 is [[use-cases/uc-1.md]]
  url of UC2 is [[use-cases/uc-2.md]]
  url of UC3 is [[use-cases/uc-3.md]]
  url of UC4 is [[use-cases/uc-4.md]]
}

client --> UC1
client --> UC2

UC1 <-- manager
UC2 <-- manager
UC3 <-- manager

UC4<-- warehouse
```
## Use cases
- [UC-1](use-cases/uc-1.md) Просмотр каталога продуктов.
- [UC-2](use-cases/uc-2.md) Посмотр карточки продукта.
- [UC-3](use-cases/uc-3.md) Изменение цены, описания продукта.
- [UC-4](use-cases/uc-4.md) Добавление/изменение остатков продукта.

