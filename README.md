# Домашнее задание к занятию "Policing и Shaping трафика"

<img width="726" height="205" alt="169968632-a928dbfa-3821-4b96-8810-56e2aa6f3099" src="https://github.com/user-attachments/assets/503617d3-734f-447f-a4fd-c38b23ad62fb" />

В указанной топологии ограничить входящую и исходящую скорость всего IPv4 трафика client1. Для ограничения использовать полисер с моделью single rate Two Color. Полисер должен обеспечивать постоянную полосу в 10Мбит/c и позволять амортизировать всплески, превышающие допустимую полосу на 6 Мбит на протяжении 1 секунды.

Отправьте полный список конфигураций: class-map, policy-map service-policy.

# ОТВЕТ:

Настройки для R1:

! --- ACL для трафика ОТ Client1 (Source) --- \
ip access-list extended CLIENT1-SRC \
 permit ip host 192.168.200.200 any

! --- ACL для трафика К Client1 (Destination) --- \
ip access-list extended CLIENT1-DST \
 permit ip any host 192.168.200.200

! --- Class-map для исходящего трафика (от клиента) --- \
class-map match-all CLIENT1-OUT \
 match access-group name CLIENT1-SRC

! --- Class-map для входящего трафика (к клиенту) --- \
class-map match-all CLIENT1-IN \
 match access-group name CLIENT1-DST

! --- Policy-map для исходящего трафика --- \
policy-map POLICE-CLIENT1-OUT \
 class CLIENT1-OUT \
  police cir 10000000 bc 125000 be 750000 conform-action transmit exceed-action drop

! --- Policy-map для входящего трафика --- \
policy-map POLICE-CLIENT1-IN \
 class CLIENT1-IN \
  police cir 10000000 bc 125000 be 750000 conform-action transmit exceed-action drop

! --- Применение на интерфейсе --- \
interface Ethernet0/1 \
 service-policy input POLICE-CLIENT1-IN \
 service-policy output POLICE-CLIENT1-OUT 

Тестирование с помощью ipref3.

 <img width="1530" height="523" alt="iperf3" src="https://github.com/user-attachments/assets/6857e7ac-02d3-4608-987f-cdfe301496aa" />

На скриншоте видно что сервер занят, кто-то тестит его другой.
