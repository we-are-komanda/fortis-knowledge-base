# ADR-0022 — Trusted enterprise provisioning и первый workspace project

- Дата: 2026-09-21
- Статус: принято пользователем; implementation acceptance фиксируется отдельно в FRC-03 evidence
- Основание: [[ADR-0017-enterprise-access-boundary]], [[ADR-0018-immutable-project-revisions]], [[../01_Requirements/Release_2026_09_20_Autonomous_Run]]

## Контекст

Новый зарегистрированный пользователь без membership корректно получает пустой workspace. Самостоятельного назначения прав нет; отсутствие штатного operator process блокировало первый рабочий проект. Существующие enterprise/project API уже проверяют membership и поддерживают серверную pagination.

## Решение

Первый релиз использует trusted local operator process с существующей runtime/DB configuration и OS/database privileges. Режим `enterprise-provision` принимает existing user UUID, enterprise UUID, name/address и обязательные явные latitude/longitude. Координаты валидируются как GIS, включая диапазоны и конечность; `(0,0)` допустим, отсутствие координат — ошибка. Процесс не создаёт пользователя и не назначает права обычному JWT через HTTP.

Создание enterprise и membership — одна транзакция в application/infrastructure границе существующего enterprise модуля. Ошибка membership откатывает enterprise. Стабильный явно переданный enterprise UUID позволяет безопасно повторить тот же operator command после потерянного результата: совпадают данные и тот же пользователь; изменённые данные/пользователь или unrelated existing enterprise не перезаписываются и не усыновляются. Composite membership key не допускает duplicate rows. Нового idempotency хранилища не требуется.

Результат stdout содержит только enterpriseId/userId; ошибки и ORM logs не должны раскрывать secrets или полный payload. Команда документируется как trusted operator workflow. SQL не является штатным способом выдачи доступа. Реальная пользовательская local DB не используется для release fixtures.

Workspace перечитывает доступные предприятия и выбранную серверную страницу проектов через Refresh. Selection scoped по identity и проверяется по свежему списку доступа. Проекты запрашиваются с enterpriseId/limit/offset; totalItems относится к тому же фильтру. Пустой список не подменяется чужими/demo проектами. Loading/empty/error/forbidden различаются.

Создание конфигурации использует existing frozen save attempt и backend idempotency. Имя после trim содержит 1…120 Unicode code points. До подтверждённого server projectId/version/enterprise прежний current context сохраняется. Ошибка оставляет ввод и допускает явный replay того же запроса; успех атомарно устанавливает подтверждённый контекст. Missing enterprise не выводится из baseObject/названия/координат. FRC-06 guards сохраняются.

## Последствия и границы

Нет публичного enterprise self-service/admin endpoint, полного RBAC, нового organization model или автоматического геокодирования. Production operator ownership и operational controls остаются FRC-14 gate. FRC-03 доказывается в disposable PostgreSQL и настоящем Next→Go browser flow, а не одним HTTP200.
