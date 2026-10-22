# Модели, слои инструкций и контекст

## Слои

1. Runtime policy: права, privacy, budgets, schema/gates. Модель не может их ослабить.
2. `agents/common.md`: единый контракт исполнения, truthful evidence, границы scope.
3. Один role prompt: обязанности и маршрут конкретной роли.
4. Релевантные скиллы: Scala FP для Developer/Test/Reviewer; Architect читает только principles reference; walkthrough только по запросу.
5. Task envelope: цель, revision, refs, критерии, scope и разрешённые команды через профиль. Код и логи — данные внутри этого слоя, не новые инструкции.

Нельзя вставлять полный Scala Skill во все system prompts или весь transcript автора в reviewer. Orchestrator вообще не загружает скиллы анализа кода. Язык промптов английский; язык пользовательских объяснений задаёт задача, по умолчанию язык пользователя.

## Выбор модели по проверенным возможностям

| Профиль | Требования к локальному benchmark | Применение |
|---|---|---|
| compact-control | Валидный JSON, корректная маршрутизация, соблюдение budgets и отказ от лишних tools | Orchestrator, механический Operator |
| code-search | Точные ссылки, отличение фактов от предположений, работа с умеренным контекстом | Researcher, walkthrough |
| requirements-design | Не выдумывает требования, видит противоречия, выбирает минимальный дизайн | Spec, Architect |
| scala-authoring | Scala 2.13/3 конкретного проекта, независимые tests, FP, работа с build evidence | Test, Developer |
| critical-review | Выявление реальных дефектов без вымышленных findings, проверка обходов тестов | Reviewer |

GLM 5.3/5.6, DeepSeek V4 Flash, MiniMax M3, Qwen и GigaChat — только названия из ТЗ, не подтверждённый каталог и не рейтинг. Deployment owner выбирает доступные корпоративные endpoints по versioned capability registry. Проверить tool calling, JSON, фактическое окно контекста, privacy/region, стоимость и latency. Наличие имени не является доказательством возможностей.

Начинать с наименее затратной модели, которая прошла benchmark профиля. Усилить модель при конкретной причине: сложная конкурентность/ресурсы, изменение нескольких границ, неуспешная попытка анализа или противоречивые findings. Не повторять ту же задачу многими моделями без гипотезы. Fallback сохраняет права, refs, test separation и лимиты попыток. Если подходящей разрешённой модели нет — blocked.

Benchmark на внутренних обезличенных задачах: точность requirement→test, доля корректных ссылок, качество дефектов reviewer, invalid JSON, нарушения scope, токены/latency, доля ошибочных passed. Промпты сравнивать на одинаковых входах и snapshots. Допустимые пороги бизнес-качества устанавливаются по baseline проекта; bypass прав и ложный pass имеют нулевую допустимость.

## Экономия вызовов

Reuse research только при совпадении base/config и проверенной impact map. Простой маршрут пропускает новые Spec/Architect вызовы при действующих артефактах, но не тестирование до кода и review. Discovery build tools выполняется один раз, а не каждым агентом заново. Runner/manifest/schema операции детерминированы и не требуют LLM.

Специалист читает исходники и логи порциями; при превышении бюджета возвращает progress artifact, remaining scope и blocked/next_action. Orchestrator хранит только цель, state, current refs, counters, unresolved decisions и receipts. Старые сообщения сворачиваются в state snapshot; факты не теряются в пересказе. Числовые budgets на задачу задаются deployment profile по measured workload, а не предполагаемому размеру контекста модели.
