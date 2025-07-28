# Домашнее задание к занятию «Базы данных в облаке»

## Отчет о создании кластера PostgreSQL в Yandex Cloud

### Задание выполнено успешно ✅

---

## Ответы на вопросы задания

### 1. Создание кластера PostgreSQL

**Параметры кластера:**
- Название: `test-postgres-cluster`
- Класс хоста: `s2.micro`
- Тип диска: `network-ssd`
- Размер диска: `20 ГБ`
- Версия PostgreSQL: `14`
- Окружение: `PRODUCTION`

**Хосты:**
- **Мастер**: `rc1a-ndb35jeuvla17ioi.mdb.yandexcloud.net` (зона `ru-central1-a`)
- **Реплика**: `rc1b-tlr0culo08o6ah6k.mdb.yandexcloud.net` (зона `ru-central1-b`)

Оба хоста имеют публичные IP адреса для внешнего доступа.

**Сеть:**
- Создана сеть: `postgres-network`
- Подсети:
  - `postgres-subnet-a` в зоне `ru-central1-a` (10.1.0.0/24)
  - `postgres-subnet-b` в зоне `ru-central1-b` (10.2.0.0/24)

**База данных и пользователь:**
- База данных: `testdb`
- Пользователь: `testuser`
- Пароль: `TestPassword123!`

### 2. Статус кластера

```
+----------------------+-----------------------+---------------------+--------+---------+
|          ID          |         NAME          |     CREATED AT      | HEALTH | STATUS  |
+----------------------+-----------------------+---------------------+--------+---------+
| c9q6m9isi0ac9f9q4p3r | test-postgres-cluster | 2025-07-28 11:58:17 | ALIVE  | RUNNING |
+----------------------+-----------------------+---------------------+--------+---------+
```

### 3. Информация о хостах

```
+-------------------------------------------+----------------------+---------+--------+---------------+-----------+
|                   NAME                    |      CLUSTER ID      |  ROLE   | HEALTH |    ZONE ID    | PUBLIC IP |
+-------------------------------------------+----------------------+---------+--------+---------------+-----------+
| rc1a-ndb35jeuvla17ioi.mdb.yandexcloud.net | c9q6m9isi0ac9f9q4p3r | MASTER  | ALIVE  | ru-central1-a | true      |
| rc1b-tlr0culo08o6ah6k.mdb.yandexcloud.net | c9q6m9isi0ac9f9q4p3r | REPLICA | ALIVE  | ru-central1-b | true      |
+-------------------------------------------+----------------------+---------+--------+---------------+-----------+
```

### 4. Тестирование подключения к мастеру

**Проверка роли узла:**
```sql
SELECT case when pg_is_in_recovery() then 'REPLICA' else 'MASTER' end;
```
**Результат:** `MASTER` ✅

**Количество подключенных реплик:**
```sql
SELECT count(*) from pg_stat_replication;
```
**Результат:** `1` ✅

**Создание тестовой таблицы:**
```sql
CREATE TABLE test_table(text varchar);
INSERT INTO test_table VALUES('Строка 1');
```
**Результат:** `INSERT 0 1` ✅

### 5. Тестирование подключения к реплике

**Проверка роли узла:**
```sql
SELECT case when pg_is_in_recovery() then 'REPLICA' else 'MASTER' end;
```
**Результат:** `REPLICA` ✅

**Проверка репликации данных:**
```sql
SELECT * FROM test_table;
```
**Результат:**
```
   text   
----------
 Строка 1
(1 row)
```
✅ **Репликация работает корректно!**

### 6. Команды для подключения

**Подключение к мастеру (для записи):**
```bash
docker run --rm -v "${PWD}:/workspace" -w /workspace -e PGPASSWORD=TestPassword123! postgres:14 psql \
  "host=rc1a-ndb35jeuvla17ioi.mdb.yandexcloud.net,rc1b-tlr0culo08o6ah6k.mdb.yandexcloud.net \
   port=6432 sslmode=require sslrootcert=root.crt dbname=testdb user=testuser \
   target_session_attrs=read-write"
```

**Подключение к реплике (только для чтения):**
```bash
docker run --rm -v "${PWD}:/workspace" -w /workspace -e PGPASSWORD=TestPassword123! postgres:14 psql \
  "host=rc1b-tlr0culo08o6ah6k.mdb.yandexcloud.net \
   port=6432 sslmode=require sslrootcert=root.crt dbname=testdb user=testuser"
```

### 7. Ссылки

- **Консоль кластера:** https://console.cloud.yandex.ru/folders/b1gnpj43l8c5f3hvtphl/managed-postgresql/cluster/c9q6m9isi0ac9f9q4p3r
- **SSL сертификат:** Скачан как `root.crt`

### 8. Результаты выполнения задания

#### Требуемые скриншоты:

**1) Созданная база данных:**
- Кластер PostgreSQL создан с именем `test-postgres-cluster`
- Статус: `RUNNING`
- Класс хоста: `s2.micro`
- Диск: `network-ssd` (20 ГБ)
- Два хоста в разных зонах доступности (`ru-central1-a` и `ru-central1-b`)
- Публичные IP адреса настроены для обоих хостов

**2) Результат вывода команды на реплике `SELECT * FROM test_table;`:**
```
   text
----------
 Строка 1
(1 row)
```

#### Проверка выполнения всех требований:

✅ **Создание кластера PostgreSQL:**
- Класс хоста: s2.micro
- Диск: network-ssd
- Два хоста в разных зонах доступности
- Публичные IP адреса для хостов
- Пользователь и база данных созданы

✅ **Подключение к мастеру и реплике:**
- SSL-сертификат скачан
- Подключение к мастеру с `target_session_attrs=read-write` работает
- Проверена роль узла: `MASTER`
- Количество реплик: `1`

✅ **Проверка работоспособности репликации:**
- Таблица `test_table` создана на мастере
- Данные вставлены: `'Строка 1'`
- Подключение к реплике выполнено успешно
- Роль узла подтверждена: `REPLICA`
- Данные успешно реплицированы и доступны на реплике

### 9. Заключение

Кластер PostgreSQL успешно создан и настроен в соответствии с требованиями задания. Все функции репликации работают корректно, данные синхронизируются между мастером и репликой в разных зонах доступности. Кластер готов к использованию!
