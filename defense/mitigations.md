# Mitigations

## 1. Hardcoded credentials у APK

Секрет не повинен бути частиною коду. Замість статичного рядка —
токен який додаток отримує з сервера при запуску і який живе обмежений час:

```java
// ❌ пароль вшитий у код, видно після декомпіляції
String auth = "Basic cmVnX3VzZXI6...";

// ✅ токен отримується з бекенду, дійсний обмежений час
String token = authService.getShortLivedToken(deviceId, certificate);
```

Автоматична перевірка на секрети при кожному коміті:

```bash
gitleaks detect --source=.
```

---

## 2. Unauthenticated Moodle endpoint

Кожна функція яка повертає дані повинна спочатку перевірити
хто робить запит і чи має він право. В Moodle для цього є вбудовані функції:

```php
public function get_posts() {
    require_login();                                      // перевірка сесії
    require_capability('block/moco_news:view', $context); // перевірка права
    return json_encode(['posts' => $this->fetch_posts()]); // тільки потрібні дані, не $CFG
}
```

---

## 3. SQL Injection

Проблема: ключ масиву з URL потрапляв напряму в SQL.
Рішення: приймати тільки конкретний список дозволених значень:

```php
// ❌ рядок з URL йде в SQL без перевірки
$sql = "WHERE filter_id = " . key($_GET['filter'][8]);

// ✅ whitelist — якщо значення не в списку, запит не виконується
$allowed = [490, 491, 492];
$key = (int) key($_GET['filter'][8]);
if (!in_array($key, $allowed, true)) throw new InvalidArgumentException();
```

---

## 4. Реєстрація без модерації

Два рядки в конфізі SuiteCRM вмикають обов'язкове підтвердження:

```php
'require_admin_approval' => true,      // акаунт неактивний до підтвердження адміном
'email_verification_required' => true, // спочатку підтвердити email
```

Rate limit на nginx — не більше 3 реєстрацій з одного IP на годину:

```nginx
limit_req_zone $binary_remote_addr zone=reg:10m rate=3r/h;
```

---

## 5. LFI

PHP читав будь-який файл на диску бо не перевіряв шлях.
Рішення: переконатись що шлях веде тільки в дозволену папку:

```php
$uploadDir = realpath('/var/www/uploads/');
$filePath  = realpath($uploadDir . '/' . basename($filename));
// basename() прибирає ../ і не дає вийти за межі папки

if (!str_starts_with($filePath, $uploadDir))
    throw new SecurityException('Path traversal'); // спроба вийти з папки

if (!in_array(pathinfo($filePath, PATHINFO_EXTENSION), ['csv', 'xlsx']))
    throw new SecurityException('Invalid extension'); // тільки дозволені типи
```

На рівні PHP — обмежити до яких папок PHP взагалі може звертатись:

```ini
open_basedir = /var/www/uploads/:/tmp/
```

---

## 6. PHAR Deserialization

Найпростіший захист — повністю відключити phar:// обгортач.
Тоді навіть якщо хтось передасть phar:// шлях, PHP його не обробить:

```php
stream_wrapper_unregister('phar'); // phar:// більше не існує для PHP
```

Якщо unserialize потрібен — обмежити які класи можна відтворювати.
`false` означає що об'єкти взагалі не дозволені, тільки базові типи:

```php
$obj = unserialize($data, ['allowed_classes' => false]);
```

Заборонити виконання PHP у папці завантажень на рівні nginx —
навіть якщо файл туди потрапив, сервер не запустить його як PHP:

```nginx
location ~* /upload/.*\.php$ { deny all; }
```

---

## 7. WAF Body Padding Bypass

WAF перевіряв тільки перші 180 КБ тіла запиту.
ModSecurity дозволяє збільшити цей ліміт до розумного розміру:
ModSecurity дозволяє збільшити цей ліміт до розумного розміру:

SecRequestBodyLimit 10485760 # максимум 10 МБ на запит
SecRequestBodyNoFilesLimit 10485760 # те саме для запитів без файлів


---

## 8. Grafana → Zabbix Write Access

Grafana підключалась до Zabbix БД з правами на запис — це зайве.
Для читання графіків потрібен тільки SELECT:

```sql
CREATE USER 'grafana_ro' IDENTIFIED BY 'унікальний-пароль';
GRANT SELECT ON zabbix.* TO 'grafana_ro';
-- тепер Grafana може тільки читати, вставити фейкову сесію — не може
```

---

## 9. Zabbix Agent від root

Zabbix агент працював від root — тому виконання команди через нього
одразу давало root на хості. Окремий непривілейований користувач вирішує це:

```ini
User=zabbix                 # агент запускається від непривілейованого юзера
DenyKey=system.run[*]       # заборонити виконання довільних команд через агента
```

---

## 10. SSH ключ у бекапі

Приватні SSH-ключі не повинні потрапляти в архів бекапу.
`--exclude` виключає папки з ключами при створенні архіву:

```bash
tar --exclude='root/.ssh' --exclude='home/*/.ssh' \
    -czf backup.tar.gz /etc /var/www
```

Зашифрувати весь архів через `age` — навіть якщо хтось дістане файл,
без ключа шифрування він побачить тільки випадковий набір байтів:

```bash
tar -czf - /data | age -r $(cat pubkey.txt) > backup.age
```
