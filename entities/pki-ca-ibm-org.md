---
title: PKI CA (Root-CA + Issue-CA)
created: 2026-05-19
updated: 2026-05-19
type: entity
tags: [pki, ca, certificates, crl, ocsp, iis]
---

# PKI: Корневой и подчинённый центры сертификации

Двухуровневая иерархия PKI: офлайн-корневой Root-CA и Enterprise подчинённый Issue-CA.

## Серверы

| Параметр | Root-CA | Issue-CA |
|----------|---------|----------|
| **IP** | 192.168.2.33 | 192.168.2.32 |
| **FQDN** | certsrv.ca-ibm.org | issue-ca.ca-ibm.org |
| **Имя ПК** | CAI-Root-CA | issue-ca |
| **Тип** | Standalone Root CA | Enterprise Subordinate CA |
| **ОС** | Windows Server 2022 | Windows Server 2022 |
| **Учётка** | CAI-Root-CA\hermes | cai\hermes |

## Сертификаты CA

### Root-CA (самоподписанный)
- Subject: CN=Root-CA, O=CAI, C=EU
- Thumbprint: 875CFF5DA338A7D59200B43416CD8743F2B811EA
- Действует до: 12.05.2051
- Файлы: `C:\Windows\system32\CertSrv\CertEnroll\Root-CA.crt`, `Root-CA.crl`

### Issue-CA (выдан Root-CA)

| Параметр | Старый | Новый |
|----------|--------|-------|
| **Thumbprint** | A15011C3... | 7ED607860A6474EA4196C10304F07183CA1036E0 |
| **Серийный** | 6E...000003 | 6E...000004 |
| **Действует до** | 12.05.2027 | 19.05.2027 |
| **CDP (CRL Root-CA)** | pki.ca-ibm.org | certsrv.ca-ibm.org |
| **AIA (серт Root-CA)** | pki.ca-ibm.org | certsrv.ca-ibm.org |

## Точки распространения

### Root-CA (certsrv.ca-ibm.org)
- **AIA:** `http://certsrv.ca-ibm.org/aia/` → `C:\Windows\system32\CertSrv\CertEnroll`
- **CDP:** `http://certsrv.ca-ibm.org/crl/` → `C:\Windows\system32\CertSrv\CertEnroll`
- IIS: Default Web Site, порт 80, Directory Browsing включён
- MIME-типы: `.crl` → `application/pkix-crl`, `.crt` → `application/pkix-cert`

### Issue-CA (pki.ca-ibm.org)
- **AIA:** `http://pki.ca-ibm.org/aia/` → `C:\Windows\system32\CertSrv\CertEnroll`
- **CDP:** `http://pki.ca-ibm.org/crl/` → `C:\Windows\system32\CertSrv\CertEnroll`
- **OCSP:** `http://pki.ca-ibm.org/ocsp/` (POST, ISAPI-расширение `ocspisapi.dll`)

## OCSP-ответчик (Issue-CA)

- Служба: OCSPSvc (Automatic, Running)
- Конфигурация: `issue-ca-ocsp` (через `ocsp.msc`)
- Подписывающий сертификат: новый Issue-CA (7ED6...)
- Доступ: только POST с телом OCSP-запроса в DER
- GET-запросы возвращают 500 — норма

## Периоды CRL (Issue-CA)

- Базовый CRL: 7 дней
- Delta CRL: 1 день

## Публикация URL (реестр)

### Root-CA
```
CRLPublicationURLs:
  65:C:\Windows\system32\CertSrv\CertEnroll\%3%8%9.crl
  2:http://certsrv.ca-ibm.org/crl/%3%8%9.crl

CACertPublicationURLs:
  1:C:\Windows\system32\CertSrv\CertEnroll\%1_%3%4.crt
  2:http://certsrv.ca-ibm.org/aia/%1_%3%4.crt
```

### Issue-CA
```
CRLPublicationURLs:
  65:C:\Windows\system32\CertSrv\CertEnroll\%3%8%9.crl
  2:http://pki.ca-ibm.org/crl/%3%8%9.crl

CACertPublicationURLs:
  1:C:\Windows\system32\CertSrv\CertEnroll\%1_%3%4.crt
  2:http://pki.ca-ibm.org/aia/%1_%3%4.crt
  32:http://pki.ca-ibm.org/ocsp
```

## История изменений

- 2026-05-19: Настроен IIS на Root-CA (вирт. директории /aia, /crl)
- 2026-05-19: Обновлены URL CDP/AIA на Root-CA: pki.ca-ibm.org → certsrv.ca-ibm.org
- 2026-05-19: Перевыпущен сертификат Issue-CA (RequestId: 4) с новыми URL
- 2026-05-19: Настроен IIS на Issue-CA (/aia, /crl → CertEnroll)
- 2026-05-19: Настроен OCSP-ответчик на Issue-CA (ocsp.msc → issue-ca-ocsp)

## Связанные страницы

- [[server-94-130-51-161]] — внешний сервер с nginx (возможный reverse proxy для pki)
- [[mail-ca-ibm-org]] — Exchange Server, клиент PKI
