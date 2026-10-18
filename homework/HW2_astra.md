Смотрим какие есть пакеты
apt list | grep postgres

```sh
#Установка пакета 
sudo apt install -y postgresql-common
    # проверка установки пакета 
	dpkg -l postgresql-common
	    #Удаление пакета
            sudo apt remove --purge postgresql-common
# Установка как на ubuntu не работает, идём другим путём
    # Установка curl и ca-certificates
    sudo apt install -y curl ca-certificates
    #Импорт ключа подписи репозитория
    sudo install -d /usr/share/postgresql-common/pgdg

    # подключение репозитория 
    sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
    sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc
    
    # Создание файла конфигурации репозитория
    sudo sh -c 'echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt bookworm-pgdg     main" > /etc/apt/sources.list.d/pgdg.list'
#Установка postgresql
sudo apt install -y postgresql-15 
 

``` 