# ADR-0024: Сравнение двух сохранённых ревизий

Дата:2026-09-21. Реализация утверждённого пользователем FRC08 после принятой FRC07. Acceptance фиксируется отдельно.

Обе стороны сравнения — явные ProjectRef:{projectId,projectVersion}, соответствующие существующим immutable snapshots [[ADR-0018-immutable-project-revisions]]. Сервис загружает каждую выбранную ревизию один раз и использует её состав, frozen bindings, цены [[ADR-0020-canonical-versioned-cost]], бюджет [[ADR-0023-authoritative-project-budget]] и происхождение. Current state и изменившийся внешний каталог не обогащают старую ревизию. Отсутствующая поддержка её calculationVersion даёт fail-closed503.

GET /api/v1/projects/compare требует projectIdA,projectVersionA,projectIdB,projectVersionB. Compatibility id1/id2 допустимы только как ID aliases: версии обязательны, несовпадающие aliases400. Авторизация обеих сторон предшествует расчёту. Чужая/отозванная/отсутствующая сторона404 без частичного результата; два доступных разных предприятия либо полностью одинаковые ссылки400 incompatible_projects. Разные версии одного проекта допустимы. Прежнее implicit-current сравнение не является release API.

Ответ CompareResult:{enterpriseId,a,b,differenceMinor,differencePercent,warnings}; каждая сторона содержит identity,name,updatedAt,cost,budget той же pinned revision. Разность B−A — signed decimal minor string. Процент (B−A)/A×100 рассчитывается без float и округляется до двух знаков; при A=0 либо неполной стороне null. При unknown total обеих/одной стороны разность null, knownSubtotal сохраняется. Cost lines/groups дают реальные objectId/assetId/layerId, количества и происхождение; одинаковые имена не являются ключами совпадения.

UI хранит выбранные ссылки независимо от обновляемого списка текущих проектов. Фоновое сохранение не передвигает сравнение на новую версию. Для затронутого dirty project пользователь явно сохраняет и сравнивает подтверждённую новую версию либо сравнивает last-saved. Неоднозначный/failed save не запускает сравнение нового состояния. Поздний ответ при смене ссылки, предприятия или identity не показывается как новый результат. Старый результат убирается на новую операцию/ошибку.

Сравнение показывает стоимость/бюджет/состав, а не эффективность защиты. Отсутствие conflict flags не доказывает покрытие или достаточность. Предложение CMP09 о viewer/viewCost отложено по [[ADR-0017-enterprise-access-boundary]]: действующая граница — membership, полный RBAC не вводится этим этапом. HideCost официального отчёта — отдельная обязательная FRC10 server projection.

Без новой модели конфигурации, миграций, reservation state или внешних сервисов. Parent evidence: docs/releases/2026-09-20/frc08/.
