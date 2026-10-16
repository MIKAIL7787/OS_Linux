### Задача 1. Определение знака числа

```bash
#!/bin/bash

read -p "Введите число: " num

if [ "$num" -gt 0 ]; then
  echo "Число положительное"
elif [ "$num" -lt 0 ]; then
  echo "Число отрицательное"
else
  echo "Число равно нулю"
fi
```

### Задача 2. Проверка файла

```bash
#!/bin/bash

read -p "Введите имя файла: " file

if [ ! -e "$file" ]; then
  echo "Файл не существует"
else
  echo "Файл существует"
  if [ -d "$file" ]; then
    echo "Это директория"
  else
    echo "Это обычный файл"
  fi
  if [ -r "$file" ]; then
    echo "Есть право на чтение"
  else
    echo "Нет права на чтение"
  fi
fi
```

### Задача 3. Оценка через case

```bash
#!/bin/bash

read -p "Введите оценку (1-5): " grade

case "$grade" in
  5) echo "Отлично" ;;
  4) echo "Хорошо" ;;
  3) echo "Удовлетворительно" ;;
  1|2) echo "Плохо" ;;
  *) echo "Неверная оценка" ;;
esac
```

### Задача 4. Високосный год

```bash
#!/bin/bash

read -p "Введите год: " year

if (( (year % 4 == 0 && year % 100 != 0) || year % 400 == 0 )); then
  echo "$year — високосный год"
else
  echo "$year — не високосный год"
fi
```

---

## Циклы

### Задача 5. Числа от 1 до 100, делящиеся на 3 или 5

```bash
#!/bin/bash

for ((i=1; i<=100; i++)); do
  if (( i % 3 == 0 || i % 5 == 0 )); then
    echo -n "$i "
  fi
done
echo
```

### Задача 6. Таблица умножения

```bash
#!/bin/bash

read -p "Введите число (1-10): " n

for ((i=1; i<=10; i++)); do
  echo "$n x $i = $((n * i))"
done
```

### Задача 7. Сумма чисел от 1 до N

```bash
#!/bin/bash

read -p "Введите N: " n
sum=0

for ((i=1; i<=n; i++)); do
  sum=$((sum + i))
done

echo "Сумма чисел от 1 до $n = $sum"
```

### Задача 8. Файлы по размеру

```bash
#!/bin/bash

ls -lS --block-size=1 | awk 'NR>1 {printf "%10d  %s\n", $5, $NF}'
```

Или более явно с `find`:

```bash
#!/bin/bash

find . -maxdepth 1 -type f -printf "%s\t%p\n" | sort -rn | while read size path; do
  echo "$(numfmt --to=iec $size)  $path"
done
```

### Задача 9. Пароль с 3 попытками

```bash
#!/bin/bash

correct="secret"
attempts=3

for ((i=1; i<=attempts; i++)); do
  read -s -p "Введите пароль (попытка $i из $attempts): " pass
  echo
  if [ "$pass" == "$correct" ]; then
    echo "Доступ разрешён"
    exit 0
  fi
done

echo "Доступ запрещён: попытки исчерпаны"
exit 1
```

### Задача 10. Ёлочка из звёздочек

```bash
#!/bin/bash

read -p "Введите высоту ёлочки: " n

for ((i=1; i<=n; i++)); do
  for ((j=1; j<=i; j++)); do
    echo -n "* "
  done
  echo
done
```

---

## Функции

### Задача 11. Функция is_even

```bash
#!/bin/bash

is_even() {
  local num=$1
  if (( num % 2 == 0 )); then
    return 0
  else
    return 1
  fi
}

for ((i=1; i<=20; i++)); do
  if is_even "$i"; then
    echo -n "$i "
  fi
done
echo
```

### Задача 12. Функция max_of_three

```bash
#!/bin/bash

max_of_three() {
  local a=$1 b=$2 c=$3
  local max=$a
  [ "$b" -gt "$max" ] && max=$b
  [ "$c" -gt "$max" ] && max=$c
  echo "$max"
}

max_of_three 10 25 7
```

### Задача 13. Факториал через рекурсию

```bash
#!/bin/bash

factorial() {
  local n=$1
  if (( n <= 1 )); then
    echo 1
  else
    local prev
    prev=$(factorial $((n - 1)))
    echo $((n * prev))
  fi
}

read -p "Введите число: " num
echo "Факториал $num = $(factorial "$num")"
```

### Задача 14. Функция count_files

```bash
#!/bin/bash

count_files() {
  local dir="$1"
  if [ ! -d "$dir" ]; then
    echo "Ошибка: $dir не директория" >&2
    return 1
  fi
  local count
  count=$(find "$dir" -maxdepth 1 -type f | wc -l)
  echo "$count"
}

read -p "Введите путь к директории: " path
echo "Количество файлов: $(count_files "$path")"
```

### Задача 15. Функция check_host

```bash
#!/bin/bash

check_host() {
  local host=$1
  if ping -c 1 -W 1 "$host" > /dev/null 2>&1; then
    echo "$host — доступен"
  else
    echo "$host — недоступен"
  fi
}

hosts=("8.8.8.8" "1.1.1.1" "ya.ru" "localhost" "192.0.2.1")

for h in "${hosts[@]}"; do
  check_host "$h"
done
