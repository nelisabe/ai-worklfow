# Моделирование взаимодействия

Это документальные симуляции на условном Scala-проекте, не результаты запуска реальных моделей, Scala-компилятора или тестов. `S0`, `C1`, `R1` и другие короткие refs ниже — условные IDs. Таблицы показывают payload по существу; при исполнении он оборачивается в JSON-контракт со всеми обязательными полями. Ни один пример не добавляет требования настоящему проекту.

## 1. Простое изменение существующего Scala-метода

Условное принятое требование OpenSpec `limit-inclusive`: существующий метод `withinLimit(value, limit)` должен принимать равенство границе. Сейчас он использует строгую границу. API, зависимости и модель ошибок не меняются. Значения и допустимый диапазон уже определены действующим spec.

| Вызов | Вход | Результат исполнителя | Действие Orchestrator |
|---|---|---|---|
| Code Researcher | Вопрос о методе, S0, spec ref | success; R1 с методом/путём/строками, тестами, toolchain и подтверждением local scope | Reuse действующих spec/design; новые Spec/Architect вызовы не нужны |
| Test Engineer / pre | S0, spec, existing contract, R1; без новой реализации | success; T1 с cases ниже/на/выше границы, oracle refs; red receipt: equality assertion fails, baseline classified | Фиксирует TESTS_LOCKED → RED_CONFIRMED |
| Backend Developer | Spec, contract, T1, Red, только production path | success; C1 с минимальным исправлением, tests hash T1 сохранён, self-check refs | Передаёт C1 на независимую проверку |
| Test Engineer / verify | C1, T1, required checks | success; V1 с compile/test/format passed на C1, test count/selection | Запускает Reviewer |
| Independent Reviewer | S0→C1, spec/design/T1/V1; свежий контекст | success; RV1 accept, issues пусты, проверена граница | Запрашивает DONE по receipts |

Пример краткого финального смысла: «Граница стала включительной; проверены значения ниже, на и выше лимита; обязательные проверки подтверждены на C1». Реальный оркестратор сообщает только факты, присутствующие в trusted receipts.

Обязательные роли не умножаются из-за малого изменения. Workspace Operator не вызывается, если snapshot/diff service уже отдаёт нужные refs. Отсутствие актуального spec/design возвращает обычный маршрут, а не превращает Developer в автора требований.

## 2. Новая функциональность с test-first и отсутствующим API

Условный запрос: добавить retry только из состояния Failed; из Running возвращать существующую типизированную ошибку InvalidTransition. Поведение других состояний должно быть определено до реализации.

| Вызов | Вход | Результат | Следующий шаг |
|---|---|---|---|
| Researcher | S0, вопрос о state machine и persistence | success; R2: существующий state owner, ошибки, tests, атомарность | Spec |
| Specification Engineer | Запрос, OpenSpec, R2 | blocked; Q2: какие остальные состояния допускаются? варианты с последствиями | Пользователь уточняет: retry только из Failed, остальные — InvalidTransition |
| Specification Engineer | Принятый ответ и те же refs | success; SP2 в существующем change, scenario refs, validation receipt | Architect |
| Architect | SP2, R2, S0 | success; D2: использовать текущий state owner, добавить метод, сохранить атомарность; новый service не нужен; API signature + scaffold boundary + tasks | Test Engineer |
| Test Engineer / pre | SP2/D2/S0, existing test stack | success для подзадачи lock; T2 и cases всех состояний, повторного вызова/ошибки хранения по spec; `missing_api` receipt | TESTS_LOCKED; полной реализации ещё нет |
| Developer / scaffold | D2 signatures, T2, узкий grant | success; S1 — только отсутствующие объявления, явно незавершённое поведение | Test Engineer проверяет scaffold |
| Test Engineer / Red | T2, S1, SP2/D2 | success; компиляция прошла, тесты упали на целевом поведении; RED2 behavioral; scaffold без новой логики | RED_CONFIRMED |
| Developer / implement | S1, SP2/D2/T2/RED2 | success; C2, implementation + self-checks; T2 не изменён | Independent validation |
| Test Engineer / verify | C2, T2, check registry | success; V2, обязательные checks passed и актуальны | Reviewer |
| Reviewer | S0→C2, SP2/D2/T2/RED2/V2 | success; RV2 accept после проверки перехода и тестов | DONE |

Неуспех компиляции на отсутствующем API не выдаётся за Red поведения. Сценарии и test sources существуют до scaffold и реализации; scaffold — отдельная фаза с ограниченными правами. Если scaffold содержит готовую логику, Test Engineer возвращает failed с protocol violation: новый test-first цикл на чистом base либо эскалация, без ретроспективного объявления порядка соблюдённым.

Изменение архитектурного контракта, требующее другой сигнатуры, создаёт D3 и инвалидирует T2/RED2; оркестратор не передаёт старый Red как разрешение реализации нового API.

## 3. Ошибка или избыточная архитектура на ревью

В условном C3 добавлены `RetryManager`, `RetryValidator` и отдельный интерфейс с единственной реализацией, хотя D2 назначил ответственность существующему state owner. Также обнаружено, что проверка состояния и запись разнесены так, что параллельные запросы нарушают заданную атомарность.

