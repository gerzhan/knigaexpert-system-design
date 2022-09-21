```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml
' Components
!define actors https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/actors
!define frontends https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/frontends  
!define services https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/services

!include frontends/shop/web_app.puml
!include frontends/shop/gateway.puml

!include frontends/backoffice/web_app.puml
!include frontends/backoffice/gateway.puml

' !include frontends/courier/courier_mobile_app.puml
' !include frontends/courier/gateway.puml
```