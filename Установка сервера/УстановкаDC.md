### Альт Сервер 10

Запускаем установку и выбираем Альт Сервер который будем устанавливать

<img width="925" height="482" alt="image" src="https://github.com/user-attachments/assets/994d9dca-7143-440e-8f99-82782ecb598a" />


<img width="1277" height="767" alt="image" src="https://github.com/user-attachments/assets/0697e65c-6462-4b31-b51b-c56db262d0a5" />


<img width="1275" height="767" alt="image" src="https://github.com/user-attachments/assets/5b258b54-78f7-43ce-a052-9dc021da0e3b" />


Отмечаем галочки, что мы будем устанавливать

<img width="1279" height="766" alt="image" src="https://github.com/user-attachments/assets/25a9f9de-8da8-4cb5-a55f-d17de823108f" />

<img width="1278" height="767" alt="image" src="https://github.com/user-attachments/assets/93f41f58-407c-41d1-9964-726aca50172d" />

<img width="1278" height="766" alt="image" src="https://github.com/user-attachments/assets/ebbf785c-91c2-4024-a0f4-d4deafc38744" />


В конце установки выбираем диск и по желанию устанавливаем пароль на загрузчик

<img width="1274" height="767" alt="image" src="https://github.com/user-attachments/assets/90c7c0b2-0e93-4fc4-aaf4-d972e051b863" />


Далее, настраиваем статический адрес

<img width="1277" height="767" alt="image" src="https://github.com/user-attachments/assets/b3bcd088-0eed-421c-bc5d-a24fff283bfe" />


Устанавливаем пароль локального администратора

<img width="1274" height="767" alt="image" src="https://github.com/user-attachments/assets/2be5ea97-a613-47cf-97ee-b5a36cf5df9f" />


Создаём дополнительную учетную запись

<img width="1277" height="765" alt="image" src="https://github.com/user-attachments/assets/a91576dd-4d58-48ab-b0af-e6601d5d687e" />


Завершение установки

<img width="1277" height="764" alt="image" src="https://github.com/user-attachments/assets/1b5393e4-450b-40c0-805e-c7698b055e4d" />


Заходим на сервер

<img width="910" height="614" alt="image" src="https://github.com/user-attachments/assets/f884f18f-8dd1-412a-840d-0a80253b58dd" />

<img width="1281" height="634" alt="image" src="https://github.com/user-attachments/assets/fe92ffd4-736f-4eed-832f-815e0b1e7132" />

<img width="915" height="839" alt="image" src="https://github.com/user-attachments/assets/1af336ee-b9da-4c8e-8962-39a45c7e8092" />


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

<img width="916" height="377" alt="image" src="https://github.com/user-attachments/assets/67a9e667-0dbf-4869-abca-d887061b27b0" />

Для ввода компьютера в Active Directory потребуется установить пакет task-auth-ad-sssd и все его зависимости (если он еще не установлен):
```bash
# apt-get install task-auth-ad-sssd
```
Синхронизация времени с контроллером домена производится автоматически.

Далее можно воспользоваться инструкцией от Альт Линукс, как поднимать контроллер домена.

 👻 [Ссылка 1](https://docs.altlinux.org/ru-RU/alt-server/11.1/html/alt-server/sambadc--chapter.html)

 👻 [Ссылка 2](https://docs.altlinux.org/ru-RU/alt-domain/11.0/html/alt-domain/index.html)

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
