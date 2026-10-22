# Мультиагентная разработка: проект системы

Версия проекта: 1.0 · 22 октября 2026 · статус: готов к адаптации и пилоту, не установлен.

Пакет подготовлен по предоставленному ТЗ. Это новые инструкции, а не реконструкция оригинальных промптов. Исходные проекты и действующие конфигурации не изменялись. Корпоративная платформа, репозиторий, версии инструментов и доступность моделей не предоставлены; совместимость с ними не заявляется.

## Принятое решение

Восемь ролей, но не восемь обязательных вызовов на каждую задачу. Orchestrator только управляет. Code Researcher исследует. Specification Engineer ведёт требования в существующем OpenSpec. Software Architect определяет минимальное решение и план. Test Engineer создаёт тесты до реализации. Backend Developer реализует. Independent Reviewer независимо проверяет. Workspace Operator обслуживает снимки, diff и общие артефакты по необходимости.

Специалисты пишут только в своей зоне. Технические ограничения обеспечивают runtime, файловая изоляция и проверяющий шлюз; промпты сами по себе прав не ограничивают. Сборка исполняется в изолированном runner. OpenSpec остаётся единственным источником требований. Implementation Planner объединён с архитектором. Change Walkthrough запускается по запросу через Code Researcher в свежем контексте.

## Навигация

| Результат | Документ |
|---|---|
| Архитектура, роли, схема, Red/Green, возвраты | [architecture/workflow.md](architecture/workflow.md) |
| Матрица прав и способы обхода | [architecture/permissions.md](architecture/permissions.md) |
| Входы, результаты, артефакты, подтверждения | [architecture/agent-contracts.md](architecture/agent-contracts.md) |
| Выбор моделей и управление контекстом | [architecture/model-routing.md](architecture/model-routing.md) |
| Источники и границы применимости | [architecture/sources.md](architecture/sources.md) |
| Английские промпты | [agents/common.md](agents/common.md) + один из восьми role-файлов в agents/ |
| Scala FP и Scalafmt | [skills/scala-fp-conventions/SKILL.md](skills/scala-fp-conventions/SKILL.md) |
| Объяснение изменений | [skills/change-walkthrough/SKILL.md](skills/change-walkthrough/SKILL.md) |
| Платформонезависимый профиль | [config/platform-profile.example.yaml](config/platform-profile.example.yaml) |
| Схема результата | [contracts/result.schema.json](contracts/result.schema.json) |
| Смоделированные сценарии | [validation/workflow-scenarios.md](validation/workflow-scenarios.md) |
| Проверки технических ограничений | [validation/security-acceptance.md](validation/security-acceptance.md) |
| Проверка архитектуры и соответствие ТЗ | [validation/design-review.md](validation/design-review.md) |
| Внедрение и откат | [migration/implementation-plan.md](migration/implementation-plan.md) |

## Как применять впоследствии

1. Прочитать workflow и permissions; выбрать строгий профиль или явно обозначенный ограниченный пилот.
2. Адаптировать platform-profile к реальному репозиторию и runtime. Это декларация требований, не готовая конфигурация какой-либо платформы. Не загружать её как действующую политику.
3. Для каждого агента собрать system prompt из `agents/common.md` и его role-файла. Скиллы подключать отдельно. Ссылки на документы не заменяют загрузку общего промпта.
4. Провести acceptance-проверки из validation. Только после этого переключать роли по плану миграции.

Основные правила уже написаны; адаптация требует конкретных путей, команд, identity, схемы OpenSpec и моделей, а не самостоятельного дописывания поведения агентов.

## Что подтверждено

Проведена документальная проверка архитектуры и моделирование сценариев. Локальные проверки пакета описаны в [validation/package-checks.md](validation/package-checks.md). Запуск реальных корпоративных агентов, Scala-сборки и тестов безопасности целевой платформы не выполнялся. Все условные имена, пути и результаты в сценариях обозначают примеры.
