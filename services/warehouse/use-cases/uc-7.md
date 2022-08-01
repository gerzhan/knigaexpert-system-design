# UC-7 Получить схему склада
Диаграмма последовательности показывает жизненный цикл объекта на единой временной оси, взаимодействие с другими обьектами и актерами информационной системы в рамках одного варианта использования.

```plantuml
actor Менеджер as manager
participant "warehouse" as warehouse


manager -> warehouse: получить схему склада
note over warehouse: [GET] api/v1/warehouse

manager <-- warehouse: схема склада
note over warehouse
200 Ok
end note
```