1. Архитектура ANSI-SPARC. 
   - Внешний (Пользовательский) уровень
     Это представление пользователем предметной области, с которой он взаимодействует частично.  Пользователь не знает о фактическом размещении данных в памяти компьютера, механизмах для оптимизации операций с данными, обработке запроса и осуществлении поиска.
     И отображение ему в виде пользовательского интерфейса.
   - Промежуточный (концептуальный)
     то с чем ебашится сис админы формируют модели бд
   - Внутренний (физический)
     способ хранения организации на уровне процессов памяти, как хранится бд в субд.
     
2. Архитектура PostgreSQL. 
   ![[Pasted image 20240623231533.png]]
   [[001 Архитектура]]
3. Разделяемая память PostgreSQL. 
   [[001 Архитектура]]
4. Буферная память процессов PostgreSQL. 
   [[001 Архитектура]]
5. Процессы, обеспечивающие работу PostgreSQL. 
   [[001.1.Фоновые (красные)]]
6. Системный каталог, организация, способы взаимодействия. 
7. Схема, особенности работы со схемами в PostgreSQL, search_path. 
8. Управление доступом к данным в PostgreSQL, привилегии, пользователи, роли. 
9. Работа с ролями, INHERIT, NOINHERIT. 
      ```sql 
    CREATE ROLE parent_role;
	GRANT SELECT ON ALL TABLES IN SCHEMA public TO parent_role;
	
	CREATE ROLE child_role WITH INHERIT LOGIN PASSWORD 'password'; GRANT parent_role TO child_role;
	```
10. Установка и запуск PostgreSQL.
     [[003.Установка и запуск]]
11. PostgreSQL: template0, template1, назначение, особенности работы. 
    template0 — служит в виде резервной копии. 
    template1 — используется в качестве шаблона при создании других БД.
    При изменении template1 все бд тоже примут изменения.
12. Подключение к PostgreSQL, настройка, особенности.
    [[003.Установка и запуск]]
13. Виды и методы подключений к PostgreSQL, их настройка. 
	[[004.Подключение к DB]]
14. PostgreSQL, создание Базы Данных.
    По умолчанию бд создаётся из template1.
    `CREATE DATABASE newDb3 TEMPLATE newDB1;` 
	
	Чтобы установить бд в template бд:
	`UPDATE pg_database SET datistemplate = true WHERE datname = 'my_template';`
	
	`UPDATE pg_database SET datallowconn = false WHERE datname = 'my_template';` - запрещаем новые подключения к шаблону, чтобы избежать неконсистентности.
	
15. Файловая структура и конфигурация PostgreSQL. 
    ##### Файловая структура
    ![[Pasted image 20240624165503.png|450]]
    **pg_default** соответствует директории base.
    **pg_global** - global
    ##### Конфигурационные файлы
    **postgresql.conf**
    **postgresql.auto.conf** - для динамических изменений типа `ALTER SYSTEM`.
    **pg_hba.conf**
    **pg_ident.conf** - для сопоставления имён при подключении. В **pg_hba.conf** можно указать разные файлы сопоставления с помощью map.
    ##### Конфигурация
    Ручками через **postgresql.conf**.
    На уровне кластера с помощью команды - `ALTER SYSTEM SET shared_buffers = '256MB';`
    - На уровне бд - `ALTER DATABASE mydb SET work_mem = '64MB';`
    - На уровне пользователя - `ALTER ROLE myuser SET work_mem = '32MB';`
    - Для сессии - `SET work_mem TO '16MB';`
    - Параметры командной строки при запуске - `postgres -D /path/to/data -c max_connections=200`
16. Табличные пространства. Назначение. Организация и настройка. 
    
    Позволяют физически/логически распределять объекты бд по физической системе. Можно на другой диск вынести к примеру для безопасности какие то данные, если место заканчивается тоже раскидать можно.
    Системный каталог: **pg_tablespace**
```sql
    CREATE TABLESPACE newTablespace LOCATION '/diskA/someDir'; 
    CREATE TABLE STUDENT ( id integer NOT NULL ) TABLESPACE newTablespace;
```

.
	Когда таблицы создаются -  помещаются в **pg_default**. 
	Есть также **pg_global**, где находтся глобальные объекты типа ролей,  разных каталогов и индексов.
	
17. Транзакции. Назначение. ACID. 
    **Atomicity** Атомарность - выполняются либо все операции, либо ни одна
    
	**Consistency** Согласованность - после выполнения транзакций бд должна быть целостной в соответствии с ограничениями
	
	**Isolation** Изолированность - параллельно выполняемые транзакции не должны знать друг о друге пока не выполнились
	
	**Durability** Устойчивость, долговечность - результат после выполнения должен сохраняться (системы восстановления)
    
18. PostgreSQL: явные, неявные транзакции. Транзакционный DDL. 
	
	- Implicit (Неявные) - любой sql неявно оборачивается в транзакцию (BEGIN, COMMIT)
	- explicit (Явные) - пользователь сам задаёт
	
	Транзакционный DDL - чёто менять и если потом не нравится состояние бд - то откатиться можно
	```bash
	BEGIN;
	//DDL
	ROLLBACK;
	```
	    
19. PostgreSQL: время начала транзакции, время внутри транзакции. 
    Между двумя неявными время будет меняться, внутри явной - одинаковое время, тк возвращает время начала транзакции (SELECT CURRENT_TIME).
    Если хотим получить действительное время - **clock_timestamp();**
    
20. PostgreSQL: идентификация транзакции, особенности работы xid 
    [[005.1.xid, служебная информация]]
    
21. PostgreSQL: MVCC, SSI 
    [[005.5.Snapshot Isolation, snapshot]]
22. Виды и особенности конфликтов при параллельном доступе к данным. [[005.3.Конфликты]]
    [[005.4.Виды изоляций, режимы орагиназции доступа к данным в postgresql]]
23. Изоляция транзакций. Режимы для организации доступа к данным. 
    [[005.4.Виды изоляций, режимы орагиназции доступа к данным в postgresql]]
24. VACUUM 
    [[005.6.Vacuum]]
25. Восстановление данных. Базовые понятия. Организация. Контрольные точки. 
26. (Adv.) UNDO-журнал. 
27. (Adv.) REDO-журнал. 
28. (Adv.) UNDO/REDO-журнал. 
29. (Adv.) Восстановление данных. Подходы (steal/no-steal, force/no-force). ARIES. 
30. Восстановление данных в PostgreSQL. WAL. LSN. 
31. Резервное копирование. Базовые понятия. Виды. 
32. Логическое резервное копирование. pg_dump, pg_dumpall, pg_restore. 
33. Физическое резервное копирование. Способы организации в PostgreSQL. 
34. PostgreSQL: непрерывное архивирование. 
35. Репликация данных в PostgreSQL. Базовые понятия. 
36. Виды репликации. Особенности. 
37. Физическая репликация. Реализация в PostgreSQL. 
38. Настойка физической репликации. wal_keep_size, слоты. 
39. (Adv.) Синхронная и асинхронная репликация в PostgreSQL. 
40. (Adv.) Ступенчатая (каскадная) репликация. 
41. (Adv.*) Логическая репликация в PostgreSQL.