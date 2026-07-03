# wasender-service

Управление сервисом WhatsApp Sender (wasender) внутри LXC-контейнера на хосте cai-main-b2-tm (192.168.30.12).

## Доступ

Хост: `sobolevrv@192.168.30.12` (пароль: Superp@ss2020sudo)
Контейнер: `wasender` (LXC, Ubuntu 24.04)
Сервис: `wasender.service` (systemd внутри контейнера)
Бот: `/opt/wa-bot/index.js` (Node.js, от пользователя ubuntu)

## Команды

### Перезагрузить wasender
```bash
sshpass -p 'Superp@ss2020sudo' ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null sobolevrv@192.168.30.12 'echo "Superp@ss2020sudo" | sudo -S lxc-attach -n wasender -- systemctl restart wasender'
```

### Статус
```bash
sshpass -p 'Superp@ss2020sudo' ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null sobolevrv@192.168.30.12 'echo "Superp@ss2020sudo" | sudo -S lxc-attach -n wasender -- systemctl status wasender --no-pager -l'
```

### Логи (последние 50 строк)
```bash
sshpass -p 'Superp@ss2020sudo' ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null sobolevrv@192.168.30.12 'echo "Superp@ss2020sudo" | sudo -S lxc-attach -n wasender -- journalctl -u wasender -n 50 --no-pager'
```

### Остановить
```bash
sshpass -p 'Superp@ss2020sudo' ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null sobolevrv@192.168.30.12 'echo "Superp@ss2020sudo" | sudo -S lxc-attach -n wasender -- systemctl stop wasender'
```

### Запустить
```bash
sshpass -p 'Superp@ss2020sudo' ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null sobolevrv@192.168.30.12 'echo "Superp@ss2020sudo" | sudo -S lxc-attach -n wasender -- systemctl start wasender'
```

## Структура бота

- `/opt/wa-bot/index.js` — основной скрипт
- `/opt/wa-bot/.env` — конфигурация
- `/opt/wa-bot/.cred/` — учётные данные сессии WhatsApp
- Пользователь: `ubuntu` (от него запускается сервис)

## Примечания

- SSH-ключа на хосте нет — используется `sshpass` с паролем
- Внутри контейнера нет docker, pm2, supervisor — только systemd
- rclone монтирует `cai.info.tm:services/ok-wa` в `/mnt/ok-wa`
