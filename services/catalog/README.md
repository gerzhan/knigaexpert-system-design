[[_TOC_]]

## О сервисе
Позволяет отображать каталог товаров.
Так же отвечает за отображение карточки товара.

### Container diagram
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


' Services
Container_Ext(auth, "Auth", "Keycloak, Java", "Сервер аутентификации")
Rel_R(shop_app, auth, "Аутентифициуется", "HTTPS")
Rel_L(backoffice_app, auth, "Аутентифициуется", "HTTPS")


Container(catalog, "Catalog", ".Net, Docker", "Управление каталогом витрины")
Rel_L(warehouse, catalog, "Базовая информация о продуктакх, остатки", "Async, Kafka")
Rel(shop_bff, catalog, "Просматривает каталог, карточку товара", "HTTPS")
Rel(backoffice_bff, catalog, "Изменение цены, описания продукта", "HTTPS")
}

```

## Use case diagram
```plantuml
left to right direction
skinparam packageStyle rectangle

actor Покупатель as cleint
actor Менеджер as manager
actor Курьер as courier

rectangle Catalog {
  usecase (UC-1 Просмотр каталога продуктов) as UC1
  usecase (UC-2 Посмотр карточки продукта) as UC2
  usecase (UC-3 Изменение цены, описания продукта) as UC3

  url of UC1 is [[uc-1.md]]
  url of UC2 is [[uc-2.md]]
  url of UC3 is [[uc-3.md]]
}

cleint --> UC1
cleint --> UC2

UC1 <-- manager
UC2 <-- manager
UC3 <-- manager
  
UC1<-- courier  
UC2<-- courier
```
## Use cases

- [UC-1](uc-1.md) Просмотреть каталог продуктов.
- [UC-2](uc-2.md) Посмотреть детали продукта.
- [UC-3](uc-3.md) Добавить продукт в корзину.

