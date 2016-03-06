rails-nginx-passenger-ubuntu
============================

Установка и настройка production server на Ubuntu 14.04
RVM Nginx Passenger PostgreSQL Redis SphinxSearch

Пользователи, rsa-ключи и права доступа
-----------------------------------------------------------
  заходим на сервер под root
  
    ssh root@192.168.0.100
    
  создаем пользователя deployer
  
    sudo adduser deployer

  открываем конфигурацию управления правами пользователя
  
    sudo visudo

  найти строчку root ALL=(ALL:ALL) ALL и после нее добавить
  
    deployer ALL=(ALL:ALL) ALL

  меняем порт подключения по SSH
  
    sudo nano /etc/ssh/sshd_config

  найти строчку Port 22 и заменить ее на
  
    Port 4321 (или любой другой)
  
  перезагружаем SSH
  
    sudo reload ssh

  завершаем сеанс пользователя root
  
    exit

  заходим на сервер под deployer
  
    ssh deployer@192.168.0.100 -p 4321
  
  создаем папку ssh
  
    mkdir ~/.ssh
  
  на локальном компьютере генерируем shh
  
    ssh-keygen -t rsa
  
  на локальном компьютере отправляем ssh ключ на сервер
  
    cat ~/.ssh/id_rsa.pub | ssh -p 4321 deployer@192.168.0.100 'cat >> /home/deployer/.ssh/authorized_keys'
  
  на сервере нужно убедиться что authorized_keys создан
  
    ls ~/.ssh/
  
  завершаем сеанс пользователя deployer
  
    exit
  
  заходим на сервер под deployer если все хорошо, то пароль не спросит
  
    ssh deployer@192.168.0.100 -p 4321

Update and upgrade
------------------

    sudo apt-get update
    sudo apt-get upgrade
    sudo reboot
    
  после перезагрузки заходим на сервер под deployer
    ssh deployer@192.168.0.100 -p 4321

Тайм зона
--------------

  комфигурируем тайм зону
  
    sudo dpkg-reconfigure tzdata
  
  выбираем регион
  
    Europe
  
  выбираем город
  
    Kiev
  
  убелиться что таймзона изменилась
  
    date

Устанавливка RVM
------------------------

  устанавливаем curl
    
    sudo apt-get install curl
  
  устанавливаем RVM
    
    gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3

    \curl -sSL https://get.rvm.io | bash -s stable
  
  загружаем RVM в текущую сесию
    
    source /home/deployer/.rvm/scripts/rvm
	
  устанавливаем пакеты нужны для установки RVM
    
	rvm requirements
	
	или
    
	rvmsudo /usr/bin/apt-get install build-essential openssl libreadline6 libreadline6-dev curl git-core zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt-dev autoconf libc6-dev ncurses-dev automake libtool bison subversion
   
  устанавливаем ruby
    
    rvm install 2.2.3
  
  указываем что эта версия ruby будет использоваться по умолчанию
    
    rvm use 2.2.3 --default
  
  проверяем версию ruby
    
    ruby -v
	
  устанавливаем bundler
  
    gem install bundler

Устанавливка PostgreSQL
-------------------------------

  устанавливаем PostgreSQL
  
    sudo apt-get install postgresql postgresql-contrib postgresql-server-dev-9.3
  
  устанавливаем пароль для postgres
  
    sudo -u postgres psql
  
    alter user postgres with password 'qwerty';
    \q
      
GIT
---

  уставливаем git
  
    sudo apt-get install git-core

Nginx
-----

  устанавливаем passenger + nginx
  
    gem install passenger
  
    sudo apt-get install libcurl4-openssl-dev
  
    rvmsudo passenger-install-nginx-module
    
    1. жмем просто Enter
	2. Стрелочками переходим на Python снмаем галочку нажатием Space, убедились что выделен только Ruby и жмем Enter
	3. Жмем 1 и далее Enter
	4. Жмем Enter
	5. Завершаем установку Жмем Enter
	
  открываем конфигурационный файл nginx
  
    sudo nano /opt/nginx/conf/nginx.conf
  
  редактируем блок server
  
      gzip on;

      server {
        listen 80;
        server_name 192.168.0.100;
        root /home/deployer/qna/current/public;
        passenger_enabled on;

        location ^~ /assets/ {
          gzip_static on;
          expires max;
          add_header Cache-Control public;
        }
      }

  загружаем скрипт управления nginx
  
    git clone https://github.com/sanyco86/rails-nginx-passenger-ubuntu.git
  
  копируем скрипт
  
    sudo cp rails-nginx-passenger-ubuntu/nginx/nginx /etc/init.d/
  
  даем скрипту права на запуск 
  
    sudo chmod +x /etc/init.d/nginx
  
  удаляем загруженный скрипт
  
    rm -rf rails-nginx-passenger-ubuntu/
  
  запускаем nginx
  
    sudo /etc/init.d/nginx start
  
  проверяем на локальном компьютере что сервер доступен
  
    http://192.168.0.100

Redis
-----

  устанавливаем redis для sidekiq
  
    sudo apt-get install redis-server
  
  конфигурируем redis
  
    sudo cp /etc/redis/redis.conf /etc/redis/redis.conf.default
  
  перезапускаем redis
  
    sudo service redis-server restart

Sphinx
------

  устанавливаем sphinx
    
    sudo apt-get install sphinxsearch
    
    
