# add-user-sudo-asb0cj

Добавление пользователя в группу sudo на сервере asb-0cj (192.168.30.12) через SSH.

**Сервер:** `192.168.30.12` (asb-0cj / cai-main-b2-tm)  
**SSH:** `cai-root` / `Cai$2023#`

---

Используется Python + pexpect (интерактивный SSH). Пароль `Cai$2023#` содержит `$` — защищаем single quotes в bash. `sudo -S` с pipe через `printf '%s\n'`. Для установки пароля — `chpasswd` (не `passwd`), вызывается через `sh -c`.

Подробности: `skill_view('add-user-sudo-asb0cj')`
