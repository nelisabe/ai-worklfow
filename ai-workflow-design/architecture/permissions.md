# Права: техническая модель

## Матрица ролей

`R` — чтение разрешённого snapshot; `W(zone)` — запись только перечисленных путей; `Q` — типизированный запрос сервису, без shell; `—` — инструмент отсутствует. Любой read ограничен проектом, revision и классификацией данных. Собственный отчёт публикуется через `artifact.publish`, а не через произвольную файловую запись.

| Роль | FS Read | FS Write/Edit | Shell/Bash | Git | IDE MCP | Build/Test | Delegation | Skills | OpenSpec edit |
|---|---|---|---|---|---|---|---|---|---|
| Orchestrator | — | — | — | — | — | — | Q, роли allowlist | Только routing metadata | — |
| Code Researcher | R scoped base/candidate | — | — | Q read | Q read, если изолирован | — | — | walkthrough по запросу | — |
| Specification Engineer | R OpenSpec + research; base при нужде | W(spec change) | — | Q read | — | Q spec validation | — | OpenSpec workflow проекта | Только proposal/requirements выделенного change |
| Software Architect | R base + research/spec | W(design/tasks) | — | Q read | Q read | — | — | Scala principles reference | Только design/tasks выделенного change |
| Test Engineer | R base; candidate после реализации | W(test zone) | — | Q read | Q read/approved test edit | Q compile/test/format | — | Scala FP | — |
| Backend Developer | R scoped code/spec/tests | W(production zone) | — | Q read | Q read/approved production edit | Q compile/test/format | — | Scala FP | — |
| Independent Reviewer | R exact candidate, spec, tests, reports | — | — | Q read | Q read | Q проверки в отдельном runner | — | Scala FP, review mode | — |
| Workspace Operator | R manifests/diff; полный код лишь если необходим операции | W(shared operational artifacts) | — | Q snapshot/status/diff | — | — | — | Только operational instructions | Только механические отметки tasks по receipts |

Workspace Operator может запрашивать интеграцию bundle через сервис; это не право редактировать чужие зоны. Service проверяет автора, его исходные права, base и hash. Tool для отметки tasks принимает task IDs и receipts, а не произвольный текст. Акты утверждения требований, grants, checks и review создают разные identities; оператор не может их подписывать.

## Реальные границы записи

Production zone — конкретные исходники и ресурсы выбранного изменения, не весь `src/`. Test zone — конкретные tests, fixtures и test resources; туда входят snapshots/golden files, влияющие на assertions. Файлы со смешанной ответственностью требуют отдельного path grant и двухстороннего review; glob по расширению `.scala` недостаточен. Для нового файла gateway разрешает только конкретный путь или узкий каталог с проверкой категории.

Build scripts, CI, зависимости, `.scalafmt.conf`, IDE settings, agent prompts, policy, Git metadata и hooks по умолчанию защищены. Если задаче необходима смена зависимости/build: Architect обосновывает, пользователь или действующая политика проекта санкционирует узкое расширение, runtime выдаёт отдельную capability с expiry, затем независимый review. Агент не может выдать grant себе. Семантическое изменение build принадлежит Developer по специальному grant, а не «механическому» Operator.

Test Engineer не может менять main source через генератор, symlink, IDE refactor или shell. Developer не может менять assertions, fixtures, выбор тестов и skip-флаги, чтобы получить Green. Проверка неизменности test hash необходима, но недостаточна: reviewer также проверяет обходы в production и build, изменение часов/random/окружения, специальные ветви для тестов.

## Строгий профиль

