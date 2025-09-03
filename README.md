# Oracle XE + HR через Docker Desktop + SQL Developer

## 0) Установка Docker Desktop

1. Скачай [Docker Desktop](https://www.docker.com/products/docker-desktop/) и установи.
2. Включи поддержку **WSL 2** (если спросят).
3. После перезагрузки проверь:

   ```bash
   docker --version
   ```

   Должна показаться версия.

---

## 1) Запуск Oracle XE

```bash
docker run -d --name oracle-xe -p 1521:1521 -p 5500:5500 -e ORACLE_PASSWORD=MyStrongPass1 -v oracle-data:/opt/oracle/oradata gvenzl/oracle-xe
```

* База: Oracle XE 21c
* Host/Port: `localhost:1521`
* Service: `XEPDB1`
* Админ: `system/MyStrongPass1`
* Данные → Docker volume `oracle-data`.

Проверка:

```bash
docker logs -f oracle-xe
```

Ждём `DATABASE IS READY TO USE!`.

---

## 2) SQL Developer

Скачай [SQL Developer](https://www.oracle.com/tools/downloads/sqldev-downloads.html).

* Для Windows — бери ZIP bundle with JDK.
* Для macOS/Linux — тоже zip, запускается через скрипт `sqldeveloper.sh`.

---

## 3) Подключение SYSTEM

Connection:

* User: `system`
* Pass: `MyStrongPass1`
* Host: `localhost`
* Port: `1521`
* Service: `XEPDB1`

---

## 4) Установка HR

1. Скачай [db-sample-schemas](https://github.com/oracle-samples/db-sample-schemas).
2. В папке `human_resources` открой `hr_install.sql` в SQL Developer.
3. Запусти (F5) → укажи пароль `hr`, tablespace `USERS`, temp `TEMP`, log = `hr_install.log`.
4. Если спросит *overwrite* → пиши `YES`.

---

## 5) Проверка

Создай соединение `hr/hr@XEPDB1`:

```sql
select table_name from user_tables order by 1;
select count(*) from employees;
```

---

## 6) Типовые ошибки

* **ORA-01920 (конфликт HR)**:

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
* **Темы**: только редактор кода → `Tools → Preferences → Code Editor → PL/SQL Syntax Colors → Scheme = Twilight`.

---

## 7) Управление контейнером

* Стоп:

  ```bash
  docker stop oracle-xe
  ```
* Старт:

  ```bash
  docker start oracle-xe
  ```
* Пересоздание:

  ```bash
  docker rm -f oracle-xe
  docker run -d --name oracle-xe -p 1521:1521 -p 5500:5500 -e ORACLE_PASSWORD=MyStrongPass1 -v oracle-data:/opt/oracle/oradata gvenzl/oracle-xe
  ```

---

## 🔹 Примечания по ОС

* **Windows 10/11**
  Обязательно включи WSL2 при установке Docker Desktop. SQL Developer качай с JDK внутри.
* **macOS (Intel и Apple Silicon)**
  Docker Desktop тоже работает. Команды те же. SQL Developer качается для macOS (zip-архив).
* **Linux (Ubuntu, Debian, Fedora)**
  Можно поставить просто Docker Engine без Desktop. SQL Developer запускается скриптом `sqldeveloper.sh`.
  Многие предпочитают DBeaver вместо SQL Developer (нативный dark mode, стабильнее UI).


Хочешь, я дополню этот гайд ещё «минимальным пакетом запросов» по HR (например, выборка всех сотрудников, join с департаментами, группировка по должности), чтобы люди сразу могли проверить схему на практике?
