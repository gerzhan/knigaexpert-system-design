```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml
' Components
!define actors https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/actors
!define frontends https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/frontends  
!define services https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/services

!include services/payment/ext.puml
!include services/payment/normal.puml
!include services/payment/db.puml
```