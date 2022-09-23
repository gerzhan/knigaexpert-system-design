# UC-5 Оплата заказа

```plantuml
actor "Покупатель" as customer
participant "shop" as shop_app
participant "shop BFF" as shop_bff
participant "basket" as basket
participant "payment" as payment

alt Успешный случай
customer -> shop_app: Оплачивает заказ
shop_app -> shop_bff: Оплачивает заказ
note over shop_bff: [POST] /api/v1/carts/current/checkout

shop_bff -> basket: Проксирует запрос
note over basket: [POST] /api/v1/carts/current/checkout

basket -> payment: Списание денежных средств со счета клиента
note over payment: [POST] /api/v1/payment
basket <--  payment: 200 OK
note over basket: Успешное списание

shop_app <--  basket: 200 OK
note over shop_app: Сообщение "Заказ оформлен"
customer <--  shop_app: Заказ
end

alt Ошибка оплаты
basket <--  payment: 409 "Payment problem"
note over basket: Ошибка списания

shop_app <--  basket: 409 "Checkout problem"
note over shop_app: Сообщение "Ошибка при оплате"
customer <--  shop_app: Экран с ошибкой
end
```