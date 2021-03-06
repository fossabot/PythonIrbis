### Класс IrbisConnection

Класс `IrbisConnection` - "рабочая лошадка". Он осуществляет связь с сервером и всю необходимую перепаковку данных из клиентского представления в сетевое.

Экземпляр клиента создается конструктором:

```python
from pyirbis.core import IrbisConnection

client = IrbisConnection()
```

При создании клиента можно указать (некоторые) настройки:

```python
from pyirbis.core import IrbisConnection

client = IrbisConnection(host='irbis.rsl.ru', port=5555, username='ninja')
```

Можно задать те же настройки с помощью полей `host`, `port` и т. д.:

```python
client.host = 'irbis.rsl.ru'
client.port = 5555
```

Поле|Тип|Назначение|Значение по умолчанию
----|---|----------|---------------------
host        |str | Адрес сервера|'127.0.0.1'
port        |int | Порт|6666
username    |str | Имя (логин) пользователя|None
password    |str | Пароль пользователя|None
database    |str | Имя базы данных|'IBIS'
workstation |str | Тип АРМа (см. таблицу ниже)| 'C'

Типы АРМов

Обозначение|Тип
-----------|---
'R' | Читатель
'C' | Каталогизатор
'M' | Комплектатор
'B' | Книговыдача
'K' | Книгообеспеченность
'A' | Администратор

Обратите внимание, что адрес сервера задается строкой, так что может принимать как значения вроде `192.168.1.1`, так и `irbis.yourlib.com`.

Если какой-либо из вышеперечисленных параметров не задан явно, используется значение по умолчанию.

#### Подключение к серверу и отключение от него

Только что созданный клиент еще не подключен к серверу. Подключаться необходимо явно с помощью метода `connect`, при этом можно указать параметры подключения:

```python
client.connect(host='192.168.1.2')
```

Отключаться от сервера можно двумя способами: во-первых, с помощью метода `disconnect`:

```python
client.diconnect()
```

во-вторых, с помощью контекста, задаваемого блоком `with`:

```python
from pyirbis.core import *

with IrbisConnection(host='192.168.1.3') as client:
    client.connect(username='itsme', password='secret')
    
    # Выполняем некие действия.
    # По выходу из блока отключение от сервера произойдет автоматически.
```

При подключении клиент получает с сервера INI-файл с настройками, которые могут понадобиться в процессе работы:

```python
ini = client.connect()
format_menu_name = ini.get_value('Main', 'FmtMnu', 'FMT31.MNU')
```

Полученный с сервера INI-файл также хранится в поле `ini_file`.

Повторная попытка подключения с помощью того же экземпляра `IrbisConnection` игнорируется. При необходимости можно создать другой экземпляр и подключиться с его помощью (если позволяют клиентские лицензии). Аналогично игнорируются повторные попытки отключения от сервера.

Проверить статус "клиент подключен или нет" можно с помощью преобразования подключения к типу `bool`:

```python
if not client:
    # В настоящий момент мы не подключены
    return
```

Вместо индивидуального задания каждого из полей `host`, `port`, `username`, `password` и `database`, можно использовать метод `parse_connection_string`:

```python
client.parse_connection_string('host=192.168.1.4;port=5555;username=itsme;password=secret;db=RDR;')
``` 

#### Многопоточность

Клиент написан в наивном однопоточном стиле, поэтому не поддерживает одновременный вызов методов из разных потоков.

Для одновременной отсылки на сервер нескольких команд необходимо создать соответствующее количество экземпляров подключений (если подобное позволяет лицензия сервера).

#### Подтверждение подключения

`pyirbis` самостоятельно не посылает на сервер подтверждений того, что клиент все еще подключен. Этим должно заниматься приложение, например, по таймеру. 

Подтверждение посылается серверу методом `nop`:
 
```python
client.nop()
```

#### Чтение записей с сервера

```python
record = client.read_record(mfn)
```

#### Сохранение записи на сервере

```python
client.write_record(record)
```

#### Поиск записей

```python
found = client.search('"A=ПУШКИН$"')
```
