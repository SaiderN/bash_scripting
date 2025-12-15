## Вложенные условия
### Что такое вложенные условия?
Это когда одно условие находится внутри другого. В Bash это могут быть:

1. `if` внутри `if`

2. `case` внутри `if` 

3. `if` внутри `case`

4. Комбинации циклов и условий

### Базовый синтаксис
```bash
if [[ внешнее_условие ]]; then
    # Внешний блок
    if [[ внутреннее_условие ]]; then
        # Внутренний блок
    fi
fi
```
**Пример 1: case внутри if**
```bash
#!/bin/bash
echo "Hello World"
read -p "Введите имя бренда: " name

if [[ "$name" == "samsung" ]] || [[ "$name" == "nokia" ]]; then 
    case $name in
        samsung)
            echo "norm"
            echo "normicccc"
            ;;
        nokia)
            echo "ok"
            echo "okeeeyyy"
            ;;
    esac
else
    echo "Неизвестный бренд: $name"
fi
```
**Пример 2: if внутри case**
```bash
#!/bin/bash
read -p "Выберите действие (check/scan/report): " action

case "$action" in
    check)
        read -p "Что проверяем? (ports/services/users): " check_type
        
        if [[ "$check_type" == "ports" ]]; then
            echo "Проверяем открытые порты..."
            # netstat -tulpn
        elif [[ "$check_type" == "services" ]]; then
            echo "Проверяем запущенные сервисы..."
            # systemctl list-units --type=service
        elif [[ "$check_type" == "users" ]]; then
            echo "Проверяем активных пользователей..."
            # who
        else
            echo "Неизвестный тип проверки: $check_type"
        fi
        ;;
        
    scan)
        read -p "Сканировать сеть или хосты? (network/hosts): " scan_target
        
        if [[ "$scan_target" == "network" ]]; then
            echo "Сканируем сеть..."
            # nmap 192.168.1.0/24
        elif [[ "$scan_target" == "hosts" ]]; then
            read -p "Введите IP-адрес: " ip
            
            if [[ "$ip" =~ ^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$ ]]; then
                echo "Сканируем хост $ip..."
                # nmap $ip
            else
                echo "Некорректный IP-адрес!"
            fi
        fi
        ;;
        
    report)
        echo "Генерируем отчёт..."
        ;;
        
    *)
        echo "Неизвестное действие: $action"
        ;;
esac
```
**Пример 3: Многоуровневое вложение**
```bash
#!/bin/bash
# Проверка доступа к файлу с логированием

read -p "Введите путь к файлу: " filepath

if [[ -e "$filepath" ]]; then
    echo "Файл существует"
    
    if [[ -f "$filepath" ]]; then
        echo "Это обычный файл"
        
        if [[ -r "$filepath" ]]; then
            echo "Файл доступен для чтения"
            
            # Проверка содержимого
            read -p "Искать в файле (y/n)? " search_choice
            
            if [[ "$search_choice" =~ ^[Yy] ]]; then
                read -p "Введите строку для поиска: " search_string
                
                if grep -q "$search_string" "$filepath"; then
                    echo "Строка найдена в файле!"
                    echo "Количество вхождений: $(grep -c "$search_string" "$filepath")"
                else
                    echo "Строка не найдена"
                fi
            fi
            
        else
            echo "ОШИБКА: Нет прав на чтение файла!"
        fi
        
    elif [[ -d "$filepath" ]]; then
        echo "Это директория"
        echo "Содержимое директории:"
        ls -la "$filepath"
    else
        echo "Это не файл и не директория (возможно симлинк)"
    fi
    
else
    echo "ОШИБКА: Файл/директория не существует!"
fi
```
**Пример 4: Вложенные условия в циклах**
```bash
#!/bin/bash
# Анализ логов на подозрительную активность

log_file="/var/log/auth.log"

if [[ -f "$log_file" && -r "$log_file" ]]; then
    echo "Анализирую $log_file на предмет подозрительных входов..."
    
    while read -r line; do
        # Проверяем FAILED login
        if [[ "$line" == *"Failed password"* ]]; then
            # Извлекаем IP
            ip=$(echo "$line" | grep -oE '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+')
            
            if [[ -n "$ip" ]]; then
                echo "Обнаружена неудачная попытка входа с IP: $ip"
                
                # Проверяем, не внутренний ли это IP
                if [[ "$ip" == 192.168.* || "$ip" == 10.* || "$ip" == 172.16.* ]]; then
                    echo "ВНИМАНИЕ: Внутренний IP ($ip) пытается подобрать пароль!"
                else
                    echo "Внешний IP ($ip) - возможная атака"
                fi
            fi
            
        # Проверяем успешные входы root
        elif [[ "$line" == *"session opened for user root"* ]]; then
            echo "ВНИМАНИЕ: Прямой вход под root обнаружен!"
        fi
        
    done < "$log_file"
    
else
    echo "Не могу прочитать файл логов: $log_file"
    echo "Запустите скрипт с правами root"
fi
```
**Пример 6: Вложенные условия с функциями**
```bash
#!/bin/bash
# Модульная проверка системы

check_ssh_config() {
    local sshd_config="/etc/ssh/sshd_config"
    
    if [[ -f "$sshd_config" ]]; then
        echo "Проверяю конфигурацию SSH..."
        
        # Проверка PermitRootLogin
        if grep -q "^PermitRootLogin yes" "$sshd_config"; then
            echo "  КРИТИЧЕСКИ: Разрешён вход root по SSH!"
            return 1
        elif grep -q "^#PermitRootLogin yes" "$sshd_config"; then
            echo "  ПРЕДУПРЕЖДЕНИЕ: PermitRootLogin закомментирован"
            return 2
        else
            echo "  OK: Root login запрещён"
            return 0
        fi
        
    else
        echo "Файл $sshd_config не найден"
        return 3
    fi
}

check_firewall() {
    if command -v ufw &> /dev/null; then
        ufw_status=$(ufw status 2>/dev/null)
        
        if [[ "$ufw_status" == *"Status: active"* ]]; then
            echo "  OK: UFW включен"
            
            # Проверяем правила
            if ufw status | grep -q "22/tcp.*ALLOW"; then
                echo "  OK: SSH порт открыт"
                return 0
            else
                echo "  ПРЕДУПРЕЖДЕНИЕ: SSH порт закрыт"
                return 1
            fi
            
        else
            echo "  КРИТИЧЕСКИ: UFW выключен!"
            return 2
        fi
        
    elif command -v firewall-cmd &> /dev/null; then
        echo "  Обнаружен firewalld"
        # Проверка firewalld
        return 0
    else
        echo "  КРИТИЧЕСКИ: Фаервол не найден!"
        return 3
    fi
}

# Основная логика
echo "=== ПРОВЕРКА БЕЗОПАСНОСТИ ==="

# Запускаем проверки
check_ssh_config
ssh_result=$?

check_firewall
firewall_result=$?

# Выводим итоговый результат
echo -e "\n=== ИТОГИ ПРОВЕРКИ ==="

if [[ $ssh_result -eq 0 && $firewall_result -eq 0 ]]; then
    echo " Все проверки пройдены успешно"
elif [[ $ssh_result -eq 1 || $firewall_result -eq 2 ]]; then
    echo " КРИТИЧЕСКИЕ ОШИБКИ обнаружены!"
    echo "   Необходимо срочное вмешательство"
else
    echo " Обнаружены предупреждения, рекомендуется проверка"
fi
```
**Советы по вложенным условиям:**
Избегайте слишком глубокой вложенности (больше 3 уровней)

Используйте функции для сложной логики

Комментируйте каждый уровень вложенности

Проверяйте отступы для читаемости

Рассмотрите switch-case для множественных условий
