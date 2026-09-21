# Autonomous release-hardening — 2026-09-20

## Возобновление autonomous run — 2026-09-21

Новая инструкция пользователя отменяет прежнюю остановку и задаёт цепочку **FRC-06 → FRC-03 → vertical gate → FRC-07 → FRC-08 → FRC-10 → FRC-13 → FRC-14**. Опубликованные checkpoint commits frontend `045db0d` и parent `6bf246e` остаются в истории. На входе все четыре `develop` clean, fetch origin выполнен, divergence 0/0. Локальные commits разрешены по принятому этапу, сначала owning child, затем parent pointer; push/deployment/production mutations не разрешены.

Уточнённые утверждённые контракты:

- FRC-03: trusted local operator CLI через существующие runtime/DB и DDD conventions. Existing user, enterprise name/address, обязательные явные valid GIS coordinates, atomic enterprise+membership, rollback и отсутствие duplicate membership. Нет публичного admin/self-service endpoint, RBAC или ручного SQL как штатного workflow. Реальную пользовательскую local DB/root/root2 не менять ради UI; использовать disposable fixtures.
- FRC-07: budget — server project authority. New/import unlimited; project JSON не источник budget. Copy только explicit trusted parameter из доступного server source, далее независимо. Limited updates сверх limit отклоняются HTTP422 `budget_exceeded`; если понижение budget создало overage, non-increasing/cost-reducing edits разрешены, cost-increasing запрещены. Exact minor money и unknown≠zero обязательны.
- FRC-08 сравнивает явно идентифицированные saved server revisions. FRC-10 один immutable server snapshot для всех частей отчёта; hideCost удаляет finance на backend. Dirty выбор Save/last-saved обязателен при включении отчёта.
- До FRC-13: stable `layer.id` в deck runtime identity, regression для одинаковых human codes; exact Next16.2.11 pin manifest+lock после подтверждения установленного lock/runtime и повторных проверок.
- FRC-14: отсутствие deployed artifact/TLS/migration/rollback/backup/storage/scanner/retention/monitoring/SMTP/owners evidence означает **CODE PASS / RUNTIME ACCEPTANCE BLOCKED**, а не production PASS.

FRC-06 evidence текущего продолжения: [implementation](../../docs/releases/2026-09-20/frc06/implementation-report.md), [test evidence](../../docs/releases/2026-09-20/frc06/test-evidence.md), [criteria](../../docs/releases/2026-09-20/frc06/acceptance-criteria.md), [review](../../docs/releases/2026-09-20/frc06/independent-review.md). Новые browser13/13 и focused48/48 проходят; реальный Next→Go→disposable PostgreSQL подтвердил save/recovery/conflict/copy reload. Final verification и итоговая stage acceptance фиксируются ниже/в checkpoint; до этого FRC-03 не начинается. SAVE-09 staged N/A при отключённой report capability, обязательное завершение в FRC-10; report не объявлен принятым.

## Authorization and current scope

The user authorized autonomous local implementation of the remaining package FRC-03…FRC-14 after verification of FRC-02, with no repeated approval for ordinary engineering decisions. This supersedes historical task-level instructions to stop after FRC-00/01/02. It does not authorize push, deployment, production migrations, destructive changes or invented runtime evidence.

FRC-02 entry gate was rechecked against current clean, origin-synchronized checkouts: frontend focused tests 31/31 plus 5/5; backend isolation, readiness and save/version regressions passed. Full frontend baseline remains 84/90 with six known failures. Production security acceptance remains blocked by its recorded evidence requirements.

Recovery/actual progress: [autonomous-run](../../docs/releases/2026-09-20/autonomous-run.md), [acceptance matrix](../../docs/releases/2026-09-20/acceptance-matrix.md). Code and evidence, not historical Done, decide acceptance.

## Contract authority

