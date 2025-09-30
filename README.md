# adm_training_task15
<h1 align="center">Задание 15. Настройка мониторинга</h1>
<h3 class="western"><a name="_heading=h.h6i87lkp3f19"></a> <span style="font-family: Roboto, serif;"><span style="font-size: small;">Описание домашнего задания</span></span></h3>
<p>Настроить дашборд с 4-мя графиками</p>
<ul>
<li>память;</li>
<li>процессор;</li>
<li>диск;</li>
<li>сеть.</li>
</ul>
<p>Настроить на одной из систем:</p>
<ul>
<li>zabbix (использовать screen (комплексный экран);</li>
<li>prometheus - grafana.</li>
</ul>
<h3 class="western"><a name="_heading=h.df570rpzx1qg"></a><span style="font-family: Roboto, serif;"><span style="font-size: small;">Используемые ОС</span></span></h3>
<p>Хостовая машина Windows 10</p>
<p>Две виртуальные машины:</p>
<p><strong>server</strong> (IP 10.8.2.3) &ndash; RED OS 8 Server</p>
<p><strong>client</strong>&nbsp; (IP 10.8.2.25) &ndash; RED OS 8 Desktop</p>
<p>VirtualBox 7.0.8 r156879</p>
<h3 class="western"><span style="font-family: Roboto, serif;"><span style="font-size: small;">Выполнение</span></span></h3>
<p>Для выполнения задания будем использовать систему Zabbix.</p>
<ol start="1">
<li>Установка и настройка сервера Zabbix на виртуальной машине <strong>server</strong></li>
</ol>
<p>Подключаемся по ssh к серверу под root-пользователем</p>
<p><code>ssh root@10.8.2.3</code></p>
<p>В RED OS по умолчанию включен SELinux, надо настроить политики безопасности</p>
<p><code>setsebool -P httpd_can_network_connect on</code></p>
<p><code>setsebool -P httpd_can_network_connect_db on</code></p>
<img width="817" height="227" alt="image" src="https://github.com/user-attachments/assets/160599c8-1163-4043-a32a-b1178b3775bb" />
<p>&nbsp;</p>
<p>Обновляем систему</p>
<p><code>dnf update</code></p>
<p>Устанавливаем пакеты и добавляем сервис&nbsp;<strong>httpd</strong>&nbsp;в автозагрузку</p>
<p><code>dnf install httpd zabbix-apache-conf zabbix-sql-scripts</code></p>
<p><code>systemctl enable httpd</code></p>
<p>Устанавливаем необходимые пакеты для работы с БД</p>
<p><code>dnf install mariadb mariadb-server zabbix-server-mysql zabbix-agent</code></p>
<p>Запускаем и добавляем в автозагрузку сервис&nbsp;<strong>mariadb</strong>:</p>
<p><code>systemctl start mariadb &amp;&amp; systemctl enable mariadb</code></p>
<p>Выполняем скрипт&nbsp;первоначальной настройки СУБД (пароль <strong>root</strong>-пользователя и настройка доступа)</p>
<p><code>mysql_secure_installation</code></p>
<p>Создаем базу данных <strong>zabbix</strong> и пользователя <strong>zabbix</strong> (с паролем Qwer1234 и полным доступом к базе), которые будут использованы для дальнейшей работы. Для подключения к базе используем команду&nbsp;<strong>mysql</strong></p>
<p><code>mysql -u root -p
create database zabbix character set utf8 collate utf8_bin;
grant all privileges on zabbix.* to zabbix@localhost identified by 'Qwer1234';
quit;</code></p>
<img width="974" height="487" alt="image" src="https://github.com/user-attachments/assets/a69117da-6905-48b8-82ab-38b29b9ba369" />
<p>&nbsp;</p>
<p>Импортируем содержимое SQL-дампа в базу данных</p>
<p><code>zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql -uzabbix -pQwer1234 zabbix</code></p>
<p>Отредактируем файл конфигурации сервера Zabbix. Надо прописать данные для подключения к базе данных</p>
<p><code>nano /etc/zabbix/zabbix_server.conf</code></p>
<p>Изменяем строки (их нужно раскомментировать, если они закомментированы, и присвоить нужные значения, если их нет или они отличаются)</p>
<p><code>DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=Qwer1234</code></p>
<p>Также необходимо отредактировать файл <strong>/etc/php.ini</strong></p>
<p><code>nano /etc/php.ini</code></p>
<p>Приведем параметры к следующему виду (аналогичные параметры есть в&nbsp;<strong>/etc/httpd/conf.d/zabbix.conf</strong>&nbsp;- они должны соответствовать):</p>
<p><code>date.timezone = Europe/Moscow
post_max_size = 16M
max_execution_time = 300
max_input_time = 300</code></p>
<p>Перезапускаем и добавлям Zabbix-сервер и агент в автозагрузку:</p>
<p><code>systemctl restart zabbix-server
systemctl enable zabbix-server
systemctl restart zabbix-agent
systemctl enable zabbix-agent
systemctl restart httpd</code></p>
<p>Для того чтобы войти в веб-интерфейс, открываем в браузере&nbsp;<a href="http://10.8.2.3/zabbix">http://10.8.2.3/zabbix</a>. Откроется страница приветствия. Выбираем русский, &laquo;Далее&raquo;.</p>
<img width="974" height="488" alt="image" src="https://github.com/user-attachments/assets/39835ee3-c4df-4fb2-b44f-f0df601f7126" />
<p>&nbsp;</p>
<p>Указываем данные для подключения к ранее созданной базе данных:</p>
<p>Тип БД &ndash;&nbsp;MySQL</p>
<p>Хост БД &ndash;&nbsp;localhost&nbsp;или&nbsp;127.0.0.1</p>
<p>Порт БД &ndash;&nbsp;3306&nbsp;(стандартный порт MySQL)</p>
<p>Имя БД &ndash;&nbsp;zabbix</p>
<p>Пользователь БД &ndash;&nbsp;zabbix</p>
<p>Пароль &ndash;&nbsp;Qwer1234</p>
<p>Вводим имя сервера и часовой пояс, проверяем правильность данных &ndash; &laquo;Финиш&raquo;.</p>
<p>Пользователь по умолчанию <strong>Admin</strong>, пароль <strong>zabbix</strong>. Установка завершена, в веб-интерфейсе видим панель</p>
<img width="974" height="376" alt="image" src="https://github.com/user-attachments/assets/0327a4c3-e727-49af-8068-ff44dc798d0f" />
<p>&nbsp;</p>
<ol start="2">
<li>Установка и настройка агента на виртуальной машине client</li>
</ol>
<p>Подключаемся по ssh к клиенту под root-пользователем</p>
<p><code>ssh root@10.8.2.25</code></a></p>
<img width="691" height="189" alt="image" src="https://github.com/user-attachments/assets/26c973af-fc23-40ae-833e-b656dff4a86e" />
<p>&nbsp;</p>
<p>Устанавливаем пакет агента</p>
<p><code>dnf install zabbix-agent</code></p>
<p>Настраиваем файл конфигурации</p>
<p><code>nano /etc/zabbix/zabbix_agentd.conf</code></p>
<p>По умолчанию достаточно просто прописать IP-адрес сервера</p>
<p><code>Server=10.8.2.3</code></p>
<img width="974" height="509" alt="image" src="https://github.com/user-attachments/assets/2283a567-0777-46cc-8cdc-9d6c589b3df1" />
<p>&nbsp;</p>
<p>Запускаем сервис и добавляем его в автозапуск</p>
<p><code>systemctl enable zabbix-agent
systemctl start zabbix-agent
systemctl status zabbix-agent</code></p>
<img width="974" height="530" alt="image" src="https://github.com/user-attachments/assets/23b602a7-136b-46bf-94f8-e233545e198b" />
<p>&nbsp;</p>
<ol start = "3">
<li>Добавляем клиента на сервер</li>
</ol>
<p>В веб-интерфейсе&nbsp;Zabbix-сервера добавляем новый узел в меню &laquo;Мониторинг&raquo; -&nbsp;&laquo;Узлы сети&raquo; -&nbsp;&laquo;Создать узел сети&raquo;.</p>
<p>Далее впишем имя узла, укажем группу, в которой он должен состоять, нажав&nbsp;&laquo;Выбрать&raquo; и отметив нужные группы.&nbsp;</p>
<p>Для параметра &laquo;Интерфейсы&raquo; нажимаем&nbsp;&laquo;Добавить&raquo;, прописываем&nbsp;IP-&nbsp;или&nbsp;DNS-адрес узла и порт подключения. После этого&nbsp;нажимаем кнопку &laquo;Добавить&raquo;.</p>
<img width="974" height="556" alt="image" src="https://github.com/user-attachments/assets/d2a9c301-03ca-4bf8-a8e4-5f15d857af36" />
<p>&nbsp;</p>
<p>В узлах сети видим два агента &ndash; собственный агент сервера и агент виртуальной машины <strong>client</strong></p>
<img width="974" height="77" alt="image" src="https://github.com/user-attachments/assets/221d9f97-7ec3-4b27-b780-59a3a5a3608c" />
<p>&nbsp;</p>
<ol start = "4">
<li>Создаем дашборд с параметрами (процессор, память, диск, сеть)</li>
</ol>
<p>Нажимаем &laquo;Изменить панель&raquo; &ndash; &laquo;Добавить&raquo; &ndash; &laquo;Виджет&raquo;</p>
<p>Тип: График</p>
<p>Добавляем 4 набора данных для узла сети client:</p>
<p>Linux: CPU utilization</p>
<p>Linux: Memory utilization</p>
<p>sda: Disk utilization</p>
<p>Interface enp0s3: Operational status</p>
<img width="974" height="608" alt="image" src="https://github.com/user-attachments/assets/f77b404c-c543-4f05-b0c9-b51e4e07c7a5" />
<p>&nbsp;</p>
<p>&laquo;Применить&raquo;, &laquo;Сохранить изменения&raquo;. Главная панель будет выглядеть так:</p>
<img width="974" height="509" alt="image" src="https://github.com/user-attachments/assets/598969ef-433d-4a46-beb7-b6236daab67e" />
<p>&nbsp;</p>
<p>Графики созданы. Задание завершено.</p>
