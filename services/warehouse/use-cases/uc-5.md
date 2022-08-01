# UC-5 Списать товар
Диаграмма последовательности показывает жизненный цикл объекта на единой временной оси, взаимодействие с другими обьектами и актерами информационной системы в рамках одного варианта использования.

```plantuml
actor Менеджер as manager
participant "warehouse" as warehouse


manager -> warehouse: cписать товар
note over warehouse: [POST] api/v1/goods/{id}/write-off

alt Успешный ответ
manager <-- warehouse: Успешный ответ
note over warehouse
200 Ok
end note

else Товар не найден
manager <-- warehouse: Ошибка
note over warehouse
404 Not Found
end note
end
```