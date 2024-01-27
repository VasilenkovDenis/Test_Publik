Для настройки сервера на Ubuntu 22 с IP-адресом (твой IP VPS сервра) для обхода блокировок с использованием Bind9 и Sniproxy, выполните следующие шаги. Обратите внимание, что вам потребуются права администратора (root) для выполнения большинства команд.

### 1. Подготовка сервера и установка необходимого программного обеспечения:

1.1. Обновите пакеты и установите необходимые зависимости:
```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y bind9 sniproxy ufw
```

### 2. Настройка UFW (брандмауэра):

2.1. Включите UFW:
```bash
sudo ufw enable
```

2.2. Настройте правила для SSH, DNS и HTTPS:
```bash
sudo ufw allow from [Ваш IP-адрес управления] to any port 22
sudo ufw allow 53
sudo ufw allow 443
```

### 3. Настройка Bind9:

3.1. Настройте основные параметры Bind9. Откройте файл `/etc/bind/named.conf.options` для редактирования:
```bash
sudo nano /etc/bind/named.conf.options
```

3.2. Добавьте или убедитесь, что присутствуют следующие строки:
```bash
options {
    directory "/var/cache/bind";
    dnssec-validation auto;
    listen-on { any; };
    allow-query { any; };
    forwarders {
        8.8.8.8;
        8.8.4.4;
    };
};
```

3.3. Сохраните и закройте файл.

3.4. Настройте зоны DNS. Откройте файл `/etc/bind/named.conf.local` для редактирования:
```bash
sudo nano /etc/bind/named.conf.local
```

3.5. Добавьте конфигурацию зон (пример для одной зоны, повторите для всех нужных):
```bash
zone "xsts.auth.xboxlive.com" {
    type master;
    file "/etc/bind/db.xsts.auth.xboxlive.com";
};
```

3.6. Сохраните и закройте файл.

3.7. Создайте и настройте файлы зон. Для каждой зоны выполните:
```bash
sudo nano /etc/bind/db.xsts.auth.xboxlive.com
```

3.8. Добавьте содержимое (пример для одной зоны, меняйте домен и IP на ваш):
```bash
$TTL 86400
@ IN SOA xsts.auth.xboxlive.com. root.xsts.auth.xboxlive.com. (
    1 ; Serial
    604800 ; Refresh
    86400 ; Retry
    2419200 ; Expire
    86400 ) ; Negative Cache TTL
;
@ IN NS xsts.auth.xboxlive.com.
@ IN A <IP YOUR SERVER VPS>
```

3.9. Сохраните и закройте файл.

3.10. Проверьте конфигурацию и перезапустите Bind9:
```bash
sudo named-checkconf
sudo systemctl restart bind9
```

### 4. Настройка Sniproxy:

4.1. Откройте файл конфигурации Sniproxy для редактирования:
```bash
sudo nano /etc/sniproxy.conf
```

4.2. Добавьте или измените конфигурацию:
```bash
user nobody
pidfile /var/run/sniproxy.pid
listen 443 {
    proto tls
    table hosts
}

table hosts {
    xsts.auth.xboxlive.com *
    k1-api.wbagora.com *
    prod-network-api.wbagora.com *
    account.wbgames.com *
}
```

4.3. Сохраните и закройте файл.

4.4. Перезапустите Sniproxy:
```bash
sudo systemctl restart sniproxy
```

4.5. Для автоматического запуска Sniproxy при старте системы:
```bash
sudo systemctl enable sniproxy
```

После выполнения всех этих шагов ваш сервер должен быть настроен для перенаправления DNS-запросов и HTTPS-трафика через Bind9 и Sniproxy, обеспечивая обход блокировок для указанных доменов. Убедитесь, что все службы работают корректно, и проверьте логи на наличие ошибок.