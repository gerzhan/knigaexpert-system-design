```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml
' Components
!define actors https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/actors
!define gateways https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/gateways  
!define services https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/services

!include gateways/shop/shop.puml
!include gateways/shop/gateway.puml

!include gateways/backoffice/backoffice.puml
!include gateways/backoffice/gateway.puml

!include gateways/courier/courier_app.puml
!include gateways/courier/gateway.puml
```