- [[03_Architecture/ADR-0016-runtime-backend-boundary|ADR-0016]] remains authoritative for one backend, transparent HTTP statuses, explicit workspace/demo, manual save and no silent fallback. Network boundary is 502; private container HTTP requires the accepted explicit deployment exception.
- [[03_Architecture/ADR-0017-enterprise-access-boundary|ADR-0017]] remains authoritative for enterprise membership, immutable ownership, public/private parent access, orphan fail-closed and identity isolation. Full RBAC/viewCost, membership/session audit and server session revocation remain deferred. Their presence in the older proposed package is not authority to reintroduce them silently.
- Thus proposed CMP-09 and viewer-specific parts of FRC-10 are not acceptance requirements for an unimplemented viewer role. Explicit user-selected `hideCost` still requires a safe server projection; prices cannot merely be hidden with CSS. Membership portions of CI-11 remain deferred; project/budget/document events may be introduced by their owning FRC.
- The user authorized the package's already specified revision, exact-money, idempotency and budget contracts. Record resulting cross-repository implementation decisions in new ADRs; retain old ADR history.
- Operator-only capabilities must not be assigned to ordinary members. A trusted local administrative process can use existing OS/DB privileges; publication/onboarding requires documented operator ownership. No new public admin role/API is inferred.

## Dependency order and production gates

Main chain: FRC-04 → FRC-05 → FRC-06 → FRC-03/FRC-07 → FRC-08/FRC-10 → FRC-13 → FRC-14. FRC-09 follows FRC-04; FRC-11/12 are independent after FRC-01. FRC-13/14 preparation is permitted early; acceptance is integrated last.

Runtime-only blockers do not establish code PASS and do not automatically stop independent local work. Storage/scanner pipeline, production TLS/artifact identity, real notification delivery, consent text, hosting/data-region approval, retention and support/data owners still need actual evidence. No consent wording, recipient, data policy or production success is invented.

[[Release_2026_09_20_Index]] retains the preceding work's historical observations.

## Остановка по запросу пользователя —2026-09-20

После просьбы закрыть ближайшую задачу и остановиться завершена кодовая приёмка FRC-09 (CODE PASS / RUNTIME BLOCKED). Кодовая приёмка04/05/09/11/12 есть;06 сохранена частично и не принята,03/07 только исследованы,08/10/13/14 не реализованы. Полная поставка не готова. Все subagents и созданные run тестовые серверы остановлены, пользовательскийdev3000 не затронут. Коммитов/push/deploy/productionmigrations не было.

Продолжение начинается с parent docs/releases/2026-09-20/autonomous-run.md и frc06/recovery-checkpoint.md; сохранять текущий diff. Для09 реальныеscanner/storage/TLS/retention остаются gates. Для05 дополнительныеmalformedinput fixes проверены; duplicatecustomcode maprendererconsole finding записан для последующейинтеграционнойпроверки. Новыйserver policy persistentdrafts пока частьнезавершённой06, неготовыйrecoveryUX.

## Локальный вход после остановки — 2026-09-20

По отдельному сообщению пользователя устранена `503 configuration_error` на `localhost:3000/login`: у frontend отсутствовала конфигурация backend. Создан игнорируемый Git файл `frontend/.env.development.local` с `BACKEND_URL=http://127.0.0.1:8090` и `FORTIS_LOCAL_DEVELOPMENT=true`; порядок настройки добавлен в frontend README. Существующий dev-сервер перечитал файл без перезапуска. Проверено: frontend `/api/health/ready` — 200; пустой POST `/api/auth/login` теперь возвращает backend 400 `email and password are required`, идентичный прямому ответу backend. Вход с реальными учётными данными не выполнялся. Код resolver и production policy не менялись; release run остаётся остановлен.

## Восстановление локального backend после 502 — 2026-09-20

Следующий запрос пользователя воспроизвёл `502 backend_unreachable`: процесс на 8090 уже отсутствовал (connection refused); причина завершения прежнего процесса не установлена. Найдена существующая база Fortis `rr` на 127.0.0.1:5432, версия схемы 10, одна учётная запись. Перед запуском выполнен pg_dump и проверен каталог pg_restore; приватная копия: `/private/tmp/fortis-local-backend-recovery-qpf_qe6c/before-start.dump`. Создан игнорируемый `backend/.env` (0600) для ENVIRONMENT=dev, webapp, существующей локальной базы и нового случайного JWT secret, без публикации секрета. Новая подпись требует повторного входа для старых сессий.

