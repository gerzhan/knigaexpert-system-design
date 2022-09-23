[[_TOC_]]

## Ordering
Позволяет формировать корзину.
Когда покупатель завершает покупки, то он может добавить адрес доставки и оформить корзну.

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
' LAYOUT_TOP_DOWN()
LAYOUT_WITH_LEGEND()

!include actors/customer.puml
System_Boundary(boundary, "Ordering") {
  !include frontends/shop/web_app.puml
  !include frontends/shop/gateway.puml
  !include services/ordering/normal.puml
  !include services/ordering/db.puml

  Rel(customer, shop_web_app, "Формирует корзину, делает заказ", "HTTPS")
  Rel(shop_bff, ordering, "Формирует корзину, делает заказ", "HTTPS")
}

!include services/warehouse/ext.puml
!include services/delivery/ext.puml

Rel(ordering, warehouse_ext, "Cоздан новый заказ", "Async, Kafka")
Rel(ordering, delivery_ext, "Cоздан новый заказ", "Async, Kafka")
```

## Component diagram
Диаграмма компонентов показывает, из каких «компонентов» состояит контейнер, что представляет собой каждый из этих компонентов, его обязанности, технологии и детали реализации.

```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Component.puml
LAYOUT_WITH_LEGEND()

Container_Ext(api_client, "API Client", " HTTP REST", "Внешний потребитель API")

Container_Ext(message_bus, "Message Bus", "Kafka", "Transport for business events")

Container_Boundary(ordering_service, "Ordering") {
    Container_Boundary(api_layer, "API Layer") {
    Component(ordering_http_handler, "OrderingHttpController", "Web API Controller", "Обрабатывает HTTP запросы, извлекает параметры из них")
    
    Component(goods_consumer, "GoodsConsumer", "Kafka Consumer", "Обрабатывает Message из Kafka")
    }

    Container_Boundary(application_layer, "Application Layer") {
      Container_Boundary(commands, "Commands") {
        Component(add_item_command, "AddItemCommand", "", "UC-1 Добавление товара в корзину")
        Component(remove_item_command, "RemoveItemCommand", "", "UC-2 Удаление товара из корзины")
        Component(add_address_command, "AddAddressCommand", "", "UC-4 Добавление адреса доставки")
        Component(checkout_command, "CheckoutCommand", "", "UC-5 Оплата заказа")
      }

      Container_Boundary(queries, "Queries") {
        Component(get_basket_query, "GetBasketQuery", "", "UC-3 Просмотр списка товаров в корзине")
      }
    }

    Container_Boundary(domain_layer, "Domain Layer") {
      Component(catalog_aggregate, "Basket", "Aggregate", "Корзина товаров")
    }

    Container_Boundary(infrastructure_layer, "Infrastructure Layer") {
      Component(basket_aggregate_repository, "BasketRepository", "", "Репозиторий для сохранения/восстановления аггрегата Basket")
    }

    Rel(message_bus, goods_consumer, "Добавлен новый продукт/изменены остатки существющего продукта", "Async, Kafka")
    
    Rel(api_client, ordering_http_handler, "Добавляет, удаляет товары из корзины, оформляет ее", "HTTP")

    Rel_D(api_layer, application_layer, "Uses")
    
    Rel_D(commands, domain_layer, "Uses")

    Rel_D(commands, infrastructure_layer, "Uses")
}
ContainerDb(db, "BasketDb", "Postgres", "Хранит корзины и элементы в них")
Rel_D(infrastructure_layer, db, "Uses")
Rel_L(queries, db, "Uses")
```

## Code diagram
Диаграмма классов показывает общую структуру иерархии классов системы, их коопераций, атрибутов, методов, интерфейсов и взаимосвязей между ними.

```plantuml
package "Basket Aggregate"  #DDDDDD {
  Class Basket <Aggregate>
  {
    - uuid Id
    - Item[] Items
    - Address Address
    - Status Status

    + Basket()
    + AddItem(Item item)
    + RemoveItem(uuid itemId)
    + AddAddress(Address address)
    + Checkout()
    + Done()    
  }
  
  Class Item <Entity>
  {
    - uuid Id
    - string Title
    - decimal Price    
    + Item(uuid id, string title, string description, decimal price)    
  }  

  Class Address <Value Object>{
    - string City
    - string Street
    - string Building
    + Address(string сity, string street, string building)
  }

  Class Status <Value Object>{
    - int Id
    - string Name
    + New
    + Filled
    + CheckoutStarted
    + Done

    - Status(int Id, string Name)
  }  
}

Basket *-- Item
Basket *- Address
Basket *-- Status
```
## ER diagram
Диаграмма отношений сущностей это визуальное представление базы данных, которое показывает, как связаны элементы внутри. Диаграмма ER состоит из двух типов объектов — сущностей и отношений.

```plantuml
entity Baskets {
  * id : uuid <<PK>>
  * status_id <<FK>>
  * address_city : string
  * address_street : string
  * address_building : string
}

entity Items {
  * id : uuid <<PK>>
  * basket_id <<FK>>
  * title string
  * price : decimal
}

entity Statuses {
  * id : integer <<PK>>
  * name : string
}

Baskets ||-  Items
Baskets }o--|| Statuses
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

