# ADR-0023: Авторитетный бюджет серверного проекта

Дата: 2026-09-21. Утверждённый пользователем контракт FRC-07; реализация и acceptance выполняются отдельно.

## Решение

Бюджет принадлежит серверному проекту и входит в immutable revision [[ADR-0018-immutable-project-revisions]]. Режим unlimited имеет budgetLimitMinor=null; limited — неотрицательную десятичную строку копеек, включая0. Расчёт использует [[ADR-0020-canonical-versioned-cost]], без float и без превращения неизвестной цены в ноль.

Обычные new/import создаются с unlimited независимо от финансовых полей присланного projectJSON. Создание копии может явно передать top-level budgetSource:{projectId,projectVersion}; сервер повторяет доступ к исходному проекту и читает бюджет его конкретной immutable revision. Ссылка входит в idempotency payload. У копии после создания независимый бюджет. Копия обязана помещаться в скопированный limited budget; существующее превышение исходника не переносит исключение на новый проект.

Каждый commit конфигурации читает budget, рассчитывает весь кандидат и пишет current/revision/audit в одной существующей транзакции с project row lock и expected version. Проверка UI/preflight не резервирует остаток. Новое превышение отклоняется422 budget_exceeded; неизвестная цена кандидата при limited —422 price_unknown. Version conflict остаётся409.

Явное снижение лимита ниже текущего расхода разрешено: проект продолжает существовать. Если текущая сохранённая стоимость уже выше лимита, допустим кандидат с полной известной стоимостью не выше текущей; увеличение запрещено. Preflight описывает разрешённое превышение reasonCode=non_increasing_overage, не within_budget. Изменение одного бюджета не меняет состав и разрешено также при неполной смете; projection indeterminate. Неизвестная текущая стоимость не доказывает право на новое превышение, поэтому известный кандидат должен помещаться в limit. Неизвестный кандидат limited всегда блокируется.

PUT бюджета увеличивает общую project version и сохраняет budget snapshot/audit. Предыдущие версии читаются неизменно. Ответ содержит canonical committed project и бюджет той же версии. FE сохраняет более новые локальные изменения отдельно и при неоднозначном ответе проверяет серверное состояние до ручного повтора. Загрузка исторического превышения не блокируется редакторским gate.

## HTTP-контракт

- GET /api/v1/projects/budget?projectId=P&projectVersion=V → BudgetProjection:{identity,config,spentMinor,remainingMinor,status}. Release UI передаёт конкретную версию; compatibility id alias не отменяет проверки противоречащих aliases.
- PUT того же projectId, body:{version,budgetMode,budgetLimitMinor} → canonical ProjectResponse плюс budget:BudgetProjection новой версии.
- POST /api/v1/projects/budget/check?projectId=P, body:{version,projectJson} → {basedOn,status,reasonCode,projectedSpentMinor,projectedRemainingMinor}; valid allowed/blocked200, malformed400, stale409. Запрет финальной записи422.
- Создание: budgetSource только отдельным явным полем; import не принимает его как финансовую authority.

remainingMinor может быть отрицательной строкой; spentMinor/totalMinor=null означает неполную стоимость. Все доступы проверяются до финансовых деталей, чужой/отозванный ресурс остаётся404. Никаких новых ролей, публичных обходов, soft override, отдельных reservation tables или финансов из CSS-only hiding.

## Изменённые предложения старой спецификации

Этот утверждённый контракт заменяет старый запрет понижения лимита (BUD-07) и старое предложение доверять budget в create/import JSON. Требования atomicity, zero limit, exact money, concurrency и независимость исторических snapshots сохраняются. Evidence и актуальный checkpoint: parent docs/releases/2026-09-20/frc07/.
