# StingrayPL
## Stingray is a conditionally statically typed programming language written in AtomVM for writing efficient servers on small devices like ESP32, etc
# Stingray Compiler

Язык программирования Stingray (`.sr`) → Erlang BEAM байткод.

## Структура

```
StingrayCompiler/
├── src/                          # Ядро компилятора
│   ├── stingray_tokens.erl       # Типы токенов и ключевые слова
│   ├── stingray_lexer.erl        # Лексический анализатор (binary matching)
│   ├── stingray_ast.erl          # Конструкторы AST-узлов
│   ├── stingray_parser.erl       # Рекурсивный парсер (precedence climbing)
│   ├── stingray_codegen.erl      # Кодогенератор: AST → Erlang forms → .beam
│   └── stingray_runtime.erl      # Runtime-хелперы (#flow, str_append)
│
├── examples/                     # Примеры и демонстрации
│   ├── hello_world.sr            # #flow:300# — 300 параллельных потоков
│   ├── full_example.sr           # Все фичи языка
│   ├── struct_and_enum.sr        # Структуры и перечисления
│   ├── struct_test.sr            # Структуры в функциях
│   ├── enum_test.sr              # Перечисления
│   ├── if_else_test.sr           # if/else, логика, сравнения
│   ├── while_test.sr             # Циклы с внешними переменными
│   ├── list_test.sr              # Массивы: [], [i], .length
│   ├── string_test.sr            # Сложение строк
│   ├── push_pop_test.sr          # push/pop для массивов
│   ├── all_features_test.sr      # Все фичи: fib, i++, list.length, not
│   ├── chat_server.erl           # TCP чат-сервер (localhost:9999)
│   ├── chat_client.erl           # TCP чат-клиент
│   └── chat_test.escript         # Автотест сервера
│
├── tests/
│   └── run_tests.erl             # Запуск всех 9 тестов
│
├── vscode-stingray/              # Расширение VS Code
│   ├── icon.png                  # Логотип Stingray
│   ├── package.json              # Манифест расширения
│   ├── src/extension.ts          # Команда F5 запуск
│   └── syntaxes/                 # TextMate грамматика
│
├── test.erl                      # Escript для компиляции .sr файлов
└── stingray-lang-0.2.0.vsix      # Готовое расширение для VS Code
```

## Быстрый старт

### 1. Компиляция модулей

```bash
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass

Remove-Item src\*.beam -Force

erlc -o src src/stingray_tokens.erl
erlc -o src src/stingray_lexer.erl
erlc -o src src/stingray_ast.erl
erlc -o src src/stingray_parser.erl
erlc -o src src/stingray_codegen.erl
erlc -o src src/stingray_runtime.erl
```

### 2. Запуск тестов

```bash
erlc -o tests tests/run_tests.erl
erl -pa src -pa tests -noshell -eval "run_tests:run(), halt()."
```

Результат:
```
========================================
         STINGRAY COMPILER TESTS
========================================

--- hello_world ---
Hello World × 300 (параллельно)

--- struct_and_enum ---
421020

--- if_else_test ---
x is positivezeroboth trueat least one truenot false = truex equals 42x is not zerox >= 42x <= 42always truepositivenon-positive

--- full_example ---
"Alice"137303.3333333333333335"Alice"x is bigconditions worknot worksgender is WO
MENmediumparallel hello × 3

--- enum_test ---
color is REDdirection is NORTHnot bluefavorite is GREEN

--- struct_test ---
1020301"Alice"redblue"Charlie"

--- list_test ---
13[]["Alice","Bob","Charlie"][1,2][3,4]1030

--- while_test ---
5"hello""hello""hello"42

--- all_features ---
1365551050not works03[3,4]

========================================
  Total: 9  Passed: 9  Failed: 0
========================================
```

### 3. Компиляция .sr файла

```bash
escript test.erl --compile examples/hello_world.sr
escript test.erl --compile examples/full_example.sr
```

## Синтаксис языка Stingray

### Функции

```stingray
fun main() {
    io.write("Hello World")
}

fun add(a: Int32, b: Int32) {
    a + b
}
```

### Переменные

```stingray
x: Int32 = 42              // типизированная
y = 3.14                   // тип выводится автоматически
name = "Stingray"
flag = true
```

### Строки

```stingray
a = "Hello"
b = " World"
c = a + b                  // "Hello World"
x = "Count: " + 42         // "Count: 42"
```

### if/else

```stingray
if x > 0 {
    io.write("positive")
} else {
    if x == 0 {
        io.write("zero")
    } else {
        io.write("negative")
    }
}
```

### Логические операторы

```stingray
if a > 0 && b < 10 || c == 42 {
    io.write("условия")
}

if not flag {
    io.write("не true")
}
```

### Циклы while

```stingray
i = 0
while i < 10 {
    i = i + 1
}
```

### Перечисления (enum)

```stingray
enum color {
    RED
    GREEN
    BLUE
}

c = color.RED
if c == color.RED { io.write("красный") }
if c != color.BLUE { io.write("не синий") }
```

### Структуры (struct)

```stingray
type struct Point {
    Int32 x,
    Int32 y
}

p = Point.new(10, 20)
io.write(p.x)              // 10
io.write(p.y)              // 20

// Структура с enum полем
type struct User {
    Int32 id,
    String name,
    color favColor
}
u = User.new(1, "Alice", color.RED)
io.write(u.name)           // "Alice"
```

### Массивы (List)

```stingray
nums = [1, 2, 3]           // создание
empty = []                  // пустой список
matrix = [[1, 2], [3, 4]]  // вложенный

io.write(nums[0])           // 1 — доступ по индексу (0-based)
io.write(nums.length)       // 3 — длина

nums2 = nums.push(4)       // [1,2,3,4] — добавление
last = nums2.pop()          // 4 — извлечение последнего
```

