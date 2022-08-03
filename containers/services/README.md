```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

' Catalog
!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/services/catalog/ext.puml
!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/services/catalog/normal.puml
!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/services/catalog/db.puml

' Warehouse
!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/services/warehouse/ext.puml
!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/services/warehouse/normal.puml
!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/services/warehouse/db.puml
```