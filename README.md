# **Общая информация**

Выполнение действий приведенных в README при выполение команды "vagrant up" станет созданние виртуальной машина с установленным nginx, который работает на порту TCP 4881. 
Порт TCP 4881 уже проброшен до хоста. SELinux включен. Firewall отключен

##Задание: Запустить nginx на нестандартном порту3-мя разными способами:

переключатели setsebool, добавление нестандартного порта в имеющийся тип, формирование и установка модуля SELinux##

----------------------------------------------
Способ #1

c помощью  переключателей setsebool

проверим, что в ОС отключен файервол

**systemctl status firewalld**

проверить, что конфигурация nginx настроена без ошибок

**nginx -t**

проверим режим работы SELinux

**getenforce**

Результат: **режим Enforcing**. Данный режим означает, что SELinux будет блокировать запрещенную активность.

Разрешим в SELinux работу nginx на порту TCP 4881 c помощью переключателей setsebool (для этого в файле audit.log находим время в которое был записан лог "type=AVC nsg=audit(1642583294.259:797)" 

Включим параметр nis_enabled и перезапустим nginx

**setsebool -P nis_enabled on**

**systemctl restart nginx**

**systemctl status nginx**

Проверить статус параметра

**getsebool -a | grep nis_enabled**

Вернём запрет работы nginx на порту 4881 обратно

**setsebool -P nis_enabled off**

----------------------------------------------

Способ #2

c помощью добавления нестандартного порта в имеющийся тип

установка пакета policycoreutils-python

**yum -y install policycoreutils-python**

Поиск имеющегося типа, для http трафика

**semanage port -l | grep http**

Добавим порт в тип http_port_t:

**semanage port -a -t http_port_t -p tcp 4881**

Проверем

**semanage port -l | grep http_port_t**

перезапустим службу nginx и проверим её работу

**systemctl restart nginx**

**systemctl status nginx**

Удаляем нестандартный порт из имеющегося типа

**semanage port -d -t http_port_t -p tcp 4881**

Проверяем

**semanage port -l | grep http_port_t**

перезапустим службу nginx и проверим её работу

**systemctl restart nginx**

**systemctl status nginx**

----------------------------------------------
Способ #3

c помощью формирования и установки модуля SELinux

Запустим nginx

**systemctl start nginx **

утилитой audit2allow создаем модуль, разрешающий работу nginx на нестандартном порту:

**grep nginx /var/log/audit/audit.log | audit2allow -M nginx**

применяем данный модуль

**semodule -i nginx.pp**

запустим службу nginx и проверим её работу

**systemctl restart nginx**

**systemctl status nginx**

Просмотр всех установленных модулей

**semodule -l**

удаление модуля

**semodule -r nginx**

----------------------------------------------



##Задание: Обеспечение работоспособности приложения при включенном SELinux##

клонирование репозитория

**git clone https://github.com/mbfx/otus-linux-adm.git**

из каталога otus-linux-adm/selinux_dns_problems развернём 2 ВМ с помощью vagrant

**vagrant up**

Подключимся к клиенту

**vagrant ssh client**

Установка bind-utils (для работы nsupdate)

**yum install bind-utils**

Попробуем внести изменения в зону

**nsupdate -k /etc/named.zonetransfer.key**

Установка policycoreutils-devel

**yum install policycoreutils-devel**

смотрим логи SELinux, чтобы понять в чём может быть проблема

**cat /var/log/audit/audit.log | audit2why**

Подключаемся к серверу ns01

**vagrant ssh ns01**

Установка policycoreutils-devel

**sudo -i**

**yum install policycoreutils-devel**

проверим логи SELinux

**cat /var/log/audit/audit.log | audit2why**

**sudo semanage fcontext -l | grep named**

Изменим тип контекста безопасности для каталога /etc/named

**sudo chcon -R -t named_zone_t /etc/named**

**ls -laZ /etc/named**

снова внести изменения с клиента

**nsupdate -k /etc/named.zonetransfer.key**

перезапускаем хосты

**systemctl reboot**

####################
