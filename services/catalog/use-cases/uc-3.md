# UC-3 Изменение цены, описания продукта

```plantuml
actor Менеджер as manager
participant "backoffice" as backoffice_app
participant "backoffice BFF" as backoffice_bff
participant "catalog" as catalog

alt Успешный случай
manager -> backoffice_app: Изменяет цену и описание продкта
backoffice_app -> backoffice_bff: Изменяет цену и описание продкта
note over backoffice_bff: [PUT] /api/v1/products/{id}

backoffice_bff -> catalog: Проксирует запрос
note over catalog: [PUT] /api/v1/products/{id}

backoffice_app <--  catalog: 200 OK
note over backoffice_app: Сообщение "Продукт успешно отредактирован"
manager <--  backoffice_app: Экран с сообщением
end
alt Продукт не существует
backoffice_app <--  catalog: 404 Not Found
note over backoffice_app: Ошибка "Продукт не существует"
manager <--  backoffice_app: Экран с ошибкой
end
```