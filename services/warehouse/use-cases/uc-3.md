# UC-3 Получить все товары
Диаграмма последовательности показывает жизненный цикл объекта на единой временной оси, взаимодействие с другими обьектами и актерами информационной системы в рамках одного варианта использования.

```plantuml
actor Менеджер as manager
participant "warehouse" as warehouse

manager -> warehouse: получить все товары
note over warehouse: [GET] api/v1/goods

manager <-- warehouse: товары доступные на складе
note over warehouse
[{
  "id": "uuid",
  "title": "string",
  "description": "string",
},{
  "id": "uuid",
  "title": "string",
  "description": "string",
}]
end note
```