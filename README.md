[[_TOC_]]

## О проекте
Это учебный проект.
Целью проекта является демонстрация применения микросервискной архитектуры на практике.
Проект содержит полный путь - от выявляения Bounded Context и до реализации микросервисов.

## Проблема
От CEO нам была поставлена цель - спроектировать интернет магазин с использованием микросервисной архитектуры.
Для этого нам был предоставлен набор User Stories с описанием бизнеса компании, а также макеты интерфейса для лучшего понимания задачи.

Наша задача - провести декомпозицию системы на микросервисы и реализовать.

## Решение

### Event Storming
Для выявления границ микросервисов мы приняли решение использовать практику Event Storming
![Карта контекстов](img/es.jpg)

### Landscape diagram
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

### Context diagram
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

### Container diagram
```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

skinparam wrapWidth 200
skinparam maxMessageSize 200

LAYOUT_TOP_DOWN()
LAYOUT_WITH_LEGEND()

Person(customer, Покупатель, "Совершает покупки")
Person(manager, Менеджер, "Управляет интернет магазином")
System_Boundary(microservices, "Microservices") {
  Container(ordering, "Ordering", ".Net, Docker", "Управление процессом оформления заказа")
  Rel(customer, ordering, "Делает заказ", "HTTPS")  

  Container(warehouse, "Warehouse", ".Net, Docker", "Управление складом")
  Rel(ordering, warehouse, "Cоздан новый заказ", "Async, Kafka")
  Rel(manager, warehouse, "Поставки, изменение остатков", "HTTPS")

  Container(catalog, "Catalog", ".Net, Docker", "Управление каталогом витрины")
  Rel(warehouse, catalog, "Товары, остатки", "Async, Kafka")
  Rel(customer, catalog, "Витрина, карточка товара", "HTTPS")
  Rel_U(manager, catalog, "Изменение цен и описания", "HTTPS")

  Container(payment, "Payment", ".Net, Docker", "Управление процессом оплаты")
  Rel_R(ordering, payment, "Оплата заказа", "Sync, gRPC")

  Container(delivery, "Delivery", ".Net, Docker", "Управление процессом доставки заказа")
  Rel(ordering, delivery, "Cоздан новый заказ", "Async, Kafka")
  Rel_U(manager, delivery, "Получить статус доставки", "HTTPS")
  Lay_L(microservices,manager)
}
```

### Use case diagram

```plantuml
left to right direction
skinparam packageStyle rectangle

actor Покупатель as client
actor Менеджер as manager
actor Курьер as courier

rectangle System {
  ' auth
  usecase (Аутентифицироваться) as UC1
  usecase (Выйти) as UC2
  ' catalog
  usecase (Просмотр каталога продуктов) as UC3
  usecase (Посмотр карточки продукта) as UC4
  usecase (Изменение цены, описания продукта) as UC5
  ' delivery
  usecase (Получение статуса доставки) as UC6
  usecase (Изменение статуса заказа на 'Доставлен') as UC7
  usecase (Назначение заказа) as UC8
  
  ' ordering
  usecase (Добавление товара в корзину) as UC9
  usecase (Удаление товара из корзины) as UC10
  usecase (Просмотр списка товаров в корзине) as UC11
  usecase (Добавление адреса доставки) as UC12
  usecase (Оплата заказа) as UC13

  ' warehouse
  usecase (Принятие поставки) as UC14
}

client --> UC1
UC1 <-- manager
UC1 <--courier

client --> UC2
UC2 <-- manager
UC2 <--courier

client --> UC3
UC3 <-- manager

client --> UC4
UC4 <-- manager

UC5 <-- manager

client --> UC6
UC6 <-- manager

UC7 <--courier
UC8 --> courier

client --> UC9
client --> UC10
client --> UC11
client --> UC12
client --> UC13

UC14 <-- manager
```

## Системы
### Микросервисы
| Bounded Context                           | Команда       | Сервис                            | Репозиторий                                                              | Что делает                                |
| -----------                               | -----------   | -----------                       | -----------                                                              | ----------                                |
| Управление продуктовым каталогом          | Alpha         | [catalog](services/catalog)       |[Gitlab](https://gitlab.com/microarch-ru/minimarket-csharp/catalog)       | Каталог, Товары                           |
| Управление процессом оформления заказа    | Beta          | [ordering](services/ordering)      |[Gitlab](https://gitlab.com/microarch-ru/minimarket-csharp/ordering)      | Корзина, Оформление, процессинг заказа    |
| Управление процессом сборки и доставки    | Gamma         | [delivery](services/delivery)     |[Gitlab](https://gitlab.com/microarch-ru/minimarket-csharp/delivery)      | Курьеры, Процесс досставки                |
| Управление складом                        | Delta         | [warehouse](services/warehouse)   |[Gitlab](https://gitlab.com/microarch-ru/minimarket-csharp/warehouse)     | Корзина, Оформление, процессинг           |
| Управление процессом оплаты               | Omega         | [payment](services/payment)       |[Gitlab](https://gitlab.com/microarch-ru/minimarket-csharp/payment)       | Списание денег, бонусная программа        |


### Платформенные сервисы и системы
| Назначение                                | Команда       | Сервис                            | Репозиторий                                                              | Что делает                                |
| -----------                               | -----------   | -----------                       | ----------                                                               | ----------                                |
| Безопасность, контроль доступа            | Platform      | [auth](services/auth)             |[Gitlab](https://gitlab.com/microarch-ru/minimarket-csharp/auth)          | Регистрация, аутентификация, авторизация  |


### Фронтенд
| Назначение                                | Команда             | Приложение                            | Репозиторий                                                                           | Что делает                            |
| -----------                               | -----------         | -----------                           | -----------                                                                           | ----------                            |
| Витрина интернет магазина                 | Alpha, Beta         | [shop](front-end/shop)                |[Gitlab](https://gitlab.com/microarch-ru/minimarket-csharp/front-end/shop)             | Каталог, карточка товара, корзина     |
| Панель управления интернет магазином      | Alpha, Gamma, Delta | [backoffice](front-end/backoffice)    |[Gitlab](https://gitlab.com/microarch-ru/minimarket-csharp/front-end/backoffice)      | Управление каталогом, заказами       |
| Приложение курьера                        | Gamma               | [courier](front-end/courier)          |[Gitlab](https://gitlab.com/microarch-ru/minimarket-csharp/front-end/courier)          | Принятие заказов                      |