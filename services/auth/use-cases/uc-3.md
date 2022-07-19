# UC-3 Получить cert.pub

```plantuml
participant "shop BFF" as shop_bff
participant "auth" as auth

shop_bff -> auth: Получить cert.pub
note over auth: [GET] /api/v1/jwks
shop_bff <--  auth: cert.pub
note over shop_bff: cert.pub
```