# 01 — Reconnaissance

## Вектор 1: Hardcoded credentials у APK

### Що зробили

Декомпіляція кількох версій APK через `jadx`:

```bash
jadx -d ./output app-v8.0.16.apk

# Шукаємо секрети у декомпільованому коді
grep -r "Authorization" ./output/
grep -r "Basic " ./output/
grep -r "password" ./output/ --include="*.java" -i
```

### Що знайшли
Захардкоджений `Authorization: Basic` заголовок для API реєстрації.  
Не змінювався між версіями 8.0.16 → 8.0.48.

### Чому спрацювало

| Помилка | Пояснення |
|---|---|
| Hardcoded credentials | Секрет у APK замість backend-токена |
| Без ротації | Пароль не змінювали після релізу |
| APK публічний | Будь-хто може завантажити і декомпілювати |

### Захист
- Ніяких секретів у коді → backend-issued short-lived tokens
- Static analysis у CI: `gitleaks`, `apkleaks`, `MobSF`
- Certificate Pinning

---

## Вектор 2: Unauthenticated Moodle endpoint

### Що зробили

```bash
# Сканування субдоменів
subfinder -d target.com -silent
# → education.target.com

# Відомий Moodle AJAX endpoint
curl -s -X POST https://education.target.com/md/blocks/moco_news/ajax.php \
  -d "procedure=getPosts"
```

### Що знайшли
Відповідь без авторизації: пароль БД, пароль SMTP, salt для хешів, список адмінів.

### Чому спрацювало

| Помилка | Пояснення |
|---|---|
| Немає `require_login()` | Endpoint анонімний |
| Повертає `$CFG` об'єкт | Вся конфігурація в JSON |
| Сторонній плагін без аудиту | `moco_news` не перевірявся |

### Захист
- `require_login()` + `require_capability()` на кожному AJAX endpoint
- Ніколи не повертати конфіг-об'єкти в API
- Аудит сторонніх плагінів
