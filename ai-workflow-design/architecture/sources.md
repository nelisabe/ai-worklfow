# Источники и предположения

Проверено 22 октября 2026. Ссылки служат для сверки терминов и инструментов; это не фиксация версий корпоративного окружения.

| Первичный источник | Использованный факт | Следствие для проекта |
|---|---|---|
| [OpenSpec Getting Started](https://github.com/Fission-AI/OpenSpec/blob/main/docs/getting-started.md) | CLI-команды и slash-команды ассистента различаются; change artifacts входят в существующий workflow | Не предполагать, что `/opsx:*` можно выполнить как shell-команду; не запускать init поверх проекта |
| [OpenSpec Workflows](https://github.com/Fission-AI/OpenSpec/blob/main/docs/workflows.md) | Последовательность зависит от schemas; артефакты можно пересматривать | Сопоставлять роли существующим артефактам; gates Red/Review — наша политика, не встроенная гарантия OpenSpec |
| [Scalafmt Installation](https://scalameta.org/scalafmt/docs/installation.html) | Есть CLI, sbt и интеграции с Maven/Gradle; у sbt существуют отдельные задачи для main/test и отдельных файлов | Команды определяются установленным plugin и scope, без универсального Maven/Gradle goal |
| [Scalafmt Configuration](https://scalameta.org/scalafmt/docs/configuration.html) | `.scalafmt.conf` задаёт version и dialect; есть варианты Scala 2 и Scala 3 | Проверять фактическую конфигурацию, не подменять dialect и formatter version |
| [Scala: Functional Error Handling](https://docs.scala-lang.org/scala3/book/fp-functional-error-handling.html) | Option/Try/Either представляют разные способы обработки отсутствия и ошибок | Выбирать тип по семантике, сохранять информацию об ошибке |
| [Scala: FP Summary](https://docs.scala-lang.org/scala3/book/fp-summary.html) | FP включает immutable values, pure functions и функции как значения | Языковой skill опирается на FP без навязывания effect-библиотеки |

Матрица прав, immutable receipts, workflow gates, budgets и выбор ролей — проектные рекомендации по этому ТЗ. Они не приписываются перечисленным продуктам как готовые функции. Доказательства наличия sandbox, per-role tools, build isolation и ACL должны быть получены на целевой платформе во время пилота.

Неизвестны: версия/схема OpenSpec, Scala modules, test stack, effect stack, source/test roots, formatter plugin, CI, корпоративные модели, секреты и сетевые ограничения. Их выявляет compatibility assessment. До этого никакие примерные paths, commands или model IDs не считаются разрешёнными значениями.
