```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

' Catalog
!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/services/catalog.puml
!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/services/catalog_db.puml

' Warehouse
!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/services/warehouse.puml
!include https://gitlab.com/microarch-ru/microservices/dotnet/system-design/-/raw/main/containers/services/warehouse_db.puml
```