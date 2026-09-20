# ADR-0018: неизменяемые ревизии проекта и атомарный журнал

- Дата: 2026-09-20
- Статус: принятое решение в рамках авторизованного FRC-04; фактическая приёмка кода — в release evidence.
- Связи: [[ADR-0016-runtime-backend-boundary]], [[ADR-0017-enterprise-access-boundary]], [[../01_Requirements/Release_2026_09_20_Autonomous_Run]]

## Решение

DefenseProject сохраняет полное валидированное JSON-содержимое, включая пользовательские слои/геометрию, compoundProfile, instance overrides, typed-card данные и безопасные extension-поля. Бизнес-права и ownership определяются существующей типизированной моделью и колонками БД. Неизвестные поля не дают прав и не становятся денежными входами без явного типизированного контракта.

Серверные projectId, enterpriseId, version, updatedAt и имя варианта берутся из сохранённых колонок. `projectName` остаётся заголовком бизнес-документа; существующее `name` — отдельным именем варианта. Единицы не конвертируются повторно в DTO. Top-level requestId/generatedAt не входят в snapshot.

Один transaction проверяет текущую membership и expected version, сохраняет current state, бюджет при его изменении, version+1, immutable snapshot `{project,budget,inputDataVersions}` и минимальное audit event. Конкурирующий stale writer получает409; ошибка любой записи откатывает всё. История уникальна по projectId/version и не изменяется UPDATE. SHA-256 вычисляется по каноническому сохранённому snapshot. Миграция создаёт только известную current version, не восстанавливает выдуманные предыдущие.

GET get/export принимает projectVersion. Конкретная отсутствующая версия —404 revision_not_found, без подмены current. Права повторно проверяются по текущему ресурсу на historical read и replay. Удаление закрывает доступ и удаляет snapshots по FK; долговременная retention/backup политика требует владельца в FRC-14.

Create/import принимает Idempotency-Key (релизный клиент передаёт его), scope actor+operation, TTL24h. Повтор того же payload возвращает исходный результат, другой payload —409. Удалённый/недоступный ресурс остаётся недоступен через повтор. Все browser UPDATE требуют положительную expected version. Budget API использует тот же commit boundary при интеграции FRC-07.

## Audit

`audit_events` содержит только eventId, actorId, enterpriseId, entityType/entityId, action, previous/new version, occurredAt, requestId. Payload, карты, суммы, email и токены отсутствуют. Журнал append-only для приложения и защищён DB-trigger от обычных UPDATE/DELETE; это не защита от администратора БД. Исторические actor/entity ссылки не имеют live FK, способных стереть журнал. Удаление проекта и его audit — одна транзакция. Idempotent replay не создаёт второй event.

HTTP middleware задаёт серверный UUID requestId в context и ответе. Внутренние вызовы генерируют UUID. Это новый scoped project/budget/document журнал; deferred membership/session audit и RBAC из ADR-0017 не возвращаются автоматически.

## Поставка

Миграции:1789900000_project_revisions и1789900001_audit_events. Проверка/откат — только на изолированном PostgreSQL. Production migration не выполнялась. Реальный release требует утверждённого миграционного окна, резервной копии и runtime evidence; наличие кода не означает production acceptance.
