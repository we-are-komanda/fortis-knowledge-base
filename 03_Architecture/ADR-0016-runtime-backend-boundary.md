# ADR-0016: единый backend boundary и явный runtime Fortis

Дата: 2026-09-20. Статус: Accepted по согласованию FRC-01; реализация локальная, runtime acceptance не выполнена.

## Контекст

Investigation использовал frontend `7ae4835`; реализация выполнена поверх текущего `9feeb88` после read-only reconciliation. Текущий checkout уже использует ручное сохранение и canonical assets. Возврат baseline и новый autosave не требуются.

## Решение

- `BACKEND_URL` — обязательный server-side источник backend origin для BFF и `/api/v1/*`. Resolver один. `FORTIS_API_BASE_URL` и `NEXT_PUBLIC_FORTIS_API_BASE_URL` — только проверяемые aliases: если заданы, обязаны совпадать после нормализации. Alias не заменяет обязательный `BACKEND_URL`.
- Нет public remote fallback. Localhost default разрешён только при `NODE_ENV=development` и `FORTIS_LOCAL_DEVELOPMENT=true`. Production localhost HTTP не разрешён.
- HTTPS обязателен для публичного backend. Private container HTTP, например `http://backend:8090`, разрешён при `FORTIS_BACKEND_PRIVATE_HTTP=true` и HTTPS на внешней границе. Resolver не доказывает сетевую изоляцию или наличие внешнего TLS; это ответственность deployment verification.
- `FORTIS_RUNTIME_MODE=workspace|demo`, default workspace. Решение принимает сервер. Query, localStorage, ошибки и пустые ответы не включают demo. Demo видим в UI и маркируется в ответах demo routes; canonical backend operations в demo отключены.
- Workspace использует `/api/v1/enterprises`, `/api/v1/assets`. Legacy facilities/catalog/layers/evaluate/recommend доступны только в demo; они не являются вторым production API.
- `forwardBackendRequest` передаёт исходный upstream status/body, включая JSON, HTML, empty и binary, query/method, необходимые безопасные headers и correlation id. JWT-cookie передаётся как Bearer и только необходимый cookie. Redirect наружу не следует за запросом с credentials.
- Только boundary failures получают generated error: config 503, network 502, deadline 504; есть requestId, retryable=false. Upstream 401/403/404/409/500 сохраняются без навязывания нового envelope.
- JSON adaptation отдельна: malformed successful JSON и неверная обязательная форма — typed protocol error, не empty state. 200+[] допустим для списка; пустой project envelope не превращается в 200 null.
- Ошибка asset refresh сохраняет проект; пустая библиотека остаётся пустой. Импорт/recovery не добавляет bundled assets и не заменяет пользовательские значения совпадающих asset IDs.
- Failed/cancelled/stale load сохраняет открытый проект и более свежие изменения. Повторный mount не загружает поверх уже открытого dirty проекта тот же backend ID.
- Recovery snapshot сохраняет domain source, но syncStatus становится `unverified`. Подтверждение загрузкой/сохранением отдельно от origin проекта. Общая модель revisions/recovery FRC-04/06 не вводится.
- В текущем checkout ручное сохранение. После 409/network/timeout/5xx изменения остаются локально с видимой ошибкой. Retry timer не вводится. Пользователь может повторить сохранение вручную. Будущий autosave требует отдельного решения.
- Backend liveness не зависит от DB; readiness использует DB PingContext с deadline 2 s. Frontend liveness не зависит от backend; workspace readiness проверяет конфигурацию и backend readiness. Compose frontend healthcheck использует liveness.

## Уточнение и замена прежних решений

Исторические ADR сохраняются целиком.

| Источник | Статус противоречащего положения |
|---|---|
| [[ADR-0008-asset-library-api-frontend-boundary]]: rewrite, default public VM, alias fallback | Superseded единым resolver и route forwarder; контракт км↔м сохранён |
| ADR-0008: мягкий local catalog fallback при outage | Superseded: в workspace запрещён, явный demo разрешён |
| [[ADR-0010-frontend-backend-auth-project-boundary]]: facilities/catalog backend-first с fallback, отдельный assets BFF | Superseded canonical enterprises/assets и demo gate |
| ADR-0010: автоматический тестовый bootstrap и debounce autosave | Clarified для current checkout: чистый workspace draft и ручной save; прежний механизм не восстанавливается |
| ADR-0010: local fallback для offline/empty | Superseded запретом mock fallback; recovery допустим только как unverified |
| ADR-0010: version, общий project context, backend authority | Сохранено; FRC-04/06/07 и financial methodology не меняются |

## Проверки и ограничения

См. [FRC-01 implementation report](../../docs/releases/2026-09-20/frc01/implementation-report.md).
HTTP stub и локальный Next production server подтверждают boundary, но не реальную DB/network/deployment связку. Начавшийся raw stream нельзя заменить на HTTP 504 после отправки headers: timeout прерывает stream; JSON adaptation до ответа возвращает 504.

FRC-00 остаётся **Implemented / Acceptance blocked by runtime evidence**. Новый candidate не закоммичен. Deployment, migrations и переход к FRC-02 не выполнялись.