| Вызов | Вход | Результат | Маршрут |
|---|---|---|---|
| Reviewer | C3, D2, SP2, V3 | failed; RV3 changes_requested; F1 major/blocking: необоснованные слои против D2; F2 major/blocking: воспроизводимый race с двумя вызовами и ссылкой на invariant | Developer, задача исправления F1/F2 |
| Developer | RV3, D2, production grant | blocked, если исправление требует новой transaction boundary; A3 deviation request с причиной | Architect |
| Architect | D2, A3, R2/SP2 | success; D3 сохраняет существующего владельца и конкретизирует атомарную операцию; лишние классы не нужны | Test Engineer для затронутого invariant |
| Test Engineer | SP2/D3, предыдущий test manifest, regression scope | success; T3 с дополнительным deterministic regression, пометка post-implementation discovery; Red на дефектном снимке | Developer |
| Developer | D3/T3, F1/F2 | success; C4, упрощение и исправление в production, test hash сохранён | Validation |
| Test Engineer | C4/T3 | success; V4 на C4 | Re-review |
| Reviewer | C4, V4, D3, F1/F2 | success; RV4 accept, закрытые finding IDs с проверкой смежных путей | DONE |

Для одного лишь удаления лишнего слоя в рамках D2 Architect повторно не нужен. Для реального нового архитектурного решения он обязателен. Reviewer не удаляет классы и не меняет тесты самостоятельно. Старый V3/RV3 не подходит к C4. Дополнительный тест T3 не выдаётся за первоначальный test-first набор; исходный T2 и время его создания сохраняются.

Если после двух correction cycles тот же race остаётся, Orchestrator возвращает blocked с evidence и вариантами решения пользователю; следующий model switch не обнуляет лимит.

## 4. Change Walkthrough по запросу пользователя

Условный запрос после DONE: «Объясни изменения retry: с чего начать чтение, как идёт запрос и что проверить вручную».

| Вызов | Вход | Результат |
|---|---|---|
| Orchestrator → Code Researcher | Явный запрос, base/target из завершённой задачи, SP2/D3/T3/V4 refs, skill, свежий контекст | Researcher читает diff/source напрямую; никакого Developer transcript |
| Code Researcher | Pinned S0→C4, названные refs | success; W4 report: краткая суть, 4 ключевых точки, методы/строки, data flow, требования→тесты→фактические receipts, риски, порядок чтения |
| Orchestrator → пользователь | Summary и W4 link | Понятный обзор через artifact viewer; новые development stages не запускаются |

Содержание W4 в этой симуляции: входной retry method → решение о допустимости → атомарное обновление → тесты. В настоящем отчёте имена и строки берутся из C4; здесь они намеренно не выдуманы. Если есть исходник теста, но нет V4, формулировка должна быть «тест найден, результат запуска неизвестен».

Skill не запускается автоматически после каждого DONE. При нескольких несвязанных uncommitted changes Researcher уточняет базу и диапазон; не выбирает весь workspace без основания.

## 5. Неоднозначность и ошибка oracle

Developer сообщает, что T5 ожидает повторный retry из Pending, но SP5 разрешает только Failed. Он передаёт ссылку на scenario и test ID, не редактирует тест. Test Engineer проверяет spec: если это ошибка теста — новая версия T6 с объяснением, Red на прежнем base/scaffold для изменённого oracle, новые checks и независимый review корректировки. Если spec сам противоречив — Specification Engineer → пользователь, затем новая версия SP6 и invalidation downstream. Green путём ослабления assertions без этого процесса запрещён.

## 6. Инструмент недоступен или базовые тесты красные

Scalafmt не стартовал из-за закрытого registry. После одного безопасного повторения Test Engineer возвращает blocked, outcome not_run, log ref, необходимый доступ/кэш. Orchestrator не читает большой лог и не заменяет форматирование визуальной оценкой. Runtime либо предоставляет разрешённый cached tool/runner, либо пользователь решает вопрос окружения. Неизвестный Maven plugin не превращается в выдуманный goal.

Если baseline suite уже падает, Test Engineer разделяет старые и новые ошибки. Новый нужный assertion failure может подтверждать Red, но обязательный Green нельзя объявить по сравнению «стало не хуже». Нужен исправленный baseline или ранее утверждённая явная acceptance exemption; без этого финальная задача blocked.

## 7. Попытка обхода прав и устаревший результат

Developer просит `source.edit_production` записать `../tests/RetrySpec.scala`, затем пробует IDE rename и formatter на весь проект. Gateway отклоняет пути/метод; runner не имеет доступа к live tests. Bundle с изменённым fixture отклоняется publisher. Не выдаётся новый grant по просьбе агента. Audit хранит отказ и роль.

После успешной валидации C7 другой writer публикует C8. Переход DONE с V7/RV7 отвергается state service, даже если JSON агентов говорит success. Требуются проверки и review для C8. Hash и ACL проверяются сервисом, не обещанием в промпте.
