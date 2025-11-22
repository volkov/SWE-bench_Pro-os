# Human Augmentation в SWE-Bench Pro: Детальное описание процесса

**Этап 4 создания датасета: как профессиональные инженеры улучшают задачи**

---

## 📋 Оглавление

1. [Обзор процесса](#обзор-процесса)
2. [Проблема, которую решает аугментация](#проблема-которую-решает-аугментация)
3. [Три компонента аугментации](#три-компонента-аугментации)
4. [Примеры из датасета](#примеры-из-датасета)
5. [Процесс аугментации](#процесс-аугментации)
6. [Принципы и гайдлайны](#принципы-и-гайдлайны)
7. [Контроль качества](#контроль-качества)

---

## Обзор процесса

> **Цитата из статьи:**
>
> *"In SWE-Bench Pro, human experts organize unstructured commits and issue metadata into two artifacts: a problem statement and a requirements brief with an optional interface. These provide sufficient context to reproduce the gold patch without prescribing an implementation."*

**Human Augmentation** — это четвертый и финальный этап создания датасета SWE-Bench Pro, где **профессиональные инженеры** дополняют и структурируют необработанные коммиты и GitHub issues, чтобы сделать задачи понятными и решаемыми для AI агентов, сохраняя при этом их техническую сложность.

### Ключевые цели:

- ✅ Сделать задачи **понятными** без потери сложности
- ✅ Добавить **недостающий контекст** из коммитов и issues
- ✅ Предотвратить **ложные негативы** из-за underspecified задач
- ✅ Сохранить **реалистичность** real-world проблем
- ✅ Не навязывать конкретную **реализацию**

---

## Проблема, которую решает аугментация

### Почему коммиты и issues недостаточны "как есть"?

> **Цитата из статьи:**
>
> *"Although metadata are collected during commit scraping, commit messages are often unstructured, incomplete, or entirely missing. In practice, issue reproduction and problem solving typically requires extended communication among users, contributors, and codebase maintainers, often including screenshots, links, or other media."*

#### Типичные проблемы исходных данных:

| Проблема | Описание | Пример |
|----------|----------|--------|
| **Неструктурированность** | Commit messages написаны в свободной форме | "fix bug", "update" |
| **Неполнота** | Отсутствует важный контекст | Commit без описания why |
| **Отсутствие информации** | Вообще нет описания | Empty commit message |
| **Неоднозначность** | Можно понять по-разному | "improve performance" |
| **Внешние зависимости** | Ссылки на screenshots, другие issues | "See attached image" |
| **Implicit knowledge** | Предполагается знание проекта | "Fix the usual problem" |

#### Реальный пример из Open Library:

**До аугментации:**
```
Commit message: "enable vCard v4.0 contact import (close #1328)"
Description: [empty]
```

**После аугментации:**
```
Problem Statement:
The application's contact importer recognises vCard 2.1 and 3.0,
but any file that starts with VERSION:4.0 is treated as an
unsupported format.

Requirements:
- The importer must recognize vCard 4.0 format
- Files with VERSION:4.0 should be parsed correctly
- Backward compatibility with v2.1 and v3.0 must be preserved
- [additional context from tests and codebase]

Interface:
[Optional API specifications if needed]
```

### Философия подхода

> **Цитата из статьи:**
>
> *"The goal of augmentation is to equip the SWE agent with sufficient context to resolve the issue without failing due to an underspecified task description."*

**Вместо того чтобы удалять** underspecified задачи (как в других бенчмарках), SWE-Bench Pro:
- ✅ Улучшает их через human expertise
- ✅ Добавляет необходимый контекст
- ✅ Сохраняет оригинальную техническую сложность
- ✅ Делает задачу solvable, но не trivial

---

## Три компонента аугментации

> **Цитата из статьи:**
>
> *"Each task instance in SWE-Bench Pro is complete with human-augmented problem statement, requirements and interface as the task description for the model."*

### 1. Problem Statement (Описание проблемы)

**Что это:**
Переписанная версия оригинального commit message, PR description или GitHub issue в структурированном формате.

> **Цитата из статьи:**
>
> *"The problem statement describes the issue to solve, using content from the original commits, PR and issue, then rewriting it in the style of issues and adding in missing information when necessary."*

**Процесс создания:**
1. Берут исходный текст из commit/PR/issue
2. Переписывают в стиле repository issue templates
3. Добавляют недостающую информацию
4. Убирают неоднозначности
5. Сохраняют технический вызов

**Характеристики:**
- **Длина**: 419 - 8,040 символов
- **Формат**: Структурированный текст в стиле GitHub issues
- **Обязательность**: Всегда присутствует
- **Цель**: Agent должен уметь решить задачу, используя только Problem Statement

**Принципы:**

> **Цитата из статьи:**
>
> *"Agents should be able to solve the task using only the problem statement."*

- Полнота информации для решения
- Ясность без oversimplification
- Реалистичность формулировки
- Отсутствие hints о реализации

---

### 2. Requirements (Требования)

**Что это:**
Новая фича SWE-Bench Pro — структурированный список требований к поведению системы после фикса.

> **Цитата из статьи:**
>
> *"Problems in SWE-Bench Pro can be more complex than previous iterations of SWE-Bench, and thus, requirements are introduced to resolve any potential ambiguity issues by listing out a set of requirements that give additional detail on what is needed to solve the task."*

**Процесс создания:**
1. Анализ unit tests, которые должны пройти
2. Извлечение expected behavior из тестов
3. Формулировка требований без предписывания implementation
4. Проверка grounding в реальных тестах

**Характеристики:**
- **Длина**: 124 - 6,700 символов
- **Формат**: Список требований к поведению
- **Обязательность**: Опционально (может быть null для простых задач)
- **Grounding**: Основаны на unit tests

**Принципы:**

> **Цитата из статьи:**
>
> *"Requirements specify the expected behavior but does not prescribe how the solution should be implemented, preserving the core technical challenge."*

- ✅ **Что** должно работать (expected behavior)
- ❌ **Как** это реализовать (implementation details)
- ✅ Grounded in unit tests
- ❌ Arbitrary требования без тестов

**Философия:**

> **Цитата из статьи:**
>
> *"Human experts produce a clear problem statement and a list of requirements that specify the expected behavior but not how to implement the solution, preserving the core technical challenge."*

Это имитирует **стандартную инженерную практику**, где требования описывают желаемое поведение, но не навязывают конкретную реализацию.

---

### 3. Interface (Интерфейс)

**Что это:**
Спецификация классов, функций и методов, которые ожидают тесты.

> **Цитата из статьи:**
>
> *"For feature additions, engineers documented class and function names expected by the tests to avoid the failure mode when relevant."*

**Зачем нужен:**
Предотвратить ложные негативы, когда решение правильное по логике, но тесты падают из-за:
- Неправильного naming
- Неверной сигнатуры функции
- Отсутствия ожидаемого метода

**Характеристики:**
- **Длина**: 1 - 12,200 символов
- **Формат**: Структурированная спецификация API
- **Обязательность**: Опционально (только для feature additions)
- **Детализация**: Может включать типы, параметры, return values

**Типичные элементы:**
```
Type: [Function/Class/Method]
Name: [имя]
Path: [путь к файлу]
Input: [параметры с типами]
Output: [возвращаемое значение]
Description: [краткое описание]
```

---

## Примеры из датасета

### Пример 1: NodeBB — Email Validation

**Репозиторий:** NodeBB (forum software)
**Язык:** JavaScript
**Тип задачи:** Bug fix

#### Problem Statement:
```
Email Validation Status Not Handled Correctly in ACP and
Confirmation Logic

The Admin Control Panel does not accurately reflect the email
validation status of users. Specifically, when administrators
view user profiles in the ACP, the system fails to indicate
whether a user's email address has pending validation or if
the validation link has expired.
```

#### Requirements:
```
The loadUserInfo(callerUid, uids) function should include logic
to retrieve and attach `email:pending` and `email:expired` flags
to each user object.

When called from the ACP context, the returned user objects must
contain these flags to enable proper UI rendering of validation
status.
```

#### Interface:
```
Type: Method
Name: db.mget
Path: src/database/mongo/main.js

Input:
  keys: string[] (An array of database keys to retrieve)

Output:
  Promise<any[]> (Array of values corresponding to the keys)

Description:
  Retrieves multiple values from the database given an array
  of keys. Returns null for keys that don't exist.
```

#### Fail-to-Pass Tests:
```
test/database.js | Test database
test/database/keys.js::Key methods should return multiple keys
and null if key doesn't exist
```

---

### Пример 2: Qutebrowser — Module Reorganization

**Репозиторий:** Qutebrowser (keyboard-focused browser)
**Язык:** Python
**Тип задачи:** Refactoring

#### Problem Statement:
```
Qt warning filtering tests moved to appropriate module

The `hide_qt_warning` function and related test fixtures have
been moved from `log.py` to `qtlog.py`. This reorganization
better aligns the code structure with its functionality, as
these utilities specifically handle Qt logging concerns.

However, existing tests and code that import these utilities
from the old location will break.
```

#### Requirements:
```
The hide_qt_warning context manager should continue to operate
with identical filtering behavior after the function relocation
from the log module to the qtlog module.

All existing call sites should be updated to import from the
new location (qutebrowser.utils.qtlog) instead of the old
location (qutebrowser.utils.log).

Test fixtures that depend on hide_qt_warning must be updated
to reflect the new import path.
```

#### Interface:
```
Type: Function
Name: hide_qt_warning
Path: qutebrowser/utils/qtlog.py

Input:
  pattern: str (Regex pattern to match Qt warnings to hide)
  logger: str = 'qt' (Logger name, defaults to 'qt')

Output:
  context manager

Description:
  Context manager that temporarily suppresses Qt warnings
  matching the given pattern.
```

---

### Пример 3: NodeBB — WebFinger Implementation

**Репозиторий:** NodeBB
**Язык:** JavaScript
**Тип задачи:** Feature addition

#### Problem Statement:
```
Federated identity discovery via the `.well-known/webfinger`
endpoint is not currently supported.

The WebFinger protocol (RFC 7033) is a standard for discovering
information about people and resources on the internet using
HTTP. Currently, requests to `.well-known/webfinger` result
in a 404 error.

This prevents federation tools and services from discovering
NodeBB user profiles and resources.
```

#### Requirements:
```
A new route should be registered at /.well-known/webfinger
that returns JSON responses conforming to RFC 7033.

The endpoint must accept a 'resource' query parameter
containing the URI to look up (e.g., acct:username@domain).

The output must conform to the WebFinger standard including:
- subject: the requested resource URI
- aliases: alternative URIs for the resource
- links: related links with rel, type, and href properties

For NodeBB users, the response should include links to:
- The user's profile page
- ActivityPub actor endpoint (if available)
- Avatar image URL
```

#### Interface:
```
Type: Route Handler
Path: /.well-known/webfinger
Method: GET

Query Parameters:
  resource: string (Required - URI to look up)

Response Format:
{
  "subject": "acct:user@example.com",
  "aliases": ["https://example.com/user/username"],
  "links": [
    {
      "rel": "self",
      "type": "application/activity+json",
      "href": "https://example.com/user/username"
    },
    {
      "rel": "http://webfinger.net/rel/avatar",
      "href": "https://example.com/assets/avatar.jpg"
    }
  ]
}

Error Response (404):
{
  "error": "Resource not found"
}
```

---

## Процесс аугментации

### Входные данные для инженера

1. **Git Commit**
   - Commit hash
   - Commit message (часто неполное)
   - Code diff
   - Changed files

2. **GitHub Metadata**
   - Associated PR (если есть)
   - Associated Issue (если есть)
   - Comments и discussion
   - Labels и milestones

3. **Test Information**
   - Fail-to-pass tests (новые тесты в коммите)
   - Pass-to-pass tests (существующие тесты)
   - Test execution results

4. **Repository Context**
   - README и документация
   - Issue templates проекта
   - Contributing guidelines
   - Codebase structure

### Workflow инженера

```
┌─────────────────────────────────────────────────────────┐
│ 1. Анализ исходных данных                               │
│    - Читает commit message, PR, issue                   │
│    - Изучает code diff                                  │
│    - Запускает и анализирует тесты                      │
└─────────────────┬───────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────┐
│ 2. Создание Problem Statement                           │
│    - Переписывает в стиле repository issues             │
│    - Добавляет контекст из кодовой базы                 │
│    - Убирает неоднозначности                            │
│    - Проверяет: можно ли решить только по описанию?     │
└─────────────────┬───────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────┐
│ 3. Формулировка Requirements                            │
│    - Анализирует fail-to-pass тесты                     │
│    - Извлекает expected behaviors                       │
│    - Формулирует требования без implementation hints    │
│    - Проверяет grounding в тестах                       │
└─────────────────┬───────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────┐
│ 4. Определение Interface (если нужен)                   │
│    - Смотрит, какие API ожидают тесты                   │
│    - Документирует сигнатуры функций/классов            │
│    - Указывает типы параметров и return values          │
└─────────────────┬───────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────┐
│ 5. Валидация                                            │
│    - Self-review: достаточно ли информации?             │
│    - Проверка: не раскрыта ли реализация?               │
│    - Тест: можно ли решить задачу по этому описанию?    │
└─────────────────┬───────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────┐
│ 6. Peer Review (Human Checkpoint 2 & 3)                 │
│    - Другой инженер проверяет качество                  │
│    - Проверка ясности описаний                          │
│    - Проверка релевантности тестов                      │
│    - Approval или revisions                             │
└─────────────────────────────────────────────────────────┘
```

### Временные затраты

Статья не указывает точное количество человеко-часов, но исходя из:
- 1,865 задач в датасете
- 3 human checkpoints на задачу
- Необходимость экспертизы в коде

**Оценка:** Несколько месяцев работы команды профессиональных инженеров.

---

## Принципы и гайдлайны

### 1. Сохранение технической сложности

> **Цитата из статьи:**
>
> *"[Requirements] specify the expected behavior but does not prescribe how the solution should be implemented, preserving the core technical challenge."*

**Правило:** Никогда не раскрывать implementation details в Problem Statement или Requirements.

**Правильно:**
```
Requirement: "The function must validate email addresses
according to RFC 5322 standard."
```

**Неправильно:**
```
Requirement: "Use the regex pattern /^[a-zA-Z0-9._%+-]+@.../
to validate emails."
```

### 2. Grounding в unit tests

> **Цитата из статьи:**
>
> *"Requirements specifications specify the expected behavior... grounded on the unit tests that are used for validation."*

**Правило:** Каждое требование должно быть verifiable через существующие тесты.

**Процесс:**
1. Читаем fail-to-pass тест
2. Понимаем, что он проверяет
3. Формулируем требование на основе этой проверки

### 3. Достаточность для решения

> **Цитата из статьи:**
>
> *"Agents should be able to solve the task using only the problem statement."*

**Правило:** Problem Statement должен содержать всю необходимую информацию.

**Тест достаточности:**
- Может ли опытный разработчик (без доступа к коду) понять, что нужно сделать?
- Есть ли неоднозначности, которые могут привести к разным решениям?
- Указан ли expected behavior, а не только симптом проблемы?

### 4. Реалистичность формулировки

> **Цитата из статьи:**
>
> *"[Engineers] rewrite it in the style of issues and add in missing information when necessary."*

**Правило:** Сохранять стиль real-world GitHub issues, не делать академические формулировки.

**Хорошо:**
```
"The contact importer crashes when processing vCard 4.0 files.
Users report getting a 'Format not supported' error, but the
file format is valid according to RFC 6350."
```

**Плохо (слишком академично):**
```
"Implement support for vCard version 4.0 as specified in
RFC 6350 section 3.3, ensuring backward compatibility with
versions 2.1 and 3.0 through polymorphic parsing strategies."
```

### 5. Минимизация ложных негативов

> **Цитата из статьи:**
>
> *"For feature additions, engineers documented class and function names expected by the tests to avoid the failure mode when relevant."*

**Правило:** Если тесты ожидают конкретные naming conventions, указать это в Interface.

**Зачем:** Решение может быть технически корректным, но тесты упадут из-за неправильного имени функции.

---

## Контроль качества

### Three-Stage Human Verification

> **Цитата из статьи:**
>
> *"Three human-in-the-loop checkpoints validate: environment construction, issue descriptions, and test relevance."*

#### Checkpoint 1: Environment Construction
- Не относится к augmentation
- Валидация Docker окружения

#### Checkpoint 2: Issue Description Quality
**Что проверяют:**
- ✓ Problem Statement понятен?
- ✓ Достаточно информации для решения?
- ✓ Сохранена реалистичная сложность?
- ✓ Нет oversimplification?
- ✓ Нет hints о реализации?

**Кто проверяет:** Другой инженер (peer review)

#### Checkpoint 3: Test Relevance Validation
**Что проверяют:**
- ✓ Тесты связаны с Problem Statement?
- ✓ Requirements grounded в тестах?
- ✓ Нет слишком широких тестов?
- ✓ Нет ложных срабатываний?
- ✓ Interface соответствует тестовым ожиданиям?

**Кто проверяет:** Инженер-reviewer

### Автоматические проверки

> **Цитата из статьи:**
>
> *"Automatic validation: Running gold tests multiple times to ensure consistency and eliminate flaky tests"*

**Что проверяется автоматически:**
1. Gold patch проходит все тесты
2. Fail-to-pass тесты действительно падают до фикса
3. Pass-to-pass тесты остаются passing
4. Нет flaky tests (одинаковые результаты на нескольких запусках)

### Метрики качества аугментации

**Из датасета:**

| Метрика | Значение |
|---------|----------|
| **Problem Statements** | 100% (все 1,865 задач) |
| **Requirements** | ~80-90% (опциональны для простых задач) |
| **Interface** | ~30-40% (только для feature additions) |
| **Средняя длина Problem Statement** | ~2,500 символов |
| **Средняя длина Requirements** | ~1,500 символов |
| **Средняя длина Interface** | ~2,000 символов |

---

## Отличия от других бенчмарков

### SWE-Bench (оригинальный)

❌ **Нет Requirements** — только problem statement
❌ **Нет Interface** — много ложных негативов
❌ **Меньше контроля качества** — commit messages as-is
❌ **Underspecified задачи** часто удаляются

### SWE-Bench Verified

❌ **Нет Requirements** — только problem statement
✓ **Лучше контроль качества**
❌ **Много trivial задач** (161/500 — one-liners)
❌ **Underspecified задачи удаляются**

### SWE-Bench Pro

✅ **Requirements** — явные behavioral specifications
✅ **Interface** — предотвращение ложных негативов
✅ **Строгий контроль качества** — 3 human checkpoints
✅ **Underspecified задачи улучшаются**, а не удаляются
✅ **Enterprise complexity** — реальные сложные задачи

---

## Выводы

### Философия Human Augmentation в SWE-Bench Pro

**Баланс между двумя крайностями:**

```
Слишком сырые данные          Идеальная середина          Oversimplified
         ↓                              ↓                           ↓
Commit messages as-is      SWE-Bench Pro          Detailed implementation
Underspecified             Augmentation           guides
Много false negatives                             Trivial tasks

         ❌                             ✅                          ❌
```

**Ключевые достижения:**

1. **Clarity without oversimplification**
   - Задачи понятны, но не trivial
   - Добавлен контекст, но не решение

2. **Realism with fairness**
   - Сохранена сложность real-world
   - Предотвращены unfair failures

3. **Behavior, not implementation**
   - Requirements описывают "что", не "как"
   - Preserved technical challenge

4. **Grounded in reality**
   - Все требования основаны на реальных тестах
   - Нет arbitrary specifications

### Влияние на качество бенчмарка

**Результат Human Augmentation:**

- 📊 **Сложность**: Median 107.4 строк в 4.1 файлах
- 📊 **Resolution rate**: 23-44% (vs >70% на других бенчмарках)
- 📊 **False negatives**: Минимизированы через Interface specs
- 📊 **Underspecified tasks**: 0 (все улучшены)
- 📊 **Quality**: 100% human-verified

> **Финальная цитата:**
>
> *"The goal of augmentation is to equip the SWE agent with sufficient context to resolve the issue without failing due to an underspecified task description."*

**Mission accomplished:** SWE-Bench Pro представляет задачи с профессиональным уровнем спецификации, при этом сохраняя реалистичную сложность enterprise software engineering.

---

## Источники

Информация в этом документе основана на:

- **Основная статья**: [SWE-Bench Pro: Can AI Agents Solve Long-Horizon Software Engineering Tasks?](https://arxiv.org/html/2509.16941v1)
- **Dataset**: [ScaleAI/SWE-bench_Pro на HuggingFace](https://huggingface.co/datasets/ScaleAI/SWE-bench_Pro)
- **Leaderboards**:
  - [Public Set](https://scale.com/leaderboard/swe_bench_pro_public)
  - [Commercial Set](https://scale.com/leaderboard/swe_bench_pro_commercial)
- **Blog**: [SWE-Bench Pro: Raising the Bar for Agentic Coding](https://scale.com/blog/swe-bench-pro)

---

**Дата создания документа:** 2025
**Версия датасета:** SWE-Bench Pro v1.0
**Автор документа:** Based on Scale AI Research Team publications
