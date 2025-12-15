## Массивы в Bash

### Что такое массив?
Массив — это структура данных, которая хранит набор элементов (значений) под одним именем. Каждый элемент имеет числовой индекс (начиная с 0).

**Аналогия:** Представь ящик с пронумерованными отделениями. Ящик — это массив, отделения — элементы, номера — индексы.

### Типы массивов в Bash

**1. Индексированные массивы (обычные)**
```bash
# Объявление
fruits=("яблоко" "банан" "апельсин" "виноград")

# Или явно
declare -a colors=("красный" "зелёный" "синий")
```
**2. Ассоциативные массивы (словари, хеш-таблицы)**
```bash
# Требует Bash 4.0+
declare -A users
users["имя"]="Иван"
users["возраст"]="25"
users["город"]="Москва"
```
**Ключевое отличие**: Индексированные массивы используют числа как индексы (0, 1, 2...), ассоциативные — строки как ключи.**
### Объявление массивов
**Способ 1: Прямое присваивание**
```bash
# Пустой массив
empty_array=()

# С элементами
numbers=(1 2 3 4 5)
strings=("hello" "world" "bash")
mixed=(10 "текст" 3.14 "ещё текст")
Способ 2: Через команду declare
bash
declare -a indexed_array        # Индексированный
declare -A associative_array    # Ассоциативный

# Сразу с элементами
declare -a servers=("web1" "web2" "db1" "cache1")
declare -A config=([host]="localhost" [port]="8080" [debug]="true")
```
**Способ 3: Чтение из команды**
```bash
# Файлы в текущей директории
files=(*.txt)

# Строки из файла
mapfile -t lines < file.txt  # или: lines=($(cat file.txt))

# Результат команды
processes=($(ps aux | awk '{print $1}' | uniq))
```
### Доступ к элементам массива
**Индексированные массивы:**
```bash
my_array=("первый" "второй" "третий" "четвёртый")

# Получить элемент по индексу
echo "${my_array[0]}"    # "первый"
echo "${my_array[1]}"    # "второй"
echo "${my_array[-1]}"   # "четвёртый" (последний)
echo "${my_array[-2]}"   # "третий" (предпоследний)

# Получить все элементы
echo "${my_array[@]}"    # все элементы как отдельные слова
echo "${my_array[*]}"    # все элементы как одну строку

# Получить индексы
echo "${!my_array[@]}"   # "0 1 2 3"

# Длина массива
echo "${#my_array[@]}"   # 4 (количество элементов)
echo "${#my_array[0]}"   # 6 (длина первого элемента)
```
**Ассоциативные массивы:**
```bash
declare -A employee
employee=([name]="Алексей" [age]=30 [department]="IT" [salary]=5000)

# Получить по ключу
echo "${employee[name]}"      # "Алексей"
echo "${employee[age]}"       # "30"

# Все ключи
echo "${!employee[@]}"        # "name age department salary"

# Все значения
echo "${employee[@]}"         # "Алексей 30 IT 5000"

# Количество элементов
echo "${#employee[@]}"        # 4
```
### Модификация массивов
**Добавление элементов:**
```bash
# В конец
array=("один" "два")
array+=("три")                # array=("один" "два" "три")
array+=" четыре"              # ВНИМАНИЕ: добавит к последнему элементу!
array+=("четыре" "пять")      # Добавит два элемента

# В начало
array=("два" "три")
array=("один" "${array[@]}")  # array=("один" "два" "три")

# По конкретному индексу
array[5]="шесть"              # Создаст элементы 0-4 пустыми, 5="шесть"
Удаление элементов:
bash
array=("a" "b" "c" "d" "e")

# Удалить элемент по индексу
unset array[2]                # Удаляет "c", но НЕ переиндексирует!
echo "${array[@]}"            # "a b d e"
echo "${!array[@]}"           # "0 1 3 4" (индекс 2 пропал!)

# Переиндексация массива
array=("${array[@]}")         # Теперь индексы: 0 1 2 3

# Удалить весь массив
unset array
```
**Обновление элементов:**
```bash
array=("red" "green" "blue")
array[1]="yellow"             # array=("red" "yellow" "blue")

# Частичная замена в строке
colors=("красный" "зелёный" "синий")
colors[0]="алый"              # Замена целиком
colors[1]="${colors[1]/ел/}"  # Удаление части: "зёный"
```
### Срезы (slices) массивов
```bash
array=("ноль" "один" "два" "три" "четыре" "пять" "шесть")

# Синтаксис: ${массив[@]:начало:длина}
echo "${array[@]:2:3}"        # "два три четыре" (с индекса 2, 3 элемента)
echo "${array[@]: -3:2}"      # "четыре пять" (с 3-го с конца, 2 элемента)
echo "${array[@]:3}"          # "три четыре пять шесть" (с индекса 3 до конца)

# Для одного элемента (подстроки)
element="Hello World"
echo "${element:6:5}"         # "World" (с 6 символа, 5 символов)
```
### Перебор массивов в циклах
**Цикл for по элементам:**
```bash
# Простой перебор
fruits=("яблоко" "банан" "апельсин" "виноград")

for fruit in "${fruits[@]}"; do
    echo "Фрукт: $fruit"
done

# С индексом (если нужно)
for i in "${!fruits[@]}"; do
    echo "Индекс $i: ${fruits[$i]}"
done
Цикл for по диапазону:
bash
# C-стиль
for ((i=0; i<${#fruits[@]}; i++)); do
    echo "$i: ${fruits[$i]}"
done

# С seq
for i in $(seq 0 $((${#fruits[@]} - 1))); do
    echo "$i: ${fruits[$i]}"
done
```
**Цикл while:**
```bash
# С использованием индексов
i=0
while [[ $i -lt ${#fruits[@]} ]]; do
    echo "Элемент $i: ${fruits[$i]}"
    ((i++))
done

# С shift (для аргументов)
args=("$@")
while [[ ${#args[@]} -gt 0 ]]; do
    echo "Обрабатываю: ${args[0]}"
    args=("${args[@]:1}")  # аналог shift
done
```
**Многомерные массивы (эмуляция)**
В Bash нет настоящих многомерных массивов, но можно эмулировать:

