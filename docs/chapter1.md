## Пакет pyirbis

### Введение 

Пакет `pyirbis` представляет собой фреймворк для создания клиентских приложений для системы автоматизации библиотек ИРБИС64 на языке Python.

Пакет не содержит неуправляемого кода и не требует irbis64_client.dll. Успешно работает на 32-битных и 64-битных версиях операционных систем Windows и Linux.

Основные возможности пакета:

* Поиск и расформатирование записей.
* Создание и модификация записей, сохранение записей в базе данных на сервере.
* Работа с поисковым словарем: просмотр термов и постингов.
* Администраторские функции: получение списка пользователей, его модификация, передача списка на сервер, создание и удаление баз данных.
* Импорт и экспорт записей в формате ISO 2709 и в обменном формате ИРБИС.

Пакет состоит из нескольких модулей:

* **core** - содержит основную функциональность, в т. ч. для работы с записями;
* **ext** - содержит инструменты для работы с файлами MNU, TRE, OPT и т. д.;
* **iso2709** - содержит функции для импорта и экспорта записей в формате ISO 2709;
* **export** - содержит функции для импорта и экспорта записей обменном текстовом формате ИРБИС;

Поддерживаются Python, начиная с версии 3.6 и сервер ИРБИС64, начиная с 2014. Более ранне версии Python будут выдавать ошибки, т. к. в пакет использует конструкции, присущие Python 3.6. Аналогично обстоит дело и с более ранними версиями сервера ИРБИС64.

### Установка

`pyirbis` загружен в централизованный репозиторий пакетов PyPI, поэтому можно установить его с помощью стандартного клиента `pip`, входящего в поставку Python:

```
pip install pyirbis --user --upgrade
```

или

```
python -m pip install pyirbis --user --upgrade
```

(оба способа эквивалентны).

Здесь `--user` означает установку только для текущего пользователя (без этого ключа установка будет выполняться для всех пользователей и может потребовать администраторских прав), а `--upgrade` - обновление пакета при необходимости. Если уже установлена последняя версия пакета, то `pip` просто сообщит об этом и завершит работу.

Также можно установить пакет, скачав необходимые файлы с репозитория GitHub: [https://github.com/amironov73/PythonIrbis](https://github.com/amironov73/PythonIrbis).

Кроме того, доступны ночные dist-сборки на AppVeyor: [https://ci.appveyor.com/project/AlexeyMironov/pythonirbis/build/artifacts](https://ci.appveyor.com/project/AlexeyMironov/pythonirbis/build/artifacts).

### Примеры программ

Ниже прилагается пример простой программы. Сначала находятся и загружаются 10 первых библиографических записей, в которых автором является А. С. Пушкин. Показано нахождение значения поля с заданным тегом и подполя с заданным кодом. Также показано расформатирование записи в формат brief.

```python
from pyirbis.core import *

# Подключаемся к серверу
client = IrbisConnection()
client.parse_connection_string('host=127.0.0.1;port=6666;database=IBIS;user=librarian;password=secret;')
client.connect()

# Ищем все книги, автором которых является А. С. Пушкин
# Обратите внимание на двойные кавычки в тексте запроса
found = client.search('"A=ПУШКИН$"')
print(f'Найдено записей: {len(found)}')

# Чтобы не распечатывать все найденные записи, отберем только 10 первых
for mfn in found[:10]:
    # Получаем запись из базы данных
    record = client.read_record(mfn)
    
    # Извлекаем из записи интересующее нас поле и подполе
    title = record.fm(200, 'a')
    print('Заглавие:', title)
    
    # Форматируем запись средствами сервера
    description = client.format_record(BRIEF, mfn)
    print('Биб. описание:', description)
    
    print()  # Добавляем пустую строку
    
# Отключаемся от сервера
client.disconnect()
```

В следующей программе создается и отправляется на сервер 10 записей. Показано добавление в запись полей с подполями.

```python
from pyirbis.core import *

SF = SubField

# Подключаемся к серверу
client = IrbisConnection()
client.parse_connection_string('host=127.0.0.1;port=6666;database=IBIS;user=1;password=1;')
client.connect()

for i in range(10):
    # Создаем запись
    record = MarcRecord()

    # Наполняем её полями: первый автор
    record.add(700, SF('a', 'Миронов'), SF('b', 'А. В.'),
               SF('g', 'Алексей Владимирович'))

    # заглавие
    record.add(200, SF('a', f'Работа с ИРБИС64: версия {i}.0'),
               SF('e', 'руководство пользователя'))

    # выходные данные
    record.add(210, SF('a', 'Иркутск'), SubField('c', 'ИРНИТУ'),
               SF('d', '2018'))

    # рабочий лист
    record.add(920, 'PAZK')

    # Отсылаем запись на сервер.
    # Обратно приходит запись, обработанная AUTOIN.GBL
    client.write_record(record)
    print(record)  # распечатываем обработанную запись
    print()

# Отключаемся от сервера
client.disconnect()
```
