# Домашнее задание к занятию 1 «Disaster recovery и Keepalived» - `Khalatov Alexandr`

### Задание 1

- Дана [схема](https://github.com/IMiroxxI/Disaster-recovery-Keepalived/blob/main/1/hsrp_advanced.pkt) для Cisco Packet Tracer, рассматриваемая в лекции.
- На данной схеме уже настроено отслеживание интерфейсов маршрутизаторов Gi0/1 (для нулевой группы)
- Необходимо аналогично настроить отслеживание состояния интерфейсов Gi0/0 (для первой группы).
- Для проверки корректности настройки, разорвите один из кабелей между одним из маршрутизаторов и Switch0 и запустите ping между PC0 и Server0.
- На проверку отправьте получившуюся схему в формате pkt и скриншот, где виден процесс настройки маршрутизатора.

### *Решение:*

Схема pkt: [hsrp_advanced-hw.pkt](https://github.com/IMiroxxI/Disaster-recovery-Keepalived/blob/main/1/hsrp_advanced-hw.pkt)

#### Процесс настройки маршрутизатора и проверка:
![1](https://github.com/IMiroxxI/Disaster-recovery-Keepalived/blob/main/1/img1/1.png)
![1.1](https://github.com/IMiroxxI/Disaster-recovery-Keepalived/blob/main/1/img1/1.1.png)

-----

### Задание 2

- Запустите две виртуальные машины Linux, установите и настройте сервис Keepalived как в лекции, используя пример конфигурационного [файла](1/keepalived-simple.conf).
- Настройте любой веб-сервер (например, nginx или simple python server) на двух виртуальных машинах
- Напишите Bash-скрипт, который будет проверять доступность порта данного веб-сервера и существование файла index.html в root-директории данного веб-сервера.
- Настройте Keepalived так, чтобы он запускал данный скрипт каждые 3 секунды и переносил виртуальный IP на другой сервер, если bash-скрипт завершался с кодом, отличным от нуля (то есть порт веб-сервера был недоступен или отсутствовал index.html). Используйте для этого секцию vrrp_script
- На проверку отправьте получившейся bash-скрипт и конфигурационный файл keepalived, а также скриншот с демонстрацией переезда плавающего ip на другой сервер в случае недоступности порта или файла index.html.

### *Решение:*
Bash-скрипт: [check_server.sh](https://github.com/IMiroxxI/Disaster-recovery-Keepalived/blob/main/2/check_server.sh)
Код скрипта:
```bash
#!/bin/bash
if [[ $(netstat -ant | grep LISTEN | grep 80) ]] && [[ -f /var/www/html/index.nginx-debian.html ]]; then
  exit 0
else
  sudo systemctl stop keepalived
fi
```
Обязательно для скрипта выдаем права на исполнение `sudo chmod +x /usr/local/bin/check_port.sh`

Далее проводим установку Keepalive и настраиваем конфигурационные файлы.
Конфигурационный файл MASTER: [keepalived-11.conf](https://github.com/IMiroxxI/Disaster-recovery-Keepalived/blob/main/2/keepalived-11.conf)
Дополнительно сама конфигурация:
```bash
vrrp_script check_server {
        script "/etc/keepalived/check_server.sh"
        interval 3
}

vrrp_instance VI_1 {
        state MASTER
        interface enp0s8
        virtual_router_id 15
        priority 255
        advert_int 1

        virtual_ipaddress {
                192.168.123.99/24
        }

        track_script {
                check_server
        }

}
```

Конфигурационный файл BACKUP: [keepalived-22.conf](https://github.com/IMiroxxI/Disaster-recovery-Keepalived/blob/main/2/keepalived-22.conf)
Дополнительно сама конфигурация:
```bash
vrrp_instance VI_1 {
        state BACKUP
        interface enp0s8
        virtual_router_id 15
        priority 155
        advert_int 1

        virtual_ipaddress {
                192.168.123.99/24
        }

}
```

Устанавливаем Nginx, проводим настройку стартовых страниц на обеих виртуалных машинах с выводом их ip-адресов (для простоты работы) и проверяем их работоспособность.
![2](https://github.com/IMiroxxI/Disaster-recovery-Keepalived/blob/main/2/img2/2.png)
![2.1](https://github.com/IMiroxxI/Disaster-recovery-Keepalived/blob/main/2/img2/2.1.png)
![2.2](https://github.com/IMiroxxI/Disaster-recovery-Keepalived/blob/main/2/img2/2.2.png)

Далее проверяем как отрабатывает keepalive и скрипт. Меняем название стартовой страницы `sudo mv index.nginx-debian.html index.nginx-debian1.html`. Таким образом скрипт должен определить по одному из условий что файл отсутствует и сделать переброс на другой сервер.
![2.3](https://github.com/IMiroxxI/Disaster-recovery-Keepalived/blob/main/2/img2/2.3.png)
![2.4](https://github.com/IMiroxxI/Disaster-recovery-Keepalived/blob/main/2/img2/2.4.png)
Далее возвращаем наименование файла обратно и делаем рестарт keepalive.service.
![2.5](https://github.com/IMiroxxI/Disaster-recovery-Keepalived/blob/main/2/img2/2.5.png)
![2.6](https://github.com/IMiroxxI/Disaster-recovery-Keepalived/blob/main/2/img2/2.6.png)

-----
