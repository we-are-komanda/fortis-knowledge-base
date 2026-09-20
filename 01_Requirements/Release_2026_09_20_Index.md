# Релиз Fortis 2026-09-20 — FRC-00/01/02

Последующее задание разрешает автономно выполнять оставшиеся FRC после свежей проверки FRC-02. Текущий объём и authority: [[Release_2026_09_20_Autonomous_Run]]. Ограничения отдельных завершённых заданий ниже сохранены как история.

Статус: исходный baseline согласован; FRC-00 реализуется рабочими изменениями,
runtime-приёмка заблокирована. Коммиты, push и deployment требуют отдельной команды.

## Источник задания

Пользователь подтвердил parent `docs/fortis-release-specs/`. Его реальные ID —
FRC-00…FRC-14. Ранее названные FR-00…FR-15 не являются отдельным доступным пакетом.
Текущая задача — только `specs/FRC-00_release_baseline.md`.

- [Manifest](../../docs/releases/2026-09-20/release-manifest.json)
- [Матрица приёмки](../../docs/releases/2026-09-20/acceptance-matrix.md)
- [Доказательства и ограничения](../../docs/releases/2026-09-20/work-log.md)
- [Предлагаемые контракты](../../docs/fortis-release-specs/00_SHARED_CONTRACTS.md)

Ссылки на parent работают в общем workspace; отдельный clone knowledge-base
требует соседнего parent checkout для чтения этих файлов.

## Согласованный исходный комплект

| Репозиторий | SHA |
|---|---|
| parent | `dbc2d054798ec72fa7b607a64cae18319cfac3c3` |
| frontend | `7ae48358d10df796d6c5bc95e6b16791c9688ad9` |
| backend | `706665641894c966d2b0b83554554bc3d88b8f94` |
| knowledge-base | `eca5ee3be1acbc543fa64706e654d6d631a2815c` |

Это исходные версии, а не SHA ещё не закоммиченных артефактов FRC-00.
Parent gitlinks совпадают с комплектом. Рабочий frontend пользователя остаётся
на `9feeb88b273bad5fbaf8ffc52f9eedc6c52a54ae`; для baseline используется отдельный checkout.
Сохраняются границы [[03_Architecture/ADR-0003-return-to-submodules|ADR-0003]]
и модель [[03_Architecture/ADR-0005-defense-project-placement-authority|ADR-0005]].

## Принятые технические решения FRC-00

- Источник состава проверки — manifest с полными SHA; branch указан справочно.
- Backend canonical URL — `git@github.com:Ramilko37/fortis-back.git`, совпадает
  с текущим origin; чтением remote develop подтверждён SHA `70666564…`.
- Metadata checkout дополнена knowledge-base SHA и dirty flags. Она не доказывает
  состав запущенных образов. Никакие новые production endpoints не добавлены.
- Неизвестные deployment ID, image digest, applied migrations, фактические версии
  инструментов и окружение остаются `null`. BASE-02/05 блокируют приёмку.
- Новая матрица различает implemented, verified и accepted; Done в Linear не равен проверке.

## Открытые решения (не приняты этим изменением)

1. Ручное серверное сохранение из пакета против debounce autosave в
   [[03_Architecture/ADR-0010-frontend-backend-auth-project-boundary|ADR-0010]].
2. Запрет mock fallback в workspace против общего fallback из
   [[03_Architecture/ADR-0008-asset-library-api-frontend-boundary|ADR-0008]].
3. Автосоздание тестового backend-проекта в ADR-0010 против чистого первого запуска
   в [[Продуктовый_план_Fortis]] и [[05_Design/Fortis_initial_state_and_first_echelon_2026-07-30]].
4. Неизменяемые ревизии, minor units RUB, обязательная version, новый error envelope,
   роли/viewCost, закрытый onboarding, hard budget gate, ограничения отчёта,
   отдельный demo runtime и outbox остаются предложениями других FRC.
