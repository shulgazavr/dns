# Vagrant-стенд c DNS

### Описание домашнего задания
На предложенном [стенде](https://github.com/erlong15/vagrant-bind) выполнить слудующие задания:
### Часть 1. Типовые задачи DNS-сервера.
1. Добавить ВМ.
2. Создать новую зону `dns.lab` с записями: 
- `web1` - смотрит на сервер `client1`;
- `web2` - смотрит на сервер `client2`.
3. Создать новую зону `dns.lab` с записями: 
- www - смотрит на оба клиентских сервера.

### Часть 2. Настройка `split-dns`.
1. `client1` видит обе зоны, но в зоне dns.lab только web1.
2. `client2` видит только dns.lab.
* настроить все без выключения selinux

### Структура каталогов и файлов проекта. Краткое описание.
```
.
├── provisioning
│   ├── client-motd
│   ├── client-resolv.conf
│   ├── master-named.conf
│   ├── named.ddns.lab
│   ├── named.dns.lab
│   ├── named.dns.lab.client
│   ├── named.dns.lab.rev
│   ├── named.newdns.lab
│   ├── named.zonetransfer.key
│   ├── playbook.yml
│   ├── rndc.conf
│   ├── servers-resolv.conf.j2
│   ├── slave-named.conf
│   └── zonetransfer.key
├── README.md
└── Vagrantfile
```
`Vagrantfile` - файл сценария для Vagrant. Создание виртуальных машин, установка ОС и первичная настройка.
`client-motd` - файл-приветствие. <br/>
`client-resolv.conf` - файл с перечнем DNS-серверов. <br/>
`master-named.conf` - основной конфигурационный файл DNS (named.conf) для сервера в роли `master`. <br/>
`named.ddns.lab` - файл зоны ddns.lab. <br/>
`named.dns.lab` - файл зоны dns.lab. <br/>
`named.dns.lab.client` - файл зоны dns.lab. <br/>
`named.dns.lab.rev` - файл зоны обратного просмотра. <br/>
`named.newdns.lab` - файл зоны  newdns.lab. <br/>
`named.zonetransfer.key` - файл, содержащий в себе "ключ", для доступа к копиям файлов зон основного сервера DNS. <br/>
`playbook.yml` - пьеса `Ansible`. Установка и настройка ПО, копирование конфигурационных файлов сервисов на ВМ. <br/>
`rndc.conf` - . <br/>
`servers-resolv.conf.j2` - jinja2 темплейт. Универсализация типовых шаблонов (в данном случае, файла `resolv.conf`). <br/>
`slave-named.conf` - основной конфигурационный файл DNS (named.conf) для сервера в роли `slave`. <br/>
`zonetransfer.key` - файл, содержащий в себе "ключ", для доступа к копиям файлов зон основного сервера DNS. <br/>

### Реализация.
#### Часть 1.
1. В `Vagrantfile` добавлен блок:
```
  config.vm.define "client2" do |client2|
    client2.vm.network "private_network", ip: "192.168.50.16", virtualbox__intnet: "dns"
    client2.vm.hostname = "client2"
  end
```
2. В `named.dns.lab` добавлены строки:
```
;WEB
web1            IN      A       192.168.50.15
web2            IN      A       192.168.50.16
```
3. В `named.newdns.lab` добавлены строки:
```
;WWW
www             IN      A       192.168.50.15
www             IN      A       192.168.50.16
```

#### Часть 2.
`Split DNS` - это конфигурация, позволяющая отдавать разные записи зон DNS в зависимости от адреса(подсети) источника запроса.

> Технология Split-DNS реализуется с помощью описания представлений (view), для каждого отдельного acl. В каждое представление (view) добавляются только те зоны, которые разрешено видеть хостам, адреса которых указаны в access листе.

Порядок выполнения задачи:
1. Создание дополнительного файл зоны dns.lab `named.dns.lab.client`.
2. Создание ACL для client1 и client2, при помощи утилиты `tsig-keygen`.
3. Разделение зон по представлениям:
- `view "client1"` - содержит зоны "dns.lab" и "newdns.lab"
- `view "client2"` - содержит зоны "dns.lab" и "50.168.192.in-addr.arpa"
- `view "default"` - содержит зоны ".", "dns.lab", "50.168.192.in-addr.arpa", "ddns.lab", "newdns.lab"

Проверка осужествляется резолвером.

С сервера `client1`:
```
[vagrant@client1 ~]$ ping www.newdns.lab -c 1
PING www.newdns.lab (192.168.50.15) 56(84) bytes of data.
64 bytes from client1 (192.168.50.15): icmp_seq=1 ttl=64 time=0.009 ms

--- www.newdns.lab ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.009/0.009/0.009/0.000 ms
```
```
[vagrant@client1 ~]$ ping web1.dns.lab -c 1
PING web1.dns.lab (192.168.50.15) 56(84) bytes of data.
64 bytes from client1 (192.168.50.15): icmp_seq=1 ttl=64 time=0.008 ms

--- web1.dns.lab ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.008/0.008/0.008/0.000 ms
```
```
[vagrant@client1 ~]$ ping web2.dns.lab
ping: web2.dns.lab: Name or service not known
```
С сервера `client2`:
```
[vagrant@client2 ~]$ ping www.newdns.lab -c 1
ping: www.newdns.lab: Name or service not known
```
```
[vagrant@client2 ~]$ ping web1.dns.lab -c 1
PING web1.dns.lab (192.168.50.15) 56(84) bytes of data.
64 bytes from 192.168.50.15 (192.168.50.15): icmp_seq=1 ttl=64 time=0.888 ms

--- web1.dns.lab ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.888/0.888/0.888/0.000 ms
```
```
[vagrant@client2 ~]$ ping web2.dns.lab -c 1
PING web2.dns.lab (192.168.50.16) 56(84) bytes of data.
64 bytes from client2 (192.168.50.16): icmp_seq=1 ttl=64 time=0.036 ms

--- web2.dns.lab ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.036/0.036/0.036/0.000 ms
```

Исходя из полученных данных, видно, что серверу `client1` доступны зоны "dns.lab" и "newdns.lab", однако информацию о хосте web2.dns.lab он получить не может.

В свою очердь, серверу `client2` доступна вся зона "dns.lab", но недоступна зона "newdns.lab".

