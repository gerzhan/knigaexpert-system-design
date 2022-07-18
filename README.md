## Оглавление
[[_TOC_]]

## О проекте
Мы проектируем интернет магазин.

## Проблема

## Event Storming
![Карта контекстов](img/es.jpg)

## Решение

## Микросервисы
| BoundedContext                            | Команда       | Сервис                                                                      | Что делает                                |
| -----------                               | -----------   | -----------                                                                 | ----------                                |
| Управление продуктовым каталогом          | Alpha         |[catalog](https://gitlab.com/microarch-ru/minimarket-csharp/catalog)         | Каталог, Товары                           |
| Управление процессом оформлением заказа   | Beta          |[ordering](https://gitlab.com/microarch-ru/minimarket-csharp/ordering)       | Корзина, Оформление, процессинг заказа    |
| Управление процессом сборки и доставки    | Gamma         |[delivery](https://gitlab.com/microarch-ru/minimarket-csharp/delivery)       | Курьеры, Процесс досставки                |
| Управление складом                        | Delta         |[warehouse](https://gitlab.com/microarch-ru/minimarket-csharp/warehouse)     | Корзина, Оформление, процессинг           |
| Управление процессом оплаты               | Omega         |[payment](https://gitlab.com/microarch-ru/minimarket-csharp/payment)         | Списание денег, бонусная программа        |


## Платформенные сервисы и системы
| Назначение                            | Команда       | Приложение                                                           | Что делает                                  |
| -----------                           | -----------   | -----------                                                          | ----------                                |
| Безопасность, контроль доступа        | Core          |[auth](https://gitlab.com/microarch-ru/minimarket-csharp/auth)        | Регистрация, аутентификация, авторизация  |


## Фронтенд
| Назначение                            | Команда       | Приложение                                                                            | Что делает               |
| -----------                           | -----------   | -----------                                                                           | ----------               |
| Витрина интернет магазина             | Core          |[shop](https://gitlab.com/microarch-ru/minimarket-csharp/front-end/shop)       | Отображает статус заказа |
| Панель управления интернет магазином  | Core          |[backoffice](https://gitlab.com/microarch-ru/minimarket-csharp/front-end/backoffice)   | Отображает статус заказа |

## Context diagram
```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Context.puml

skinparam wrapWidth 200
skinparam maxMessageSize 200

LAYOUT_TOP_DOWN()
LAYOUT_WITH_LEGEND()

Person(customer, Покупатель, "Совершает покупки")

Enterprise_Boundary(shop_boundary, "Shop") {
Person(manager, Менеджер, "Управляет интернет магазином")

' Shop
System(shop, "Shop", "Интернет магазин")
Rel(customer, shop, "Делает покупки")
Rel_U(manager, shop, "Управляет")

' Services
System_Ext(auth, "Auth", "Сервер аутентификации")
Rel_L(shop, auth, "Использует")
Rel_R(customer, auth, "Авторизуется")
}
```

## Container diagram
```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

skinparam wrapWidth 200
skinparam maxMessageSize 200

LAYOUT_TOP_DOWN()
LAYOUT_WITH_LEGEND()

Person(customer, Покупатель, "Совершает покупки")
Person(manager, Менеджер, "Управляет интернет магазином")
System_Boundary(shop, "Shop") {
' Shop
Container(shop_app, "Shop", "Web, React", "Витрина интернет магазина")
Container_Ext(shop_bff, "Shop BFF", "Api Gateway, Ocelot", "Маршрутизация трафика c web приложения shop, аутентификацяи, авторизация")
Rel(shop_app, shop_bff, "Использует", "HTTPS")
Rel(customer, shop_app, "Делает покупки", "HTTPS")

' Backoffice
Container(backoffice_app, "Backoffice", "Web, React", "Панель управления интернет магазином")  
Container_Ext(backoffice_bff, "Backoffice BFF", "Api Gateway, Ocelot", "Маршрутизация трафика, аутентификацяи, авторизация")
Rel(backoffice_app, backoffice_bff, "Использует", "HTTPS")
Rel(manager, backoffice_app, "Управляет интернет магазином", "HTTPS")

Container(microservices, "Microservices", ".Net, Docker", "Группа микросервисов")
Rel(shop_bff, microservices, "Использует", "HTTPS")
Rel(backoffice_bff, microservices, "Использует", "HTTPS")

' Services
Container_Ext(auth, "Auth", "Keycloak, Java", "Сервер аутентификации")
Rel_R(shop_app, auth, "Аутентифициуется", "HTTPS")
Rel_L(backoffice_app, auth, "Аутентифициуется", "HTTPS")
}


```

## Microservices Container diagram
```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

skinparam wrapWidth 200
skinparam maxMessageSize 200

LAYOUT_TOP_DOWN()
LAYOUT_WITH_LEGEND()

Person(customer, Покупатель, "Совершает покупки")
Person(manager, Менеджер, "Управляет интернет магазином")
System_Boundary(microservices, "Microservices") {
  Container(ordering, "Ordering", ".Net, Docker", "Управление процессом оформлением заказа")
  Rel(customer, ordering, "Делает заказ", "HTTPS")  

  Container(warehouse, "Warehouse", ".Net, Docker", "Управление складом")
  Rel(ordering, warehouse, "Информация о заказах", "Async, Kafka")
  Rel(manager, warehouse, "Поставки, изменение остатков", "HTTPS")

  Container(catalog, "Catalog", ".Net, Docker", "Управление каталогом витрины")
  Rel(warehouse, catalog, "Товары, остатки", "Async, Kafka")
  Rel(customer, catalog, "Витрина, карточка товара", "HTTPS")
  Rel_U(manager, catalog, "Изменение цен и описания", "HTTPS")

  Container(payment, "Payment", ".Net, Docker", "Управление процессом оплаты")
  Rel_R(ordering, payment, "Оплата", "Sync, gRPC")

  Container(delivery, "Delivery", ".Net, Docker", "Управление процессом доставки заказа")
  Rel(ordering, delivery, "Заказы в доставку", "Async, Kafka")
  Rel_U(manager, delivery, "Статус доставки", "HTTPS")
}
```

## Варианты использования
```plantuml
left to right direction
skinparam packageStyle rectangle

actor Покупатель
actor Менеджер
actor Кладовщик
actor Курьер

rectangle Minimarket {
  usecase (UC-1 Просмотр каталога продуктов) as UC1
  usecase (UC-2 Посмотр деталей продукта) as UC2
  usecase (UC-3 Добавление продукта в корзину) as UC3
  usecase (UC-4 Удаление продукта из корзины) as UC4
  usecase (UC-5 Оформление заказа) as UC5
  
  usecase (UC-6 Получение статуса заказа)  as UC6
  usecase (UC-7 Отмена заказа)  as UC7

  usecase (UC-8 Сборка заказа)  as UC8
  usecase (UC-9 Доставка заказа)  as UC9
  usecase (UC-10 Приемка товаров на склад)  as UC10

  Покупатель --> UC1
  Покупатель --> UC2
  Покупатель --> UC3
  Покупатель --> UC4
  Покупатель --> UC5
  Покупатель --> UC6

  UC6 <-- Менеджер
  UC7 <-- Менеджер

  UC8<-- Кладовщик
  UC10<-- Кладовщик
  
  UC9<-- Курьер
}
```

- [UC-1](/use-cases/1-viewing-product-catalog.md) Просмотреть каталог продуктов.
- [UC-2](use-cases/2-viewing-product-details.md) Посмотреть детали продукта.
- [UC-3](use-cases/3-adding-product-to-the-cart.md) Добавить продукт в корзину.
- [UC-4](use-cases/4-remove-product-from-shopping-cart.md) Удалить продукт из корзины.
- [UC-5](use-cases/5-make-order.md) Оформить заказ.
- [UC-6](use-cases/6-get-order-status.md) Посмотреть статус заказа.
- [UC-7](use-cases/7-order-cancellation.md) Отменить заказ.
- [UC-8](use-cases/8-order-assembly.md) Собрать заказ.
- [UC-9](use-cases/9-order-delivery.md) Доставить заказ.
- [UC-10](use-cases/10-acceptance-goods-to-warehouse.md) Принять товары на склад.

### Стэйкхолдеры

- **Покупатель** - хочет получить свой заказ.
- **Менеджер** - управляет интернет магазином.
- **Курьер** - собирает и доставляет заказ.
- **Кладовщик** - принимает товары на склад и выдает их курьерам.

