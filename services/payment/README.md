[[_TOC_]]

## Payment
Платформенный сервис. 
Отвечает за проведение платежей.

## Container diagram
```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

skinparam wrapWidth 200
skinparam maxMessageSize 200

LAYOUT_TOP_DOWN()
LAYOUT_WITH_LEGEND()

Person(customer, Покупатель, "Совершает покупки")

' Shop
Container_Ext(shop_app, "Shop", "Web, React", "Витрина интернет магазина")
Container_Ext(shop_bff, "Shop BFF", "Api Gateway, Ocelot", "Маршрутизация трафика c web приложения shop, аутентификацяи, авторизация")
Rel(shop_app, shop_bff, "Формирует корзину, делает заказ", "HTTPS")
Rel(customer, shop_app, "Формирует корзину, делает заказ", "HTTPS")

Container_Ext(ordering, "Ordering", ".Net, Docker", "Управление процессом оформления заказа")
Rel(shop_bff, ordering, "Формирует корзину, делает заказ", "HTTPS")

System_Boundary(boundary, "Payment") {
  Container(payment, "Payment", ".Net, Docker", "Управление процессом оплаты")
  Rel_R(ordering, payment, "Оплата заказа", "Sync, gRPC")
}
```

## Use case diagram
```plantuml
left to right direction
skinparam packageStyle rectangle

actor Ordering as ordering << Система >>

rectangle Payment {
  usecase (UC-1 Списание денежных средств с карты) as UC1
  
  url of UC1 is [[use-cases/uc-1.md]]  
}

ordering --> UC1
```
## Use cases
- [UC-1](use-cases/uc-1.md) Списание денежных средств с карты.
