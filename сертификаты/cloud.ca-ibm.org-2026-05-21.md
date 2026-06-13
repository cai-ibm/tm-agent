---
title: cloud.ca-ibm.org — Let's Encrypt сертификат
created: 2026-05-21
updated: 2026-06-13
type: entity
tags: [сертификат, ssl, tls, issue-ca, 3x-ui]
sources: []
---

# cloud.ca-ibm.org — выпуск сертификата (2026-05-21)

## Сервер
- **Хост**: cloud.ca-ibm.org (192.168.40.17)
- **OS**: Ubuntu, hostname x-ui-server
- **Сервис**: 3x-ui (xray-core) на порту 443

## Центр сертификации
- **CA**: Issue-CA (issue-ca.ca-ibm.org, 192.168.2.32)
- **Тип**: Enterprise CA, домен cai
- **Шаблон**: WebServer
- **Учётка**: cai\hermes

## Процедура

### 1. Генерация CSR на cloud.ca-ibm.org
```bash
openssl req -new -newkey rsa:2048 -nodes \
  -keyout /tmp/cloud.ca-ibm.org.key \
  -out /tmp/cloud.ca-ibm.org.csr \
  -subj "/CN=cloud.ca-ibm.org" \
  -addext "subjectAltName=DNS:cloud.ca-ibm.org"
```

### 2. Подписание на Issue-CA через WinRM
CSR скопирован на `C:\temp\cloud.ca-ibm.org.csr` через WinRM PowerShell.
```powershell
certreq -submit -config "issue-ca.ca-ibm.org\Issue-CA" \
  -attrib "CertificateTemplate:WebServer" \
  C:\temp\cloud.ca-ibm.org.csr \
  C:\temp\cloud.ca-ibm.org.crt
```
RequestId: 819. Статус: Issued.

### 3. Установка сертификата на cloud.ca-ibm.org
Файлы:
- `/home/cai/xui.crt` — сертификат (1879 bytes, root:root 600)
- `/home/cai/xui.key` — приватный ключ (1704 bytes, root:root 600)

### 4. Перезапуск 3x-ui
```bash
systemctl restart x-ui
```

## Результат
- **Subject**: CN = cloud.ca-ibm.org
- **Issuer**: C = EU, O = CAI, CN = Issue-CA
- **Valid**: 2026-05-21 → 2027-05-19
- **Service**: active (running), порт 443 LISTEN
- **Modulus**: ключ и сертификат совпадают ✅

## Связанные страницы

[[pki-ca-ibm-org]], [[mail-ca-ibm-org]]