Текущий backend собран в `/private/tmp/fortis-local-backend-app` и оставлен работающим на 8090; лог: `/private/tmp/fortis-local-backend-recovery-qpf_qe6c/backend.log`. Автоматические локальные миграции довели схему до 1789916000, dirty=false; число пользователей осталось 1. Проверки через frontend: ready=200, пустой login=400, случайный несуществующий аккаунт=401, пустая register=400. Реальный пароль не использовался, тестовые аккаунты не создавались. Для последующих запусков — `make run-dev` из backend; обновлён README. Релизный run остаётся на паузе; production не затрагивался.

## Тупик после входа в workspace — 2026-09-20

В пользовательской вкладке `/workspace` подтверждены пустой список предприятий и отключённое создание без объяснения. Локальная база содержит два тестовых предприятия, три аккаунта (включая root/root2) и ноль назначений user_enterprises. Это корректное ограничение ADR-0017, но отсутствующий onboarding делает начальный пользовательский сценарий непроходимым. Запрошен выбор аккаунта для назначения одного конкретного тестового предприятия; до ответа права не изменены.

Минимальное исправление UI: явное объяснение отсутствия доступа и следующего шага, повторное чтение предприятий по «Обновить», перепроверка выбора по свежему списку, запрет создания при загрузке/ошибке, отсутствие списка проектов без доступного выбранного предприятия, доступное имя поля конфигурации. Проверено в текущей in-app вкладке: пустое состояние видно, после Refresh scoped SQL queries выросли с 18 до 20, console errors/warnings отсутствуют; scoped eslint, tsc --noEmit --incremental false и git diff --check прошли. Создание и открытие карты в этой сессии ещё не проверены: назначение доступа ожидает выбора аккаунта. FRC-03 остаётся непринятой, полный release run не возобновлён.

## Публикация checkpoint по запросу пользователя — 2026-09-20

После подготовки отчёта пользователь явно запросил коммит и push в develop всех репозиториев. Это разрешение на публикацию текущего промежуточного состояния, включая незавершённые FRC-03/06, а не приёмка релиза. Предпубликационная проверка: четыре origin/develop синхронизированы; backend go test ./... и frontend tsc --noEmit --incremental false прошли; FRC-06 повторно дал 4 PASS / 3 известных FAIL. Полный frontend suite и production evidence этим не заменяются. Локальные .env, JWT secret и backup базы исключены из Git. Предшествующие указания «коммитов/push не было» описывают состояние до этого запроса.


## FRC-06 принята локально — 2026-09-21

**PASS для staged FRC-06.** Все30 пользовательских критериев PASS, SAVE01–08/10 PASS; SAVE09 N/A только при закрытой report capability до обязательной FRC10. Focused48/48, Chromium13/13, real Next→Go→disposablePG PASS; все7 review findings закрыты,8 probes PASS. Types/lint/build PASS,454 source hashes совпали. Full frontend250=201PASS/6 exact historical FAIL/43opt-inSKIP; новые failures отсутствуют. Frontend `a56d128e715ed09387428ad3dd67434d18855dd5`. Дальше FRC03 и отдельный real vertical gate. Production/reports не приняты. Actualproof: parent frc06/test-evidence и acceptance-criteria.


## FRC-03 и отдельный vertical gate — 2026-09-21

**PASS локального этапа.** Trusted CLI→Next→Go→disposable PostgreSQL: первое созданиеv1, редактирование эшелона, Savev2, неизменный ID/полный canonical GET после reload/relogin,121 строк пагинации, пустое E2, изоляция B. Console/page errors0. Browser29/29; full277=212PASS/6exacthistoricalFAIL/59SKIP; types/lint/buildPASS. Operator realCLI9/PG23 и independent review пройдены. Финальное исправление auth/logout проверено root; независимые агенты после этого недоступны по лимиту аккаунта. Evidence: docs/releases/2026-09-20/frc03/test-evidence.md и real-vertical/result.json. Production не принята. Далее07→08→10→13→14; без push/deploy.
