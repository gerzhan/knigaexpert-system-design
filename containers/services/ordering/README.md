```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml
' Components
!define actors https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/actors
!define gateways https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/gateways  
!define services https://gitlab.com/microarch-ru/microservices/system-design/-/raw/main/containers/services

!include services/ordering/ext.puml
!include services/ordering/normal.puml
!include services/ordering/db.puml
```