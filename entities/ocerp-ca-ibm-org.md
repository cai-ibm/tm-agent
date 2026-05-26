# ocerp.ca-ibm.org

**Статус:** УДАЛЁН 26.05.2026

**Сервер:** 192.168.2.34 (cai@192.168.2.34)

## Что было удалено

- Docker контейнеры: `openconstructionerp-app-1`, `openconstructionerp-postgres-1`
- Docker volumes: `app_data`, `pg_data`
- Docker network: `openconstructionerp_default`
- Nginx конфиг: `/etc/nginx/sites-available/openconstructionerp` (symlink из `sites-enabled`)
- Let's Encrypt сертификат для `ocerp.ca-ibm.org` (live, archive, renewal)
- Репозиторий: `/home/cai/OpenConstructionERP/`
- Перезагружен nginx

## История

- v2.9.0 → переключён на **v4.12.0** в день удаления (26.05.2026), но сборка не завершилась
- Причина удаления — больше не нужен
