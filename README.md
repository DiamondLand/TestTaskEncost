### 1. Клиент не получил ежедневный почтовый отчет (excel-файл, который формируется на нашем сервере с помощью python, содержит в себе данные о простоях и работе).
1) Какой по вашему мнению алгоритм решения проблемы?
- Проверить настройку почтового сервера и изучить его логи. Посмотреть код Python для того чтобы убедиться в его исправности
2) Что Вы ответите клиенту?
- Здравствуйте, просим прощения за предоставленные неудобства. Отправим файл вручную
3) Что делать если эта ситуация повторяется несколько раз?
- Доложить руководству

### 2. К тестовому заданию приложена база данных SQLite - client, в ней есть 3 таблицы:
endpoints – представляет станки на предприятии
endpoints_reasons – представляет причины простоя
endpoints_groups – представляет участки на предприятии
1) Написать запрос, который выводит причины простоя только активных станков.
```SQL
SELECT endpoints.name, endpoint_reasons.reason_hierarchy 
FROM endpoints
JOIN endpoint_reasons ON endpoints.id = endpoint_reasons.endpoint_id
WHERE endpoints.active = 'true'
```
2) Написать запрос, который выводит количество причин простоев для каждой неактивной точки
```SQL
SELECT endpoints.id, count(endpoint_reasons.reason_hierarchy) AS reasons_count
FROM endpoints
JOIN endpoint_reasons ON endpoints.id = endpoint_reasons.endpoint_id
WHERE endpoints.active = 'false'
GROUP BY endpoints.id
```
3) Написать запрос, который выведет для каждого активного оборудования, количество причины простоя “ Перебои напряжения” (нужно учесть что это надо сделать для каждой группы(reason_hierarchy))
```SQL
SELECT endpoints.id, count(endpoint_reasons.reason_name) AS reasons_count
FROM endpoints
JOIN endpoint_reasons ON endpoints.id = endpoint_reasons.endpoint_id
WHERE endpoints.active = 'true' AND endpoint_reasons.reason_name = 'Перебои напряжения'
GROUP BY endpoints.id, endpoint_reasons.reason_name
```

### 3. Клиент просит сделать интеграцию с 1С.Предприятие(в 95% случаев клиентские запросы сначала попадают к Вам):
1) Что Вы ему ответите?
- Здравствуйте! Да, можем интегрировать 1С.Предприятие. Что конкретно вас интересует?
2) Ваш план действий после того как Вы ему ответили?
- Ознакомиться с тем, что хочет клиент. Обратиться к тому, кто дальше будет готов помочь ему

### 4.Произошло серьезное падение сервера, которое продлилось несколько часов, у множества клиентов не было данных за этот период:
1) Что Вы ответите клиенту?
- Здравствуйте! Знаем о проблемы. Приносим свои извинения, уже работаем над решением.  

### 5. Клиент не может попасть в личный кабинет (клиенту предоставляется логин/пароль), ваши действия?
- Спросить какую выдаёт ошибку, посмотреть логи. Помочь

### 6. Есть огромный текстовый файл (более 100 ГБ - он точно не поместится в оперативной памяти), состоящий из строк, как его оптимально обрабатывать? - Напишите код на python.
```python
import pandas as pd

file_path = 'encost.txt'

# Функция для обработки каждой строки
def process_line(line):
    processed_line = line
    return processed_line

# Чтение и обработка файла в DataFrame
df = pd.read_csv(file_path, header=None, names=['text'])
df['processed_text'] = df['text'].apply(process_line)

# Запись обработанных строк в другой файл
df['processed_text'].to_csv('encost_new.txt', index=False, header=False, mode='a')
```

### 7. Что бы вы ответили клиенту? С чего бы начали проверку?
- Здравствуйте, скоро разберёмся в ситуации! О изменениях сообщим.
- Проверку бы начал с логов и данных от самого предприятия

### 8. Напишите Python скрипт который будет писать в базу данных client следующую информацию:
Добавить 3 станка: “Сварочный аппарат №1”, “Пильный аппарат №2”, “Фрезер №3”, сделать их активными.
Скопировать со станков: “Фрезерный станок”, “Старый, ЧПУ”, “Сварка”, причины простоя и перенести их на новые станки.
Определить группу “Цех №2” для новых станков.
Добавить станки “Пильный станок” и “Старый ЧПУ” к новой группе.
```python
import sqlite3

# Подключение к базе данных
conn = sqlite3.connect('machines.db')
cursor = conn.cursor()

# 1. Добавить новые станки и установить их статус как "активный"
new_machines = [
    ("Сварочный аппарат №1", "активный"),
    ("Пильный аппарат №2", "активный"),
    ("Фрезер №3", "активный")
]

cursor.executemany('INSERT INTO machines (name, status) VALUES (?, ?)', new_machines)

# Получение id новых станков
cursor.execute('SELECT id, name FROM machines WHERE name IN ("Сварочный аппарат №1", "Пильный аппарат №2", "Фрезер №3")')
new_machine_ids = {name: id for id, name in cursor.fetchall()}

# 2. Скопировать причины простоев со старых станков на новые
old_machine_names = ["Фрезерный станок", "Старый, ЧПУ", "Сварка"]
new_machine_names = ["Фрезер №3", "Фрезер №3", "Сварочный аппарат №1"]

for old_name, new_name in zip(old_machine_names, new_machine_names):
    cursor.execute('SELECT id FROM machines WHERE name = ?', (old_name,))
    old_machine_id = cursor.fetchone()[0]

    cursor.execute('SELECT id FROM machines WHERE name = ?', (new_name,))
    new_machine_id = cursor.fetchone()[0]

    cursor.execute('SELECT reason FROM downtime_reasons WHERE machine_id = ?', (old_machine_id,))
    downtime_reasons = cursor.fetchall()

    for (reason,) in downtime_reasons:
        cursor.execute('INSERT INTO downtime_reasons (machine_id, reason) VALUES (?, ?)', (new_machine_id, reason))

# 3. Определить группу "Цех №2" для новых станков
cursor.execute('INSERT INTO groups (name) VALUES ("Цех №2")')
cursor.execute('SELECT id FROM groups WHERE name = "Цех №2"')
group_id = cursor.fetchone()[0]

for new_machine_id in new_machine_ids.values():
    cursor.execute('UPDATE machines SET group_id = ? WHERE id = ?', (group_id, new_machine_id))

# 4. Добавить станки "Пильный станок" и "Старый ЧПУ" к новой группе
additional_machines = ["Пильный станок", "Старый ЧПУ"]

for machine_name in additional_machines:
    cursor.execute('SELECT id FROM machines WHERE name = ?', (machine_name,))
    machine_id = cursor.fetchone()[0]
    cursor.execute('UPDATE machines SET group_id = ? WHERE id = ?', (group_id, machine_id))

# Сохранение изменений и закрытие соединения
conn.commit()
conn.close()
```
