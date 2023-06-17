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
`master-named.conf` - named.conf для сервера в роли `master`. <br/>
`named.ddns.lab` - файл зоны ddns.lab. <br/>
`named.dns.lab` - файл зоны dns.lab. <br/>
`named.dns.lab.client` - файл зоны dns.lab. <br/>
`named.dns.lab.rev` - файл зоны обратного просмотра. <br/>
`named.newdns.lab` - файл зоны  newdns.lab. <br/>
`named.zonetransfer.key` - файл, содержащий в себе "ключ", для доступа к копиям файлов зон основного сервера DNS. <br/>
`playbook.yml` - пьеса `Ansible`. Установка и настройка ПО, копирование конфигурационных файлов сервисов на ВМ. <br/>
`rndc.conf` - . <br/>
`servers-resolv.conf.j2` - jinja2 темплейт. Универсализация типовых шаблонов (в данном случае, файла `resolv.conf`). <br/>
`slave-named.conf` - named.conf для сервера в роли `slave`. <br/>
`zonetransfer.key` - файл, содержащий в себе "ключ", для доступа к копиям файлов зон основного сервера DNS. <br/>








