# UC-4 Получить информацию о товаре
Диаграмма последовательности показывает жизненный цикл объекта на единой временной оси, взаимодействие с другими обьектами и актерами информационной системы в рамках одного варианта использования.

```plantuml
actor Менеджер as manager
participant "warehouse" as warehouse

manager -> warehouse: получить информацию о товаре
note over warehouse: [GET] api/v1/goods/{id}

manager <-- warehouse: информация о товаре
note over warehouse
{
  "id": "uuid",
  "title": "string",
  "description": "string",
}
end note
```