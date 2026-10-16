## Задача 1. Базовый `$?` и `&&`/`||`

```bash
#!/bin/bash

# 1. Проверка наличия файла через test
test -f /etc/passwd

# 2. Вывод через && и ||
test -f /etc/passwd && echo "Файл есть" || echo "Файла нет"

# 3. Код возврата последней команды
echo "Код возврата: $?"
```

---

## Задача 2. Аргументы и валидация

```bash
#!/bin/bash

# Проверка количества аргументов
if [ $# -lt 2 ]; then
  echo "Использование: $0 <файл> <число>" >&2
  exit 1
fi

# Проверка, что второй аргумент — число
if ! [[ "$2" =~ ^-?[0-9]+$ ]]; then
  exit 2
fi

echo "OK"
```

---

## Задача 3. `set -e` и его ловушка

```bash
#!/bin/bash

set -e

echo "До false"
false
echo "После false — эта строка НЕ выполнится"

# ВАЖНО: if false; then ...; fi НЕ прерывает скрипт при set -e,
# потому что команда false используется в условии if.
# set -e игнорирует ошибки команд, чей код возврата проверяется
# (в условиях if/while/until, в операторах &&, ||, !).
# То есть ошибка "ожидаема" и не считается фатальной.

if false; then
  echo "не выполнится"
fi

echo "Эта строка выполнится, т.к. false стоял в условии if"
```

---

## Задача 4. Strict mode

```bash
#!/bin/bash

set -euo pipefail

trap 'echo "Переменная не задана"' ERR

echo "$UNDEFINED_VAR"
```

При запуске: сработает `trap` на `ERR` (из-за `set -u`), выведется сообщение, и скрипт завершится с ошибкой.

---

## Задача 5. Значения по умолчанию

```bash
#!/bin/bash

greet() {
  local name="${1:-Гость}"
  echo "Привет, $name"
}

# 1. Аргумент не передан
greet                       # Привет, Гость

# 2. Аргумент — пустая строка
greet ""                    # Привет, Гость (т.к. :- срабатывает и на пустую строку)

# Если нужно различать "не передано" и "пустая строка" — используем -
greet2() {
  local name="${1-Гость}"   # без двоеточия: пустая строка останется пустой
  echo "Привет, $name"
}

greet2 ""                   # Привет, (пустое имя)

# 3. Обязательный аргумент через :?
greet3() {
  local name="${1:?Имя обязательно}"
  echo "Привет, $name"
}

greet3 "Иван"               # Привет, Иван
# greet3                    # ошибка: Имя обязательно, и выход
```

---

## Задача 6. `trap` на EXIT и сигналы

```bash
#!/bin/bash

tmpdir=$(mktemp -d)
echo "Создана директория: $tmpdir"

cleanup() {
  rm -rf "$tmpdir"
  echo "Убрано"
}

trap cleanup EXIT INT TERM

echo "Работаю 30 секунд (нажмите Ctrl+C для проверки)..."
sleep 30
echo "Завершено нормально"
```

При нажатии Ctrl+C или при обычном завершении — сработает `cleanup`, директория удалится.

---

## Задача 7. Свой `die()`

```bash
#!/bin/bash

die() {
  echo "ОШИБКА: $*" >&2
  exit 1
}

file="$1"

if [ ! -r "$file" ]; then
  die "нет доступа к $file"
fi

echo "Файл доступен для чтения: $file"
```

---

## Задача 8. `command -v` и зависимости

```bash
#!/bin/bash

missing=()

for cmd in git curl jq; do
  if ! command -v "$cmd" >/dev/null 2>&1; then
    missing+=("$cmd")
  fi
done

if [ ${#missing[@]} -gt 0 ]; then
  echo "Отсутствуют: ${missing[*]}" >&2
  exit 127
fi

echo "Все зависимости на месте"
```

---

## Задача 9. Цикл с обработкой ошибок

```bash
#!/bin/bash

errors=0

for file in "$@"; do
  if [ -e "$file" ]; then
    echo "OK: $file"
  else
    echo "MISSING: $file" >&2
    ((errors++))
  fi
done

if [ "$errors" -gt 0 ]; then
  exit 1
else
  exit 0
fi
```

---

## Задача 10. Отладка через `set -x` и `PS4`

```bash
#!/bin/bash

export PS4='+ ${BASH_SOURCE}:${LINENO}:${FUNCNAME[0]}: '

multiply() {
  local a=$1
  local b=$2
  echo $((a * b))
}

set -x
result=$(multiply 6 7)
set +x

echo "Результат: $result"
```

Пример трассировки:
```
+ script.sh:11:main: result=$(multiply 6 7)
++ script.sh:5:multiply: local a=6
++ script.sh:6:multiply: local b=7
++ script.sh:7:multiply: echo $((a * b))
+ script.sh:11:main: result=42
```

---

## Задача 11. Обработка ошибок в функции

```bash
#!/bin/bash

safe_divide() {
  local a=$1
  local b=$2

  if [ "$b" -eq 0 ]; then
    echo "Ошибка: деление на ноль" >&2
    return 1
  fi

  echo $((a / b))
  return 0
}

# Вызов 1: успешный
result=$(safe_divide 10 2)
code=$?
echo "10 / 2 = $result (код: $code)"

# Вызов 2: ошибка
result=$(safe_divide 10 0)
code=$?
echo "10 / 0 = $result (код: $code)"
```

---

## Задача 12. Игнорирование ошибки точечно

```bash
#!/bin/bash

set -e

echo "Старт"

# Способ 1: через || true
false || true
echo "После false || true — скрипт продолжает работу"

# Способ 2: через if !
if ! false; then
  echo "false упала, но мы это обработали"
fi

echo "Успешная команда"
echo "Готово"

# Разница:
# - `|| true` локально "гасит" ошибку только для этой команды,
#   set -e остаётся активным для всех последующих команд.
# - `set +e` отключает режим errexit глобально — все последующие
#   ошибки тоже перестанут прерывать скрипт, пока не вернёшь set -e.
#   Это опаснее: легко забыть снова включить.
