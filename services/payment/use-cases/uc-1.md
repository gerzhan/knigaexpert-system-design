# UC-1 Списание денежных средств с карты

```plantuml
actor "Покупатель" as customer
participant "shop" as shop_app
participant "shop BFF" as shop_bff
participant "ordering" as ordering
participant "payment" as payment

alt Успешный случай
customer -> shop_app: Оплачивает заказ
shop_app -> shop_bff: Оплачивает заказ
note over shop_bff: [POST] /api/v1/carts/current/checkout

shop_bff -> ordering: Проксирует запрос
note over ordering: [POST] /api/v1/carts/current/checkout

ordering -> payment: Списание денежных средств со счета клиента
note over payment: [POST] /api/v1/payment
ordering <--  payment: 200 OK
note over ordering: Успешное списание

shop_app <--  ordering: 200 OK
note over shop_app: Сообщение "Заказ оформлен"
customer <--  shop_app: Заказ
end

alt Ошибка оплаты
ordering <--  payment: 409 "Payment problem"
note over ordering: Ошибка списания

shop_app <--  ordering: 409 "Checkout problem"
note over shop_app: Сообщение "Ошибка при оплате"
customer <--  shop_app: Экран с ошибкой
end