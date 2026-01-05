### Альт Сервер 10

Запускаем установку и выбираем Альт Сервер который будем устанавливать

<img width="925" height="482" alt="image" src="https://github.com/user-attachments/assets/f23400f0-f0a8-43f9-8858-995b25aa1bd5" />

Отмечаем галочки, что мы будем устанавливать

<img width="1278" height="767" alt="image" src="https://github.com/user-attachments/assets/696059a3-707e-4d59-a3f4-9424f091123c" />

<img width="1278" height="766" alt="image" src="https://github.com/user-attachments/assets/79014f2c-c367-4c28-ba33-d9dc47519915" />

В конце установки выбираем диск и по желанию устанавливаем пароль на загрузчик

<img width="1274" height="767" alt="image" src="https://github.com/user-attachments/assets/905be8cc-6149-4c54-a0fa-e28210c81704" />

Далее, настраиваем статический адрес

<img width="1277" height="767" alt="image" src="https://github.com/user-attachments/assets/009cf203-9969-4ac5-af28-6e81417562f6" />

Устанавливаем пароль локального администратора

<img width="1274" height="767" alt="image" src="https://github.com/user-attachments/assets/e414ed42-1c99-4058-a292-bae469f2eb1b" />

Создаём дополнительную учетную запись

<img width="1277" height="765" alt="image" src="https://github.com/user-attachments/assets/e418d1bb-894d-4f59-92eb-0ea5e7799c71" />

Завершение установки

<img width="1277" height="764" alt="image" src="https://github.com/user-attachments/assets/83c689f3-e4a9-4824-965e-6adedce5a5b6" />

Заходим на сервер

<img width="910" height="614" alt="image" src="https://github.com/user-attachments/assets/7c7d7afb-e009-4875-916f-fd637689da17" />

<img width="1281" height="634" alt="image" src="https://github.com/user-attachments/assets/f00153a5-39db-48d7-9413-6a658bf8f2f7" />

<img width="915" height="839" alt="image" src="https://github.com/user-attachments/assets/8b162b29-7ed6-4171-90c1-0f1f8c82fd86" />

Далее проверка работоспособности домена

Настройка Kerberos

На клиентском компьютере
```bash
# nano /etc/krb5.conf
проверяем содержимое, если не правильно правим под свой домен
includedir /etc/krb5.conf.d/

[logging]
# default = FILE:/var/log/krb5libs.log
# kdc = FILE:/var/log/krb5kdc.log
# admin_server = FILE:/var/log/kadmind.log

[libdefaults]
 dns_lookup_kdc = true
 dns_lookup_realm = false
 ticket_lifetime = 24h
 renew_lifetime = 7d
 forwardable = true
 rdns = false
 default_realm = TEST.ALT
# default_ccache_name = KEYRING:persistent:%{uid}

[realms]
TEST.ALT = {
  default_domain = test.alt
}

[domain_realm]
dc01 = DC.COMPANY.LOCAL
```
На сервере

<img width="916" height="377" alt="image" src="https://github.com/user-attachments/assets/05e85eec-b717-45e9-ab26-6532f2c6a3e3" />

Для ввода компьютера в Active Directory потребуется установить пакет task-auth-ad-sssd и все его зависимости (если он еще не установлен):
```bash
# apt-get install task-auth-ad-sssd
```
Синхронизация времени с контроллером домена производится автоматически.

Далее можно взять инструкцию от Альт Линукс и вводдить по ней

https://docs.altlinux.org/ru-RU/alt-server/11.1/html/alt-server/sambadc--chapter.html

https://docs.altlinux.org/ru-RU/alt-domain/11.0/html/alt-domain/index.html

Просмотр общей информации о домене:
```bash
# samba-tool domain info 127.0.0.1
Forest           : test.alt
Domain           : test.alt
Netbios domain   : TEST
DC name          : dc1.test.alt
DC netbios name  : DC
Server site      : Default-First-Site-Name
Client site      : Default-First-Site-Name
```
Просмотр предоставляемых служб:
```bash
# smbclient -L localhost -Uadministrator
Password for [TEST\administrator]:

	Sharename       Type      Comment
	---------       ----      -------
	sysvol          Disk
	netlogon        Disk
	IPC$            IPC       IPC Service (Samba 4.21.9-alt1)
SMB1 disabled -- no workgroup available
```
Общие ресурсы netlogon и sysvol создаваемые по умолчанию нужны для функционирования сервера и создаются в smb.conf в процессе развертывания/модернизации.<br>
Проверка конфигурации DNS:

Убедиться в наличии nameserver 127.0.0.1 в /etc/resolv.conf:
```bash
# cat /etc/resolv.conf
nameserver 127.0.0.1
search test.alt
```
```bash
# host test.alt
test.alt has address 192.168.0.132
test.alt has IPv6 address fd47:d11e:43c1:0:a00:27ff:fe49:2df
```
Проверить имена хостов:
```bash
# host -t SRV _kerberos._udp.test.alt.
_kerberos._udp.test.alt has SRV record 0 100 88 dc1.test.alt
# host -t SRV _ldap._tcp.test.alt.
_ldap._tcp.test.alt has SRV record 0 100 389 dc1.test.alt.
# host -t A dc1.test.alt.
dc1.test.alt has address 192.168.0.132
```
> Если имена не находятся, необходимо проверить выключение службы named.

Проверка Kerberos (имя домена должно быть в верхнем регистре):
```bash
# kinit administrator@TEST.ALT
Password for administrator@TEST.ALT:
```
Просмотр полученного билета:
```bash
# klist
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: administrator@TEST.ALT

Valid starting       Expires              Service principal
19.08.2025 17:13:17  20.08.2025 03:13:17  krbtgt/TEST.ALT@TEST.ALT
	renew until 20.08.2025 17:13:14
```
