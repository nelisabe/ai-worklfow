# Документальная проверка архитектуры

Проведена авторская проверка проекта по ТЗ и сценариям. Это не независимый аудит развёрнутой системы и не доказательство enforcement на неизвестной платформе.

## Найденные риски и внесённые решения

| Риск | Решение в итоговом пакете | Остаточное ограничение |
|---|---|---|
| Architect и Implementation Planner дублируются | Одна роль владеет design и последовательностью tasks | Для большого проекта план может быть длиннее, но это не требует ещё одной роли |
| Operator превращается в универсального разработчика | Direct scoped writes; оператор только общие механические операции | Нужен publisher с проверкой provenance |
| Оркестратору скрыли tools лишь текстом | Отдельный dispatcher/tool allowlist, без FS и artifact bodies | Платформа может не поддержать; fallback честно ограничен |
| Тест не компилируется — «Red пройден» | Разделены test lock, missing API, scaffold и behavioral Red | Требуется безопасно исполнимый test harness |
| Build/IDE обходит права | Isolated runner, MCP method allowlist, readonly mounts | Только реальные негативные тесты докажут изоляцию |
| Тесты не менялись, но проверка отключена в build | Защищены build/CI/fixtures, контролируются selection/counts, reviewer ищет обходы | Семантические обходы не всегда определяются автоматически |
| Оркестратор тонет в артефактах | Direct refs, 8 KiB reports, bounded metadata, compact state | Runtime должен реально валидировать размер и schema |
| Круговая зависимость тесты↔реализация | Однонаправленный test-first, отдельный scaffold без логики | Исправления образуют управляемые возвраты, не циклы доверия |
| Review «успешен» при blocking findings | `failed/changes_requested` для дефектов; blocked для недостающих evidence | UI должен отличать результат стадии от завершения проекта |
| Reviewer чинит найденное | Только read + isolated checks + report | Необходимо исключить все IDE write methods |
| OpenSpec заменён служебным state store | OpenSpec хранит requirements/design/tasks; state store лишь receipts и execution state | Adapter должен учитывать установленную schema |
| Изменился код после passed | Exact snapshot + dependency digests, invalidation и CAS | Нет безопасного reuse при неизвестных входах |
| Маленькая задача вызывает всех | Reuse подтверждённых research/spec/design; Operator по необходимости | Независимые tests/validation/review не сокращаются |
| Walkthrough обязателен всегда | Только явный запрос, отдельный Researcher context, explicit-only metadata | Natural-language routing на иных платформах реализуется adapter |
| Жёсткий Red делает рефакторинг бессмысленным | Для неизменного поведения characterization и явный not_applicable с review | Исключение нельзя использовать для нового поведения |

## Дублирование и зависимости

Researcher описывает факты, Spec определяет желаемое поведение, Architect определяет способ, Test задаёт независимый oracle, Developer реализует, Reviewer оценивает. Тестировщик и Reviewer оба проверяют результат, но по разным основаниям: первый владеет исполнением/полнотой тестов, второй — общей независимой оценкой с реальным diff. Это намеренное разделение, а не два одинаковых review.

Граф доверия не содержит самоутверждения: автор не выдаёт собственный review accept; runner создаёт факты запуска; runtime проверяет переходы; пользователь решает материальную бизнес-неоднозначность. Возвраты ограничены и инвалидируют downstream evidence. Отдельные services — технические обязанности, которые можно встроить в существующий CI, а не новые LLM-агенты.

## Соответствие критериям готовности ТЗ

| № | Критерий | Где реализован |
|---|---|---|
| 1 | Целостная архитектура | workflow + agent-contracts |
| 2 | Orchestrator без кода/FS/research | permissions + orchestrator prompt + profile |
| 3 | Нет лишнего дублирования | role table, сравнение Operator, объединённый Planner |
| 4 | Реальные ограничения | strict profile, fallback, P01–P22 |
| 5 | Независимое test-first | TESTS_LOCKED/RED, missing-API маршрут, test prompt |
| 6 | Минимальная архитектура | Architect contract, principles, Reviewer |
| 7 | Объяснение изменений | change-walkthrough, scenario 4 |
| 8 | Scala FP + обязательный Scalafmt | scala-fp-conventions, command discovery и receipts |
| 9 | Готовые английские prompts | common + 8 role files, каждый с 10 требуемыми разделами |
| 10 | Проверка типовых сценариев | 7 документальных симуляций; runtime acceptance отдельно |
| 11 | Реалистичное внедрение без оригиналов | migration plan и compatibility assessment |

Пакет покрывает результаты A–G: архитектура, prompts, skills, permissions/config, сценарии, проверка проекта, миграция. Проверки оформления и машинных контрактов регистрируются в [package-checks.md](package-checks.md). Наличие документов не приравнивается к внедрению.
