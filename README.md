
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

- [The Record — ATB cyberattack](https://therecord.media/atb-ukraine-cyberattack-ransomware)
- [Mezha.ua — ATB hacked](https://mezha.ua/en/news/atb-website-hacked-315842/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
