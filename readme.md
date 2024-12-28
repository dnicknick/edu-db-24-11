## ДЗ-03

Описание контейнера с PostgreSQL и pgAdmin в файле `docker-compose.yml`
Учетные данные описываются в файле `.env` или применяются из файла `docker-compose.yml`

Подключение к pgAdmin через браузер
![scheme](./.doc/web-client.png)

На скрине видна созданная для эксперимента БД `db_test_first`

Другое подключение к pgAdmin тоже через браузер
![scheme](./.doc/web-client-2.png)

Настройка соединения через внешнюю программу
![scheme](./.doc/app.png)
Этот способ также успешно подключается
![scheme](./.doc/app-ok.png)

И проверка попытки подключения через командную строку
![scheme](./.doc/terminal.png)