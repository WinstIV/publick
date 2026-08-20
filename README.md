# public

📦 Обновление системы Ubuntu после установки
Обновление пакетов и полная очистка системы сразу (удаление ненужных пакетов и кэша):

sudo apt update && sudo apt upgrade -y && sudo apt autoremove && sudo apt autoclean 
Удалить все старые ядра, кроме используемого и предпоследнего:

apt-get purge $(dpkg -l 'linux-*' | sed '/^ii/!d;/'"$(uname -r | sed "s/\(.*\)-\([^0-9]\+\)/\1/")"'/d;s/^[^ ]* [^ ]* \([^ ]*\).*/\1/;/[0-9]/!d' | head -n -1)
apt autoremove
update-grub
update-grub2
Удалить журналы старше недели:

journalctl --vacuum-time=1weeks

🌍 Проверка GEO IP сервера
Скрипты для определения географического расположения IP сервера:

Вариант 1:

bash <(wget -qO- https://github.com/Davoyan/ipregion/raw/main/ipregion.sh)
Вариант 2:

bash <(wget -qO - https://github.com/vernette/ipregion/raw/master/ipregion.sh)

🛡️ Блокировка двухстороннего пинга (ICMP)

🔹 Мягкий способ (автоматический)
Всё одной командой с созданием бэкапов:

# 1. Создаём бэкапы
sudo cp /etc/ufw/before.rules /etc/ufw/before.rules.bak.$(date +%s)
sudo cp /etc/ufw/before6.rules /etc/ufw/before6.rules.bak.$(date +%s)

# 2. Блокируем ping для IPv4
sudo sed -i '/echo-request/s/ACCEPT/DROP/' /etc/ufw/before.rules

# 3. Блокируем ping для IPv6
sudo sed -i '/echo-request/s/ACCEPT/DROP/' /etc/ufw/before6.rules

# 4. Перезагружаем UFW
sudo ufw reload

# 5. Проверяем результат
echo "=== IPv4 правила ==="
grep "echo-request" /etc/ufw/before.rules
echo "=== IPv6 правила ==="
grep "echo-request" /etc/ufw/before6.rules
🔨 Установка и настройка Fail2ban
Установка:

sudo apt update && sudo apt install fail2ban -y
Применить жёсткий конфиг (бан на 7 дней, 3 неудачные попытки за 3 минуты):

cat > /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
bantime = 7d
findtime = 3m
maxretry = 3
backend = systemd
banaction = iptables-multiport
allowipv6 = auto

[sshd]
enabled = true
mode = aggressive
port = 22,2222
EOF
Запуск и включение автозагрузки:

sudo systemctl enable fail2ban --now
Перезагрузка конфига:

sudo fail2ban-client reload
Проверка статуса и параметров:

fail2ban-client status sshd
fail2ban-client get sshd bantime
fail2ban-client get sshd maxretry
fail2ban-client get sshd findtime
