# ADR-0017: enterprise membership как граница доступа Fortis

Дата: 2026-09-20. Статус: решение согласовано заданием FRC-02; реализация в текущем checkout. Production acceptance требует отдельного runtime evidence.

## Контекст

JWT authentication и `user_enterprises` уже существовали, но `EnterpriseService.CheckUserAccess` не вызывался большинством resource use cases. Валидный JWT позволял обращаться к чужим project, budget, report, asset и document по точному ID. Общий browser recovery key и проверка только наличия cookie создавали отдельную утечку при смене аккаунта.

## Решение

**Enterprise membership — tenant authorization boundary.** Member имеет доступ к project CRUD/import/export, budget/cost/report/compare и private assets/documents своего предприятия. Полный RBAC не вводится.

| Ситуация | HTTP semantics |
|---|---|
| JWT отсутствует, invalid/expired, либо identity не установлена | 401 |
| Resource отсутствует или enterprise недоступен | 404, одинаковый public body |
| Запрещённая операция в известном контексте / закрытая operator operation | 403 |
| Ошибка access checker / хранилища | 500, без внутренних подробностей |

`AuthRequired` проверяет JWT и существование пользователя. Membership в JWT не помещается и не кэшируется как постоянное право. Каждый resource request заново проверяет storage. JWT прежнего пользователя остаётся криптографически валидным после отзыва membership, но resource operations возвращают 404.

## Application boundary

Controller извлекает actor ID и отображает ошибки; application service принимает actor ID явно, загружает ресурс/родителя, получает enterprise из сохранённых данных и использует существующий `CheckUserAccess`. Domain не зависит от transport/auth framework. Репозитории выполняют scoped lists; pagination/totalItems вычисляются после membership filtering.

- Project принадлежит enterprise. Incoming enterprise проверяется при create/import.
- Enterprise ownership immutable через обычный update. И отдельное поле DTO, и embedded project JSON с другим enterprise отклоняются validation error (400) до изменения данных; версия проверяется после authorization.
- Project transfer — отдельная будущая операция.
- Budget/cost/check/report наследуют доступ project. Compare авторизует оба operands до вычислений.
- Дополнительные catalog specifications в report читаются через asset service. Недоступный asset не обогащает ответ; storage failure не скрывается как отсутствующий asset.

## Assets и документы

- Public asset (`is_public=true`), в том числе с NULL enterprise: read/list для authenticated users.
- Создание public asset, превращение private в public, update/delete public asset и создание/удаление его документов закрыты ordinary JWT (403).
- Private asset требует membership сохранённого enterprise.
- Все document operations наследуют видимость или mutation policy parent asset. URL/storage reference возвращается только после parent authorization; document OwnerID устанавливается из actor.
- Direct/presigned URLs, уже выданные ранее, и фактическая object storage ACL требуют отдельной runtime проверки. FRC-02 не создаёт storage service и не отзывает уже выданные ссылки.

## NULL/orphan ownership

NULL-owned project и private asset закрыты ordinary users (404). Document с отсутствующим parent или orphan private parent также недоступен. Нельзя автоматически назначать ownership текущему пользователю или первому enterprise.

Существующая schema сохраняется. До production необходим read-only [orphan inventory](../../docs/releases/2026-09-20/frc02/orphan-inventory.md), решение владельца данных о cleanup и проверка applied migrations. Новая constraint migration без inventory не принимается.

## Frontend identity boundary

- В workspace `/workspace`, `/prototype`, `/calculator` проверяют actual `/auth/me` через существующий backend boundary. Наличие cookie и `FORTIS_AUTH_ENABLED` не заменяют identity verification. Отдельный demo runtime из [[ADR-0016-runtime-backend-boundary]] сохраняется.
- Client boundary до показа protected pages проверяет `/api/auth/me`; повторяет проверку при возвращении вкладки в фокус/видимость. Cookie HttpOnly; production: Secure, SameSite=Lax, Path=/; HTTP development без Secure.
- Recovery key: `fortis-defense-project:user:<encoded-user-id>`. Workspace без identity не читает и не пишет recovery. Общий старый ключ не мигрируется неизвестному владельцу. Demo имеет отдельный ключ.
- Смена identity/logout/401 очищают project, variants, enterprise/asset lists, selections, studio/configuration stores. Generation guard отвергает старые async responses. React subtree меняет key, очищая локальные calculator/report/budget/compare/form states; используемого QueryClient в этом flow нет.
- Logout удаляет cookie существующим BFF route и выполняет полную навигацию для сброса router cache. Снимок A может сохраниться под ключом A, но B его не восстанавливает.
- После 404/403 project остаётся только unverified local recovery с видимой ошибкой; save блокируется до успешного явного открытия. 401 сбрасывает identity и ведёт на login. Ошибка не включает demo и не превращается в saved.
- Текущий checkout использует ручной save. Retry/autosave subsystem не добавляется.
- Login `next` допускает только внутренний путь с неизменным origin; external, `//`, backslash и control characters отклоняются.

## Operator-only operations

Membership administration, enterprise create/delete и public catalog mutation не предоставляются plain membership. Metadata update enterprise остаётся разрешённым member.

В checkout нет отдельного подтверждённого operator role/API. Поэтому обычные mutation routes закрыты, а repository provisioning остаётся внутренним механизмом для доверенного оператора/test fixture. Перед onboarding реальных пользователей нужен согласованный operator provisioning process; новый admin UI и роль не создаются в FRC-02. Цена решения: пользователь с нулём memberships видит пустой workspace до назначения оператором.

## Deferred

Полный RBAC (Admin/Engineer/Viewer/Auditor/Integrator), project/document ACL, PostgreSQL RLS, Redis sessions, refresh tokens/session revocation, audit log, transfer и методики financial/spatial validation остаются за пределами FRC-02. Logout не отзывает ранее скопированный JWT; membership revocation действует на resource requests немедленно.

## Доказательства и пределы

[Route matrix](../../docs/releases/2026-09-20/frc02/authorization-matrix.md), [test evidence](../../docs/releases/2026-09-20/frc02/test-evidence.md), [implementation report](../../docs/releases/2026-09-20/frc02/implementation-report.md).

Локальные PostgreSQL и browser E2E используют synthetic fixtures и реальные repository/service/HTTP paths. Они не устанавливают SHA/digests работающего deployment, production inventory, TLS edge или storage ACL. FRC-00 и runtime-статус FRC-01 не повышаются. Исторические ADR не переписываются.
