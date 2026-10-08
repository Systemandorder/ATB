
# ATB Market — Security Case Study

> Educational purposes only.  
> All information is based on publicly reported data.  
> No credentials, IPs, or exploit code targeting live systems are included.

## TL;DR

У жовтні 2026 року хакерська група **DataSuckers** зламала АТБ Маркет.  
Атака пройшла через 10 послідовних кроків — від декомпіляції APK до root на тисячах серверів.

## Масштаб

| Категорія | Кількість |
|---|---|
| Клієнти (телефони, email, хеші паролів) | ~7 900 000 |
| Співробітники (паспортні дані) | 127 265 |
| Постачальники | 50 300 |
| Active Directory акаунти | 68 250 |
| Git-репозиторії | 524 |
| Сервери під контролем | 33 000+ |

## Attack Chain
Recon → Hardcoded creds у APK + незахищений Moodle endpoint
SQL Injection → Boolean blind SQLi у фільтрі каталогу
Supplier Portal → Реєстрація без модерації на SuiteCRM
LFI → Читання конфігів через import функцію
Webshell / RCE → PHP PHAR deserialization + WAF bypass
Escalation → Grafana → Zabbix → root → SSH key → all systems




## References

### Події
- [The Record — ATB cyberattack](https://therecord.media/atb-ukraine-cyberattack-ransomware)
- [Mezha.ua — ATB hacked](https://mezha.ua/en/news/atb-website-hacked-315842/)

### 01 — Розвідка / APK
- [OWASP — Mobile Top 10: Insecure Data Storage](https://owasp.org/www-project-mobile-top-10/)

### 02 — SQL Injection
- [OWASP — Blind SQL Injection](https://owasp.org/www-community/attacks/Blind_SQL_Injection)
- [PortSwigger — Blind SQLi hands-on](https://portswigger.net/web-security/sql-injection/blind)

### 03 — Реєстрація без модерації
- [OWASP — Broken Access Control (A01)](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)

### 04 — LFI / Path Traversal
- [PortSwigger — Path traversal](https://portswigger.net/web-security/file-path-traversal)
- [Acunetix — LFI explained](https://www.acunetix.com/blog/articles/local-file-inclusion-lfi/)

### 05 — PHAR Deserialization + WAF Bypass
- [Pentest-Tools — PHAR deserialization глибоко](https://pentest-tools.com/blog/exploit-phar-deserialization-vulnerability)
- [Keysight — PHAR exploit hands-on](https://blogs.keysight.com/blogs/tech/nwvs.entry.html/2020/07/23/exploiting_php_phar-PRD7.html)
- [Dark Reading — Black Hat 2018: PHAR attack surface](https://www.darkreading.com/application-security/new-php-exploit-chain-highlights-dangers-of-deserialization)

### 06 — Ескалація / Lateral Movement
- [Xygeni — Lateral movement + privilege escalation](https://xygeni.io/blog/lateral-movement-how-privilege-escalation-spreads-in-your-network/)
- [CVE-2022-23131 — Zabbix session forgery (аналогічна техніка)](https://attackerkb.com/topics/cve-2022-23131)

### Загально
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [MITRE ATT&CK — повна матриця технік атак](https://attack.mitre.org/)
