# UC-2 Выйти

### Покупатель
```plantuml
actor "Покупатель" as customer
participant "shop" as shop_app

customer -> shop_app: Выйти
shop_app -> shop_app: Удалить JWT токен из хранилища приложения
note over shop_app: Jwt токен удален
customer <--  shop_app: Вышел
```

### Менеджер
```plantuml
actor Менеджер as manager
participant "shop" as shop_app

customer -> shop_app: Выйти
shop_app -> shop_app: Удалить JWT токен из хранилища приложения
note over shop_app: Jwt токен удален
customer <--  shop_app: Вышел
```

### Курьер
```plantuml
actor "Покупатель" as customer
participant "shop" as shop_app

customer -> shop_app: Выйти
shop_app -> shop_app: Удалить JWT токен из хранилища приложения
note over shop_app: Jwt токен удален
customer <--  shop_app: Вышел
```