1. **Разные identities и контексты.** Каждая роль получает собственные credentials/capabilities и свежий контекст. Child spawn по умолчанию не наследует полный диалог; тестировщику до реализации и reviewer не передаются рассуждения Developer. Делегировать может только Orchestrator, только известным ролям, без изменения их системных правил и прав.
2. **Tool allowlist на сервере.** Отсутствующий tool действительно не маршрутизируется. Скрытие кнопки в UI недостаточно. Методы MCP allowlisted отдельно; универсальные `execute`, `eval`, `runCommand`, `applyWorkspaceEdit` запрещены там, где нельзя контролировать эффект.
3. **Изолированная файловая система.** Каждый writer получает base read-only + writable overlay своей зоны + private scratch. Запрещены запись в родительский каталог, rename/delete защищённого файла и изменение ACL. На Windows отдельные identities/контейнеры с ACL; на Linux read-only mounts/отдельный uid. Общая ОС-учётка с доступом к обходному shell не обеспечивает роль-boundary.
4. **Path gateway.** Канонизация, проверка корня и обоих путей rename, запрет traversal, symlinks, junctions/reparse points, hardlinks, UNC/device/alternate data streams вне политики. Проверка перед открытием handle и перед публикацией, без TOCTOU. Учитывать case-insensitive aliases. Удаление считается записью. Glob match сам по себе не является sandbox.
5. **Изолированный runner.** Сборка — выполнение недоверенного кода, даже команда `test` может писать файлы и открывать сеть. Snapshot источников read-only, отдельные writable output/cache/temp, без пользовательских токенов, домашнего каталога и host sockets. Test services одноразовые. Сеть закрыта, кроме корпоративных registry/service allowlists. Cache не общий writable между недоверенными задачами; dependency provenance контролируется.
6. **Публикация.** Diff вычисляется доверенным сервисом. Принимаются только изменения в granted paths, с provenance автора и актуальным base. Generated source/output не переносятся в исходники автоматически. Bundle с лишними файлами отклоняется целиком. Hash test tree, build config и spec фиксируется независимо от Developer.
7. **Проверяемые gates.** Runner сам создаёт checks receipts: команда/argv, environment fingerprint, snapshot, test selection/count, exit code, время, logs ref. Artifact service удостоверяет владельца и content hash. State machine проверяет receipts; JSON от агента без подтверждения не является доказательством запуска.

Права записи выдаются также по фазе, не только по роли. До TESTS_LOCKED Developer не получает production grant. После `missing_api` разрешается лишь grant на согласованный scaffold; после подтверждённого behavioral Red — grant на полную реализацию в пределах scope. Для рефакторинга без нового поведения требуется отдельно подтверждённый Red exemption. При изменении spec/design/base либо переходе в blocked grants зависимых writers отзываются; старый attempt не может продолжить запись. Gateway проверяет phase и attempt при каждом edit/publish, а не только при создании сессии.

Read-only Git выдаётся через adapter с фиксированными операциями status/diff/show/log/blame; без произвольных flags, hooks, external diff/textconv, pager, shell aliases и доступа к чужим repos. Операции выполняются на изолированном snapshot. Git branch/worktree — удобство, но не security boundary.

IDE read-only тоже требует проверки: language server может создавать индексы и запускать build import. Кэши и import только в isolated scratch/runner. Если сервер не может отделить read от произвольного execution, инструмент этой роли отключается.

## Если платформа ограничений не поддерживает

| Возможности платформы | Допустимый вариант | Честное ограничение |
|---|---|---|
| Tool allowlist + отдельные FS/process identities | Строгий профиль | Проверить acceptance перед запуском |
| Tool allowlist, но общий writable workspace | External sandbox и publisher; до их появления только disposable pilot | Post-hoc diff обнаруживает нарушение, но не предотвращает его |
| Один агент с общими tools, смена роли только текстом | Ручные отдельные сессии, read-only snapshots, ручная интеграция | Нет технически гарантированных прав и независимости ролей |
| Нет безопасного build runner | Внешний CI/человек запускает проверки и отдаёт заверенные результаты | До получения результатов задача blocked, не «всё прошло» |
| Нельзя запретить orchestrator filesystem/shell | Внешний dispatcher с отдельной identity либо ручной диспетчер | Такой in-app orchestrator не соответствует строгому ТЗ |

Не компенсировать отсутствие sandbox дополнительными запретами в тексте. Не объявлять ограниченный пилот промышленно готовым. Адаптируемые параметры перечислены в [profile](../config/platform-profile.example.yaml); обязательные негативные проверки — в [security acceptance](../validation/security-acceptance.md).
