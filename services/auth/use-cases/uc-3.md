# UC-3 Получить cert.pub

### Shop BFF
```plantuml
participant "shop BFF" as shop_bff
participant "auth" as auth

shop_bff -> auth: Получить cert.pub
note over auth: [GET] /api/v1/jwks
shop_bff <--  auth: cert.pub
note over shop_bff: cert.pub
```

### Backoffice BFF
```plantuml
participant "backoffice BFF" as backoffice_bff
participant "auth" as auth

backoffice_bff -> auth: Получить cert.pub
note over auth: [GET] /api/v1/jwks
backoffice_bff <--  auth: cert.pub
note over backoffice_bff: cert.pub
```

### Сourier BFF
```plantuml
participant "courier BFF" as courier_bff
participant "auth" as auth

courier_bff -> auth: Получить cert.pub
note over auth: [GET] /api/v1/jwks
courier_bff <--  auth: cert.pub
note over courier_bff: cert.pub
```