5. Нужно выбрать целевое окружение для сравнения manifest с runtime. Попытка
   read-only доступа к документированной dev VM завершилась `Permission denied (publickey)`.

FRC-00 фиксирует ограничение пакета: подготовка/сохранение/сравнение/документирование
2D-конфигураций и бюджета. Это не полное выполнение ТЗ. Spatial evaluation,
реальная эффективность, ретроспектива, сценарное моделирование, P2/P3 и realtime
collaboration остаются вне этой итерации. Их исходные спецификации сохраняются.

## Перед последующей фиксацией

Проверить рабочие изменения, закоммитить knowledge-base отдельно по команде
пользователя, затем согласованно обновить parent pointer. Не менять задним числом
исходные SHA уже выполненных проверок; записать новый candidate комплект и повторить
затронутые проверки. До этого нет ни release acceptance, ни разрешения на deployment.


## Последующее согласование FRC-01 — 2026-09-20

Пользователь отдельно разрешил только FRC-01 с уточнениями. Принятые решения находятся в [[03_Architecture/ADR-0016-runtime-backend-boundary|ADR-0016]], [reconciliation](../../docs/releases/2026-09-20/frc01/reconciliation.md) и [отчёте](../../docs/releases/2026-09-20/frc01/implementation-report.md). Они закрывают вопросы выше о fallback, demo, bootstrap и текущем ручном save в пределах FRC-01. Прочие контракты FRC остаются открытыми. Исходный FRC-00 manifest не переписан: Implemented / Acceptance blocked by runtime evidence.


## Последующее согласование FRC-02 — 2026-09-20

Реализована enterprise isolation по [[03_Architecture/ADR-0017-enterprise-access-boundary|ADR-0017]], поверх текущего рабочего checkout. Утверждённая модель: membership → project/financial/private asset → document parent. Foreign/missing 404, no valid identity 401, operator-only 403, инфраструктурная ошибка 500. Обычный member не управляет memberships, enterprise create/delete и public catalog. Full RBAC, RLS, audit log и session revocation отложены.

Frontend recovery и active state привязаны к actual identity; account switch/logout/401 сбрасывают память и старые async responses. Auth cookie в production Secure/HttpOnly/Lax/Path=/; redirect next внутренний.

- [Implementation report](../../docs/releases/2026-09-20/frc02/implementation-report.md)
- [Authorization matrix](../../docs/releases/2026-09-20/frc02/authorization-matrix.md)
- [Test evidence](../../docs/releases/2026-09-20/frc02/test-evidence.md)
- [Changed files](../../docs/releases/2026-09-20/frc02/changed-files.md)
- [Orphan inventory](../../docs/releases/2026-09-20/frc02/orphan-inventory.md)

Проверены локальные memory/HTTP, PostgreSQL и browser A→B/revocation сценарии; окончательный verification status находится в отчёте. Production inventory, storage ACL, deployment identity/TLS и реальная runtime acceptance не подтверждены. FRC-00 остаётся Implemented / Acceptance blocked by runtime evidence; FRC-01 — Implemented / code-level acceptance passed / runtime acceptance blocked. FRC-03 не начат. Commit/push/deployment/production migrations запрещены текущим заданием.


## Разрешение на фиксацию и push — 2026-09-20

После локальной реализации и отчёта пользователь отдельно поручил закоммитить и отправить всю выполненную работу FRC-00/01/02 в `develop` каждого репозитория. Это разрешение заменяет прежний запрет commit/push; deployment и production migrations не запрошены. Runtime acceptance остаётся заблокированной.

Frontend реализован поверх `9feeb88`, тогда как `origin/develop` указывал на `7ae4835` (14 последующих коммитов, 110 отличающихся файлов). Пользователь явно выбрал сохранить текущую проверенную версию файлов в итоговом `develop`, включив историю remote через merge без force push. Содержимое этих 14 коммитов не переносится в проверенное дерево; их история сохраняется. Остальные репозитории допускают fast-forward. Генерируемые browser outputs и локальный `.deploy-info` не входят в исходники.
