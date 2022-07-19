# UC-1 Аутентифицироваться

```plantuml
actor "Покупатель" as customer
participant "shop" as shop_app
participant "auth" as auth

customer -> shop_app: Вводит логин и пароль
shop_app -> auth: Передает логин и пароль
note over auth: [POST] /api/v1/auth
shop_app <--  auth: Jwt токен
note over shop_app: Jwt токен
customer <--  shop_app: Аутентифицирован
```