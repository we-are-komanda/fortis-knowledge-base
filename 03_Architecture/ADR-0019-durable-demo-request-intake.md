# ADR-0019: durable приём заявок на демонстрацию

- Дата:2026-09-20
- Статус:принятое решение в рамках FRC-12; публикация выключена до runtime/owner evidence.
- Связи:[[ADR-0016-runtime-backend-boundary]], [[ADR-0017-enterprise-access-boundary]], [[../01_Requirements/Release_2026_09_20_Autonomous_Run]]

Публичная форма отправляет same-origin POST через BFF в backend. Успех показывается только после атомарного commit заявки+outbox+idempotency receipt в PostgreSQL. Одинаковая нормализованная попытка с тем же ключом24h возвращает прежний ID. Ввод сохраняется в памяти формы при ошибке, не в browser storage. Публичный transport не изменяет рабочую auth-сессию при ошибке401/403/404.

Лимиты5 попыток/10min на доверенный client IP и3 принятые заявки/hour на нормализованный email хранятся в БД. Browser Origin проверяется точно, тело≤16KiB, поля валидируются сервером, значения не попадают в headers/logs. BFF передаёт только IP из настроенного ingress-overwritten header, backend доверяет лишь заданным socket CIDR. Generic proxy не переносит пользовательские Origin/XFF в этот канал.

Один SMTP канал использует обязательный STARTTLS с проверкой сертификата. Получатель/учётные данные — server-only. Worker арендует outbox атомарно, повторяет1/5/30min и затем hourly до24h. Стабильный eventId/Message-ID не обещает exactly-once. Заявка остаётся durable при сбое уведомления; статусpending/sent/failed доступен оператору.

Реестр — локальный read-only command `APP_WORK_MODE=demo-request-registry ./app --limit 100 --offset 0`. Доступ определяется существующими OS/DB-привилегиями оператора; новая пользовательская роль или публичный admin API не вводятся. Оператор и процедура доступа должны быть назначены владельцем; ordinary membership не даёт доступ.

Production сбор disabled по умолчанию. До включения владелец предоставляет текст согласия/версию, политику retention, получателя, ответственного за failed и проверенный alert канал. Никакой юридический текст или срок хранения данных этим ADR не придумывается. Настройка ingress trust, фактическая доставка, восстановление worker и retention являются runtime gates FRC-14. В тестах только synthetic example.test и fake notifier; реальные письма не отправлялись.

Миграция1789913977_demo_requests добавляет durable lead/outbox/receipt/rate tables. Production не применялась. Подробные env и ограничения: parent `docs/releases/2026-09-20/frc12/runtime-configuration.md`.
