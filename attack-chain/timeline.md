# Timeline

## Фаза 1 — Розвідка та початковий доступ

**День 1-2 — Recon**
- Декомпіляція APK через `jadx` (3 версії)
- Знайдено hardcoded Basic Auth для API
- Сканування субдоменів → `education.atbmarket.com`
- POST без авторизації → витік конфігу Moodle (DB pass, SMTP pass, salt)

**День 2-3 — SQL Injection**
- SQLi у `filter[]` параметрі каталогу
- Boolean blind: version(), database(), user() підтверджено
- `COUNT(users)` → 7 876 914 рядків
- Повна виємка неможлива (99 років) → шукають інший канал

**День 3 — Supplier Portal**
- Реєстрація без модерації на SuiteCRM
- Password reset через temp-mail
- Отримано сесію постачальника

## Фаза 2 — Глибокий доступ

**День 4 — LFI**
- `importFile=config.php` → MySQL credentials
- `importFile=config_override.php` → Oracle EBS PROD, RMS, MEDOC credentials
- `/etc/hostname` → назва сервера підтверджена

**День 5-6 — Webshell**
- PHP PHAR deserialization у SuiteCRM
- PHAR-поліглот замаскований під PNG
- WAF блокує `phar://` → обхід через 200 KB padding
- RCE: `uid=993(nginx)` на `sp-web-p01`

## Фаза 3 — Ескалація

**День 6 — Grafana → Zabbix**
- Grafana: вхід з паролем від Moodle (credential reuse)
- Grafana підключена до Zabbix БД з write-правами
- SQL `INSERT` → фейкова адмін-сесія
- HMAC cookie з session_key (витяг через LFI) → Zabbix admin

**День 6-7 — Root**
- Zabbix API → `script.create` → `uid=0` на `zb-app-p01`
- SSH key-trust → root на `sp-web-p01`
- Squid proxy → обхід файрволу → Jenkins
- CIFS `/mnt/BACKUP` → `root.tar.gz` → `id_rsa_root`
- Один SSH ключ: GitLab, Jenkins, Harbor, CI/CD, Zabbix

## Фаза 4 — Збір даних

**День 7-8**
- MySQL: 7 876 935 клієнтів (38 колонок, bcrypt)
- PostgreSQL: 127 265 співробітників (SUPERUSER)
- Oracle EBS / RMS / MEDOC: підключено
- Active Directory: 68 250 акаунтів
- Exchange: 15 907 листів, 27 активних reset-посилань
- GitLab: 4.77 ГБ, 524 репозиторії
- Splunk: `admin / changeme` — дефолтний пароль

## Фаза 5 — Публічна вимога

**5 жовтня 2026**
- На сайті АТБ → таймер: $400 000 (BTC або USDT), дедлайн 2 години
- АТБ: «даних немає, технічні роботи»
- DataSuckers: опублікували зразки у Telegram
- Сайт відновлено, таймер прибрано

| Фаза | Час |
|---|---|
| Recon + перший вхід | ~2-3 дні |
| LFI + Webshell | +2-3 дні |
| Ескалація до root | +1-2 дні |
| Збір даних | +1-2 дні |
| **Разом** | **~7-10 днів** |
