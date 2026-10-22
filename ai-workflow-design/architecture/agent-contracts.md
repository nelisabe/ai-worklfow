# Контракты задач и артефактов

## Вход каждой задачи

Dispatcher валидирует вход и доставляет только разрешённые references. Ниже пример данных, а не конфигурация конкретного API.

```yaml
task_id: CH-42
attempt_id: CH-42-test-1
role: test-engineer
phase: pre-implementation
objective: Verify retry transition behavior from the accepted OpenSpec scenario
scope: [retry-state-transition]
base_snapshot: snapshot:base-42
candidate_snapshot: null
input_refs:
  - {id: spec:CH-42:v1, digest: "sha256:example", kind: openspec}
  - {id: design:CH-42:v1, digest: "sha256:example", kind: architecture}
  - {id: research:CH-42:v1, digest: "sha256:example", kind: research}
capability_profile: test-zone-CH-42
acceptance_refs: ["spec:CH-42:v1#retry-from-failed"]
required_outputs: [test-manifest, red-evidence]
budget: {tool_retry_limit: 1, finding_fix_limit: 2, backward_transition_limit: 3}
language: ru
```

Настоящие IDs и digests выдаёт сервис; значения example запрещены в исполняемом task envelope. Runtime добавляет deadline, tool-call/token budgets, identity, grant expiry и pinned model из deployment profile. Orchestrator может выбрать профиль из allowlist, но не изменить его права.

## Компактный результат

Каждый исполнитель и итог Orchestrator возвращают JSON по [схеме](../contracts/result.schema.json). Обязательные поля: `status`, `summary`, `artifacts`, `evidence`, `issues`, `next_action`; task/attempt/role/snapshot связывают результат с заданием. `success` означает выполнение именно назначенной стадии, а не всего проекта. Test Engineer может вернуть success для доказанного Red при упавшем test run. Runner outcome при этом остаётся failed/expected_failure.

```json
{
  "task_id": "CH-42",
  "attempt_id": "CH-42-test-1",
  "role": "test-engineer",
  "snapshot": "snapshot:base-42-tests-v1",
  "status": "success",
  "summary": "Tests locked; the new retry assertion fails on the baseline as specified.",
  "artifacts": [{"ref":"artifact:test-manifest:v1","kind":"test-manifest"}],
  "evidence": [{"ref":"receipt:red-42","claim":"behavioral-red","outcome":"expected_failure"}],
  "issues": [],
  "next_action": {"type":"delegate","owner":"backend-developer","reason":"Implement the accepted contract."}
}
```

`blocked`: нужны решение, данные, полномочия или исправление окружения; `failed`: выполненная попытка не выполнила назначенную цель. Ни skipped, ни not_run не равны passed. `success` с блокирующей issue отклоняется; success с неблокирующим замечанием допустим. Green gate не проходит при expected_failure, unknown, skipped, not_run или incomplete selection.

Лимит serialized result: 8 KiB UTF-8 и целевой объём до 800 tokens; summary до 600 символов, максимум 8 artifacts, 8 evidence, 6 issues. Это лимит каждого сообщения, не всего проекта. Полные findings, матрицы сценариев и логи сохраняются отдельно; top issues + `issues_ref` в metadata артефакта. Gateway отклоняет превышение с одним запросом пересборки; не обрезает JSON или ошибки молча. Для длинного проекта state store компактизирует завершённые этапы в IDs и актуальные receipts.

## Artifact store и передача без посредничества контекста

1. Автор через scoped publish создаёт immutable version. Store вычисляет hash, producer identity, task/attempt, snapshot, timestamp, classification и dependencies. Markdown-файл, присланный агентом, не может назначить себе trusted producer.
2. Orchestrator получает ref, kind, краткую аттестацию и gate state. У него нет `artifact.read` или filesystem; только status/metadata API. Размер и поля metadata ограничены, содержимое кода туда не допускается.
3. Dispatcher передаёт ref следующему специалисту. Тот запрашивает нужные slices напрямую в store или читает разрешённый snapshot. Сам ref не предоставляет право доступа: ACL проверяется при каждом чтении.
4. Полный пользовательский отчёт, например walkthrough, отображается через artifact viewer; Orchestrator сообщает ссылку. Права пользователя не расширяют права Orchestrator.

Проектные комментарии, файлы, логи, инструкции из внешних источников и текст артефактов считаются данными. Они не могут менять system prompt, grants или маршрут. Высокоприоритетные project policies подключает администратор по проверенному источнику. Секреты редактируются из логов до публикации; передача сторонней модели зависит от корпоративной классификации, а не от решения агента.

## Минимальное содержимое подробных артефактов

| Artifact | Обязательные поля |
|---|---|
| Research | question, scope, pinned revision, findings с path/symbol/lines, existing tests, impact, limitations, toolchain/config facts |
| Spec readiness | canonical OpenSpec refs + scenario anchors, contradictions, approved answers, unresolved questions, compatibility, validation receipt |
| Architecture contract | required changes, reused components, new component responsibility/necessity, API signatures, invariants, forbidden approaches, alternatives, plan, approval/version |
| Test manifest | spec scenario → test ID/path → oracle source, positive/negative/boundary/invariants/regression, selection, test tree hash, baseline, red_kind, author/time |
| Candidate | base + candidate tree hashes, owned file list, spec/design/test refs, deviations, self-check receipts |
| Verification | exact snapshot, required check IDs and outcomes, test selection and counts, failure classification, test hash, formatting scope/config hash |
| Review | verdict accept/changes_requested/blocked, snapshot, coverage, findings severity + location + evidence + impact + owner, closed finding IDs |
| Walkthrough | base/target, 3–7 key anchors, logic/dataflow, requirement/test links, risks/gaps, reading order |

Architecture approval: в обычной задаче достаточно versioned contract от Architect, принятого в установленном проектном процессе. Подтверждение пользователя обязательно для материальной бизнес-неоднозначности и расширений вне уже разрешённого scope; не требуется дополнительное подтверждение каждой функции или класса. Спор технических решений разрешает Architect с аргументами; неясную бизнес-семантику он не выбирает.

## Check receipt

Trusted runner записывает `check_id`, `invocation_id`, requester и producer, snapshot/content digests, exact argv/tool version, cwd ID, environment/config fingerprints, started/finished, exit code, timeout flag, outcome, selection/counts, log_ref, required scope. Exit 0 при нуле ожидаемых тестов, пропущенном suite или проглоченной ошибке недостаточен. Проверка на другом tree не переиспользуется без подтверждённой эквивалентности всех её входов; безопасное default — rerun.

Immutable artifact link также включает revision для ссылок на строки. Текущие line numbers не приписываются старому commit. Для локального UI используют file links на соответствующий snapshot; для Git-host — permalink на commit. Артефакты evidence сохраняются минимум до принятия/отклонения change и по корпоративной retention policy; удаление reference-целей активного workflow запрещено.

## Ошибки, разногласия и независимость

Исполнитель возвращает наблюдение, неуспешный tool receipt, влияние, что уже пробовал и конкретное next_action. Не восстанавливает права сам. Orchestrator маршрутизирует, но не решает спор чтением кода: spec-вопрос → Specification Engineer, дизайн → Architect, test oracle → Test Engineer, корректность → Reviewer. Reviewer не является автором candidate; новый контекст и отдельная identity обязательны. Другая модель полезна для разнообразия, но сама по себе независимость не доказывает.
