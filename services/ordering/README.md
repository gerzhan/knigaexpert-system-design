[[_TOC_]]

## Ordering
Позволяет формировать корзину.
Когда покупатель завершает покупки, то он может добавить адрес доставки,
оплатить и тем самым создать заказ.

## Container diagram
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

## Use case diagram
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

