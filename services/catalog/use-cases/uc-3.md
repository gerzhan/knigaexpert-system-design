# UC-3 Изменение цены, описания продукта

```plantuml
actor Менеджер as manager
participant "backoffice" as backoffice_web_app
participant "backoffice BFF" as backoffice_bff
participant "catalog" as catalog

alt Успешный случай
manager -> backoffice_web_app: Изменяет цену и описание продкта
backoffice_web_app -> backoffice_bff: Изменяет цену и описание продкта
note over backoffice_bff: [PUT] /api/v1/products/{id}

backoffice_bff -> catalog: Проксирует запрос
note over catalog: [PUT] /api/v1/products/{id}

backoffice_web_app <--  catalog: 200 OK
note over backoffice_web_app: Сообщение "Продукт успешно отредактирован"
manager <--  backoffice_web_app: Экран с сообщением
end
alt Продукт не существует
backoffice_web_app <--  catalog: 404 Not Found
note over backoffice_web_app: Ошибка "Продукт не существует"
manager <--  backoffice_web_app: Экран с ошибкой
end
```