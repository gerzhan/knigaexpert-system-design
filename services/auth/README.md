[[_TOC_]]

# Auth
Платформенный сервис. 
Отвечает за аутентификацию пользователей.

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
!include actors/courier.puml

Container_Ext(warehouse, "Warehouse", "Docker", "Управление складом")

System_Boundary(boundary, "Auth") {
' Shop
!include frontends/shop/web_app.puml
!include frontends/shop/gateway.puml
Rel(customer, shop_app, "Использует", "HTTPS")

' Backoffice
!include frontends/backoffice/web_app.puml
!include frontends/backoffice/gateway.puml
Rel(manager, backoffice_web_app, "Использует", "HTTPS")

' Сourier App
!include frontends/courier/courier_app.puml
!include frontends/courier/gateway.puml
Rel(courier, courier_app, "Изменить статус доставки", "HTTPS")

' Services
!include services/auth/normal.puml
!include services/auth/db.puml
Rel(shop_app, auth, "Аутентифициуется, получает JWT токен", "HTTPS")
Rel(backoffice_web_app, auth, "Аутентифициуется, получает JWT токен", "HTTPS")
Rel_R(shop_bff, auth, "Получает cert.pub раз в 30 мин", "HTTPS")
Rel_L(backoffice_bff, auth, "Получает cert.pub раз в 30 мин", "HTTPS")
Rel_U(courier_bff, auth, "Получает cert.pub раз в 30 мин", "HTTPS")
Rel(courier_app, auth, "Аутентифициуется, получает JWT токен", "HTTPS")
}
```

## Component diagram
Диаграмма компонентов показывает, из каких «компонентов» состояит контейнер, что представляет собой каждый из этих компонентов, его обязанности, технологии и детали реализации.

Не применимо, так как мы используем коробочное решение Keycloak.

## Use case diagram
Диаграмма вариантов использования показывает, какой функционал разрабатываемой программной системы доступен каждой группе пользователей.

```plantuml
left to right direction
skinparam packageStyle rectangle

actor Покупатель as client
actor Менеджер as manager
actor Курьер as courier

actor "Shop BFF" as shop_bff << Система >>
actor "Backoffice BFF" as backoffice_bff << Система >>
actor "Courier BFF" as courier_bff << Система >>

rectangle Auth {
  usecase (UC-1 Аутентифицироваться) as UC1
  usecase (UC-2 Выйти) as UC2
  usecase (UC-3 Получить cert.pub) as UC3
  
  url of UC1 is [[use-cases/uc-1.md]]
  url of UC2 is [[use-cases/uc-2.md]]
  url of UC3 is [[use-cases/uc-3.md]]  
}

client --> UC1
client -->UC2 

courier --> UC1
courier --> UC2 

UC2 <-- manager
UC1 <-- manager

UC3 <-- shop_bff
UC3 <-- backoffice_bff
UC3 <-- courier_bff

```

**Use cases**
- [UC-1](use-cases/uc-1.md) Аутентифицироваться.
- [UC-2](use-cases/uc-2.md) Выйти.
- [UC-3](use-cases/uc-3.md) Получить cert.pub.

