---
source_url: hermes://memory/sessions
ingested: 2026-05-15
sha256: bootstrap
---

# Дамп знаний агента — Exchange mail.ca-ibm.org

Извлечено из памяти Hermes и логов сессий 09.05.2026.

## Сервер
- Exchange Server 2019 CU13, build 15.2.1748.036
- Хост: MAIL-SRV1, FQDN: mail.ca-ibm.org
- Локальный IP: 192.168.2.50
- Windows Server 2022 Standard
- 32 ГБ RAM, AMD EPYC 7401P 24 ядра
- Диск C: 199 ГБ (63.6 ГБ своб.), Диск D: 200 ГБ (174.1 ГБ своб.)

## Доступ
- WinRM: cai\sr, NTLM, порт 5985, venv ~/.hermes/venvs/winrm
- Нет прав Organization Management — Exchange cmdlets возвращают Access Denied
- SMTP relay: 192.168.2.50:2526, анонимный, скрипт sendmail.py

## Уязвимости
- CU13 устарел (ProxyShell/ProxyLogon)
- RDP 3389 открыт наружу
- WinRM HTTP открыт
- Нет HSTS, X-Frame-Options, CSP
- /api/v2.0/ доступен без аутентификации

## Инструменты Hermes
- Навык `send-email` — детали SMTP relay
- Навык `remote-command-execution` — WinRM/impacket рецепты
