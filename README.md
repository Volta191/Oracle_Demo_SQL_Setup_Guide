# Oracle_Demo_SQL_Setup_Guide
Small guide to setup local environment for SQL trainig on Windows 

# Oracle XE + HR через Docker Desktop + SQL Developer

## 0) Установка Docker Desktop

1. Скачай [Docker Desktop](https://www.docker.com/products/docker-desktop/) и установи.
2. Включи поддержку **WSL 2** (если спросят).
3. После перезагрузки проверь, что всё работает:

   ```bash
   docker --version
   ```

   Должна показаться версия (например, `Docker version 27.0.3`).

---

## 1) Запуск Oracle XE (одна строка)

```bash
docker run -d --name oracle-xe -p 1521:1521 -p 5500:5500 -e ORACLE_PASSWORD=MyStrongPass1 -v oracle-data:/opt/oracle/oradata gvenzl/oracle-xe
```

* База: Oracle XE 21c
* Host/Port: `localhost:1521`
* Service name: `XEPDB1`
* Админ: `system / MyStrongPass1` (замени на свой пароль)
* Данные хранятся в Docker volume `oracle-data`.

Проверка логов:

```bash
docker logs -f oracle-xe
```

Ждём строку `DATABASE IS READY TO USE!`.

---

## 2) Установка SQL Developer

1. Скачай **Oracle SQL Developer**: [официальная страница](https://www.oracle.com/tools/downloads/sqldev-downloads.html).
2. Для Windows бери **ZIP bundle with JDK included** (чтобы не мучиться с отдельной Java).
3. Распакуй архив в любую папку (например, `C:\sqldeveloper`).
4. Запускай `sqldeveloper.exe`.

---

## 3) Подключение из SQL Developer

Создай новое соединение:

* **Username**: `system`
* **Password**: `MyStrongPass1`
* **Hostname**: `localhost`
* **Port**: `1521`
* **Service name**: `XEPDB1`
  → **Test** → **Connect**.

---

## 4) Установка демо-схемы HR (EMPLOYEES, DEPARTMENTS и т.п.)

1. Скачай [db-sample-schemas](https://github.com/oracle-samples/db-sample-schemas).
2. Найди папку `human_resources`.
3. В SQL Developer: **File → Open…** → выбери `hr_install.sql`.
4. Запусти **Run Script (F5)** и ответь на вопросы:

   * HR password: `hr`
   * Default tablespace: `USERS`
   * Temporary tablespace: `TEMP`
   * Log file: `hr_install.log`
   * Если спросит «overwrite schema?» → введи **YES**.

---

## 5) Проверка

Создай новое соединение:

* User: `hr`
* Pass: `hr`
* Service: `XEPDB1`

В Worksheet:

```sql
select table_name from user_tables order by 1;
select count(*) from employees;
```

---

## 6) Типовые проблемы и быстрые фиксы

* **ORA-01920 / конфликт HR** → под SYSTEM:

  ```sql
  alter session set container = CDB$ROOT;
  drop user HR cascade;
  drop role HR;
  ```

  Потом снова `@hr_install.sql`.
* **Забыли пароль HR**:

  ```sql
  alter session set container = XEPDB1;
  alter user HR identified by NewStrongPass1 account unlock;
  ```
* **SQL Developer слишком светлый** →
  `Tools → Preferences → Code Editor → PL/SQL Syntax Colors → Scheme = Twilight` (только редактор кода станет тёмным).

---

## 7) Управление базой

* Остановить:

  ```bash
  docker stop oracle-xe
  ```
* Запустить снова:

  ```bash
  docker start oracle-xe
  ```
* Пересоздать (данные сохранятся в volume):

  ```bash
  docker rm -f oracle-xe
  docker run -d --name oracle-xe -p 1521:1521 -p 5500:5500 -e ORACLE_PASSWORD=MyStrongPass1 -v oracle-data:/opt/oracle/oradata gvenzl/oracle-xe
  ```
