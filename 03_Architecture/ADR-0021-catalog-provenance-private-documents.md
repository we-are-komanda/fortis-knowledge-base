# ADR-0021: Источники карточек и приватные документы

Дата: 2026-09-20. Решение FRC-09 в рамках разрешённого release run.

## Источник данных

DataProvenance хранит источник/дату/quality и серверные дату внесения, actor, revision. Подтверждение требует описания/URL/документа и даты; это ответственность пользователя, не сертификация Fortis. Старые данные без источника остаются неизвестными. Известные встроенные seed по id/legacyItemId сохраняют demo при импорте; явно предоставленный клиентский источник не делает старые demo подтверждёнными. Отдельная новая карточка позволяет заменить demo данными заказчика.

Типизированные карточки, exactminor цена/null и fieldProvenance сохраняются. Неизменённый редактор источника не заменяет собственную provenance карточки источником цены. Явный выбор документа с неизвестным quality становится estimated, но не confirmed. Запись карточки валидирует весь итоговый aggregate до persistence, чтобы смена категории не оставляла несовместимые сохранённые ТТХ.

Обновление библиотеки не изменяет frozen snapshot [[ADR-0018-immutable-project-revisions]]. Fetch/save каталога создаёт preview: применить к черновику или отменить. Preview сохраняет несколько правок, UI-selection не отменяет его, бизнес-изменение/сменаidentity делает старый preview неприменимым. Отсутствующая карточка не удаляет её frozen данные. Instance override сохраняется; для новой версии нужен явный save.

## Документы

Запрос multipart передаёт реальные bytes PDF/PNG/JPEG/TXT до10MiB. MIME/signature и bounded изображение/текст валидируются, storageKey/owner не принимаются от клиента. UUID storage key, приватный каталог0700/файлы0600 вне webroot. Metadata-only legacy upload закрыт; неизвестные старые внешние URL не выдаются и автоматически не мигрируют.

Quarantine metadata+audit фиксируются одной транзакцией до scanner. Только успешная проверка переводит ready отдельной транзакцией с повторной проверкой membership/parent. Недоступный scanner даёт503, подозрительный файл422/rejected; ни один не публикуется. Реализация scanner — явно настроенный абсолютный executable ClamAV с фиксированными аргументами и timeout. SourceUrl не скачивается сервером.

GET bytes каждый раз проверяет parent asset/enterprise, ready и SHA256/size. Ответ attachment/nosniff/private,no-store. Постоянных публичных URL нет. Новый файл с тем же именем имеет другой immutableID/revision/checksum; source snapshot содержит все три. Delete деактивирует активную ссылку и атомарно пишет audit; история не начинает указывать на новый файл. Недоступность старого документа показывается честно. Удаление приватных bytes/retention — отдельная политика поставки, не фоновая очистка неизвестных legacy данных.

## Поставка

APP_DOCUMENTS_ENABLED по умолчанию false, APP_DOCUMENTS_ROOT_DIR абсолютный приватный persistentvolume, APP_DOCUMENTS_SCANNER_EXECUTABLE абсолютный scanner. Без настроенного pipeline upload failclosed. Локальные проверки используют реальные PG/privatefiles и явно тестовый scanner; это не доказательство антивирусной проверки production. FRC14 требует реальные clean/infected/unavailable проверки, volume/backup/retention, TLS/права и ревокацию старых внешних URL у storageowner. HideCost фильтрация финансовых полей/коммерческихссылок — server projection FRC10.

Доказательства: parent docs/releases/2026-09-20/frc09/.