### Параллельное выполнение

```stingray
#sideway# func()           // async, fire-and-forget
#flow:300# func()          // 300 копий параллельно + ожидание завершения всех
```

### return

```stingray
fun fib(n: Int32) {
    if n <= 1 {
        n
    } else {
        fib(n - 1) + fib(n - 2)
    }
}
```

### Операторы

| Оператор | Описание |
|----------|----------|
| `+` `-` `*` `/` | Арифметика + сложение строк |
| `==` `!=` `<` `>` `<=` `>=` | Сравнение |
| `&&` `\|\|` `!` / `not` | Логические (короткое замыкание) |
| `.` | Доступ к полю / модулю |
| `[i]` | Доступ по индексу массива |
| `.length` | Длина массива |
| `.push(item)` | Добавление в конец |
| `.pop()` | Извлечение последнего |

### Директивы

```stingray
#use Math as math          // импорт модуля
#sideway# func()           // async
#flow:N# func()            // N копий параллельно
```

### Типы данных

| Тип | Описание |
|-----|----------|
| `Int8`–`Int128` | Целые числа |
| `Float8`–`Float128` | Числа с плавающей точкой |
| `String` | Строка |
| `Char` | Символ (ASCII) |
| `Bool` | Булево значение |
| `List` | Массив |
| `Set` | Множество |

## Пайплайн компиляции

```
.sr файл
  → Лексер (stingray_lexer)      — бинарный паттерн-матчинг
  → Парсер (stingray_parser)     — рекурсивный descent + precedence climbing
  → AST (stingray_ast)           — абстрактное синтаксическое дерево
  → Кодогенератор (stingray_codegen) — AST → Erlang abstract forms
  → stingray_runtime             — runtime-хелперы (#flow, str_append)
  → compile:forms                — Erlang compiler → .beam байткод
```

## Чат-сервер и клиент

TCP-сервер на Erlang VM (localhost:9999) с авто-генерацией имён (`user_1`, `user_2`, ...).

### Запуск сервера

```bash
erl -pa examples -noshell -eval "chat_server:start(), halt()."
```

Сервер запускается и слушает `localhost:9999`. Каждое подключение автоматически получает имя `user_N`.

### Запуск клиента (в другом терминале)

```bash
escript examples/chat_client.erl
```

Клиент подключается к серверу, получает приветствие с именем, и может:
- Писать сообщения — они отправляются всем подключённым пользователям
- `/list` — показать список онлайн-пользователей
- `/quit` — отключиться

### Пример сессии

**Терминал 1 (сервер):**
```
=== Stingray Chat Server ===
Listening on localhost:9999
Commands: /quit, /list

Server started.
[+] user_1 connected
[-] user_1 disconnected
[+] user_2 connected
[user_2]: Привет всем!
[-] user_2 disconnected
```

**Терминал 2 (клиент 1):**
```
=== Stingray Chat Client ===
Connecting to localhost:9999...
Connected!
Welcome, user_1!
> Привет всем!
> /list
Online: user_1, user_2
> /quit
Goodbye!
Disconnected.
```

**Терминал 3 (клиент 2):**
```
=== Stingray Chat Client ===
Connecting to localhost:9999...
Connected!
Welcome, user_2!
[user_1]: Привет всем!
> Привет, user_1!
> /quit
Goodbye!
Disconnected.
```

### Автотест (проверка работоспособности сервера)

```bash
escript examples/chat_test.escript
```

```
=== Chat Test ===
Connected!
Server: Welcome, user_1!
Message sent.
Test complete!
```

### Протокол

| Команда клиента | Описание |
|-----------------|----------|
| `любой текст` | Отправить сообщение всем |
| `/list` | Список онлайн-пользователей |
| `/quit` | Отключение |

## VS Code расширение

### Установка

```bash
code --install-extension stingray-lang-0.2.0.vsix
```

### Возможности

- Подсветка синтаксиса `.sr` файлов
- Автодополнение скобок
- Запуск текущего файла по F5
- Авто-обнаружение компилятора

### Настройка

В `settings.json`:
```json
{
    "stingray.compilerPath": "C:/path/to/StingrayCompiler"
}
```

Если расширение находится в папке проекта, путь определяется автоматически.

### Пересборка

```bash
cd vscode-stingray
npm install
npm run compile
npx @vscode/vsce package
```

## Пулейки кодогенератора

### Enum

```
enum color { RED GREEN BLUE }
c = color.RED
```
→ Erlang: `color = {red, green, blue}. c = element(1, color).`

### Struct

```
type struct Point { Int32 x, Int32 y }
p = Point.new(10, 20)
```
→ Erlang: `p = {point, 10, 20}.`

### Доступ к полю

```
p.x
```
→ Erlang: `element(2, p).` (tag = element 1, первое поле = element 2)

### Цикл while

```
i = 0
while i < 5 { i = i + 1 }
```
→ Erlang: именованная функция `fun Loop(N) -> case N < 5 of true -> Loop(N+1); false -> ok end end(0)`

### Параллельное выполнение

```
#flow:300# io.write("Hello")
```
→ Erlang: `stingray_runtime:flow_run(300, fun() -> io:format("~ts", [["Hello"]]) end)`

### Сложение строк

```
"Hello" + " World"
```
→ Erlang: `stingray_runtime:str_append("Hello", " World")`

## Зависимости

- **Erlang/OTP 27+** — для компиляции и `compile:forms`
- **Node.js / npm** — только для упаковки VSIX (опционально)