```bash
# Способ 1: Индексная арифметика
declare -A matrix
rows=3
cols=3

# Заполнение
for ((i=0; i<rows; i++)); do
    for ((j=0; j<cols; j++)); do
        matrix[$i,$j]=$((i * cols + j))
    done
done

# Чтение
echo "matrix[1,2] = ${matrix[1,2]}"  # 5

# Способ 2: Массивы массивов (Bash 4.3+)
declare -A grid
grid[0]=(1 2 3)
grid[1]=(4 5 6)
grid[2]=(7 8 9)

echo "${grid[1][1]}"  # Ошибка! Так нельзя
# Нужно через временную переменную
row1=(${grid[1]})
echo "${row1[1]}"     # 5
```
**Практические примеры для кибербезопасности**
Пример 1: Сканирование портов
```bash
#!/bin/bash
# Сканирование диапазона портов

target="192.168.1.1"
ports=(21 22 23 25 80 443 3306 3389 8080 8443)

echo "Сканирование хоста: $target"
echo "Проверяемые порты: ${ports[*]}"
echo ""

open_ports=()
closed_ports=()

for port in "${ports[@]}"; do
    # Имитация проверки (в реальности используйте nc, telnet или nmap)
    timeout 1 bash -c "echo >/dev/tcp/$target/$port" 2>/dev/null
    
    if [[ $? -eq 0 ]]; then
        echo " Порт $port: ОТКРЫТ"
        open_ports+=("$port")
    else
        echo " Порт $port: ЗАКРЫТ"
        closed_ports+=("$port")
    fi
done

echo ""
echo "=== ОТЧЕТ ==="
echo "Открытые порты (${#open_ports[@]}): ${open_ports[*]}"
echo "Закрытые порты (${#closed_ports[@]}): ${closed_ports[*]}"

# Анализ опасных портoв
danger_ports=(21 23 3389)  # FTP, Telnet, RDP
for port in "${open_ports[@]}"; do
    for danger in "${danger_ports[@]}"; do
        if [[ $port -eq $danger ]]; then
            echo "⚠️  ВНИМАНИЕ: Опасный порт $port открыт!"
        fi
    done
done
```
Пример 2: Анализ логов
```bash
#!/bin/bash
# Поиск подозрительных IP в логах

log_files=("/var/log/auth.log" "/var/log/secure" "/var/log/syslog")
suspicious_ips=()

for log_file in "${log_files[@]}"; do
    if [[ -f "$log_file" && -r "$log_file" ]]; then
        echo "Анализирую $log_file..."
        
        # Ищем failed login attempts
        while read -r ip; do
            if [[ -n "$ip" ]]; then
                suspicious_ips+=("$ip")
            fi
        done < <(grep -oE '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' "$log_file" | \
                 grep -E "Failed|invalid|authentication failure" | \
                 sort | uniq -c | sort -nr | head -10 | awk '{print $2}')
    fi
done

# Убираем дубликаты
unique_ips=($(printf "%s\n" "${suspicious_ips[@]}" | sort -u))

echo ""
echo "=== ПОДОЗРИТЕЛЬНЫЕ IP АДРЕСА ==="
for ip in "${unique_ips[@]}"; do
    count=0
    for check_ip in "${suspicious_ips[@]}"; do
        [[ "$check_ip" == "$ip" ]] && ((count++))
    done
    echo "IP: $ip (попыток: $count)"
done
```
Пример 3: Управление брандмауэром
```bash
#!/bin/bash
# Динамическое управление правилами фаервола

declare -A firewall_rules
allowed_ips=("192.168.1.100" "192.168.1.101" "10.0.0.5")
ports_to_open=(22 80 443)

# Создаём правила
for port in "${ports_to_open[@]}"; do
    for ip in "${allowed_ips[@]}"; do
        rule_key="${ip}_${port}"
        firewall_rules[$rule_key]="ACCEPT"
    done
done

# Применяем правила (пример для iptables)
echo "Применяю правила фаервола..."
for rule_key in "${!firewall_rules[@]}"; do
    ip=${rule_key%_*}      # До подчёркивания
    port=${rule_key#*_}    # После подчёркивания
    
    echo "iptables -A INPUT -p tcp --dport $port -s $ip -j ${firewall_rules[$rule_key]}"
    # Раскомментируй для реального применения:
    # iptables -A INPUT -p tcp --dport $port -s $ip -j ${firewall_rules[$rule_key]}
done

# Блокировка всего остального
echo "iptables -A INPUT -p tcp --match multiport --dports 22,80,443 -j DROP"
```
Пример 4: Мониторинг процессов
```bash
#!/bin/bash
# Мониторинг подозрительных процессов

dangerous_processes=("cryptominer" "backdoor" "keylogger" "rootkit")
suspicious_procs=()

# Получаем список всех процессов
mapfile -t all_procs < <(ps aux)

for process_line in "${all_procs[@]}"; do
    for danger in "${dangerous_processes[@]}"; do
        if [[ "$process_line" =~ $danger ]]; then
            pid=$(echo "$process_line" | awk '{print $2}')
            cmd=$(echo "$process_line" | awk '{for(i=11;i<=NF;i++) printf $i" "; print ""}')
            suspicious_procs+=("PID:$pid CMD:$cmd")
            break
        fi
    done
done

echo "=== РЕЗУЛЬТАТЫ ПРОВЕРКИ ==="
if [[ ${#suspicious_procs[@]} -eq 0 ]]; then
    echo " Подозрительных процессов не обнаружено"
else
    echo "  Найдено ${#suspicious_procs[@]} подозрительных процессов:"
    for proc in "${suspicious_procs[@]}"; do
        echo "   - $proc"
    done
fi
```
**Списки vs Массивы**
В Bash нет отдельного типа "список" — массивы выполняют эту роль. Однако есть важные отличия от списков в Python:
### Особенности и подводные камни
**Проблемы с пробелами:**
```bash
# НЕПРАВИЛЬНО
items=("file one.txt" "file two.txt")
for item in ${items[@]}; do  # Разобьёт по пробелам!
    echo "$item"
done

# ПРАВИЛЬНО
for item in "${items[@]}"; do  # Кавычки сохраняют пробелы
    echo "$item"
done
```
**Разница между `@ и :`**
```bash
array=("one" "two three" "four")

# "$@" - каждый элемент в кавычках
for item in "${array[@]}"; do
    echo "[$item]"
done
# Вывод: [one] [two three] [four]

# "$*" - все элементы как одна строка
for item in "${array[*]}"; do
    echo "[$item]"
done
# Вывод: [one two three four]
```
**Копирование массивов:**
```bash
original=("a" "b" "c")
copy1=("${original[@]}")      # Правильно - полная копия
copy2=${original[@]}          # Неправильно - потеря структуры
```
**Передача массива в функцию:**
```bash
process_array() {
    local -n arr=$1  # Создаём ссылку (Bash 4.3+)
    echo "Получен массив с ${#arr[@]} элементами"
}

my_array=(1 2 3 4)
process_array my_array
```
### Про кортежи (tuples)
В Bash нет кортежей как отдельного типа. Если нужно неизменяемое множество значений, можно использовать:

Обычный массив и не изменять его

Строку с разделителями: `tuple="значение1:значение2:значение3"`

Использовать readonly для массива: `readonly -a tuple=(1 2 3)`
