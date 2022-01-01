## Оглавление
- [О проекте](#о-проекте)
- [О продукте](#о-продукте)
- [Проектирование](#о-продукте)
- [Реализация](#о-продукте)

## О проекте
Мы проектируем интернет магазин.

##### Цель проекта
Демонстрация подходов и технологий:
 - DDD
 - Event Storming
 - Microservices architecture
 - DevOps practices

##### Event Storming
![Карта контекстов](img/es.jpg)

##### Контексты и сервисы
| BoundedContext                            | Команда       | Сервис                                                                      | Что делает                                |
| -----------                               | -----------   | -----------                                                                 | ----------                                |
| Управление процессом оформлением заказа   | Alpha         |[ordering](https://gitlab.com/microarch-ru/minimarket-csharp/catalog)        | Корзина, Оформление, процессинг           |
| Управление процессом оплаты               | Beta          |[payment](https://gitlab.com/microarch-ru/minimarket-csharp/catalog)         | Списание денег, бонусная программа        |
| Управление процессом доставки заказа      | Gamma         |[delivery](https://gitlab.com/microarch-ru/minimarket-csharp/catalog)        | Процессинг Сборки, доставки               |
| Безопасность, контроль доступа            | Delta         |[socket-hub](https://gitlab.com/microarch-ru/minimarket-csharp/catalog)      | Регистрация, аутентификация, авторизация  |

##### Фронтенд
| Назначение                            | Команда       | Приложение                                                        | Что делает                |
| -----------                           | -----------   | -----------                                                       | ----------                |
| Витрина интернет магазина             | core          |[showcase](https://gitlab.com/microarch-ru/minimarket-csharp/catalog)  | Отображает статус заказа. |
| Панель управления интернет магазином  | core          |[backoffice](https://gitlab.com/microarch-ru/minimarket-csharp/catalog)  | Отображает статус заказа. |


## Архитектура и технологии
- https://mehdihadeli.github.io/awesome-go-education/ddd/
- https://mehdihadeli.github.io/awesome-go-education/cqrs/


## Системная архитектура
```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/v2.0.1/C4_Component.puml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/v2.0.1/C4_Container.puml

skinparam wrapWidth 200
skinparam maxMessageSize 200

LAYOUT_TOP_DOWN()
LAYOUT_WITH_LEGEND()

' Actors
Person(customer, Пользователь, "Хочет купить продукт")
Person(manager, Менеджер, "Управляет интернет магазином")

' Frontends
Container(showcase_app, "showcase", "React", "Витрина интернет магазина")
Container(backoffice_app, "backoffice", "React", "Панель управления интернет магазином")  

' Api Gateways
Container(showcase_bff, "showcase BFF", "Virtual Ingress Istio, Api Gateway", "Маршрутизация трафика c web приложения Showcase, аутентификацяи, авторизация")
Container(backoffice_bff, "Backoffice BFF", "Virtual Ingress Istio, Api Gateway", "Маршрутизация трафика, аутентификацяи, авторизация")

' Services
System_Boundary(auth_service, "auth") {
  Container(auth, "keycloak", "Java", "Сервис управления аутентификацией")
  ContainerDb(auth_db, "Keycloak Database", "Postgress", "Пользователи")
  Rel(auth, auth_db, "read / write", "TCP")
}

System_Boundary(ordering_service, "ordering") {
  Container(ordering, "ordering", "Go lang", "Управление процессом оформлением заказа")
  ContainerDb(ordering_db, "ordering Database", "Postgress", "Продукты, заказы")
  Rel(ordering, ordering_db, "read / write", "TCP")
}

System_Boundary(payment_service, "payment") {
  Container(payment, "payment", "Go lang", "Управление процессом оплаты")
  ContainerDb(payment_db, "payment Database", "Postgress", "Карты, бонусы, финансовые операции")
  Rel(payment, payment_db, "read / write", "TCP")
}

System_Boundary(delivery_service, "delivery") {
  Container(delivery, "delivery", "Go lang", "Управление процессом доставки заказа")
  ContainerDb(delivery_db, "Keycloak Database", "Postgress", "Курьеры, маршруты")
  Rel(delivery, delivery_db, "read / write", "TCP")
}

System_Boundary(socket_hub_service, "socket-hub") {
  Container(socket_hub, "socket-hub", "Go lang", "Hub для WebSocket, слушает NATS и отправляет данные frontend приложениям")
}


' Front -> Auth
Rel_L(showcase_app, auth, "Использует", "HTTPS")
Rel_R(backoffice_app, auth, "Использует", "HTTPS")

' Actor -> Front
Rel(customer, showcase_app, "Использует", "HTTPS")

' Front -> Api Gateway
Rel(showcase_app, showcase_bff, "Использует", "HTTPS")

' Api Gateway -> Services
Rel(showcase_bff, ordering_service, "Создать / получить заказ", "HTTP")
Rel(showcase_bff, payment_service, "Списать деньги", "HTTP")
Rel_U(socket_hub_service,showcase_bff , "Event stream", "HTTPS")



' Actor -> Front
Rel(manager, backoffice_app, "Использует", "HTTPS")

' Front -> Api Gateway
Rel(backoffice_app, backoffice_bff, "Использует", "HTTPS")

' Api Gateway -> Services
Rel(backoffice_bff, delivery_service, "Статус соборки / доставки", "HTTPS")


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
