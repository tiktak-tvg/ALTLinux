
```
# apt install lxde
```

```
# sudo apt install xrdp
```

Настройки сервера хранятся в файле /etc/xrdp/sesman.ini. Некоторые настройки сервера установленные по умолчанию:

- AllowRootLogin=true — авторизация Root;
- MaxLoginRetry=4 — максимальное количество попыток подключения;
- TerminalServerUsers=tsusers — группа, в которую необходимо добавить пользователей для организации доступа к серверу;
- MaxSessions=50 — максимальное количество подключений к серверу;
- KillDisconnected=false — разрыв сеанса при отключении пользователя;
- FuseMountName=Mount_FOLDER — название монтируемой папки.

*По умолчанию для подключения по RDP используется порт 3389. Номер порта можно изменить в файле /etc/xrdp/xrdp.ini.*

```
# nano /etc/xrdp/startwm.sh
```

```
# debian, alt
#  if [ -r /etc/X11/Xsession ]; then
#    pre_start

#   . /etc/X11/Xsession
#   post_start
#   exit 0
# fi
```

<img width="675" height="651" alt="image" src="https://github.com/user-attachments/assets/c1892253-7273-4513-9c98-756fc3e7e697" /><br>
<img width="672" height="484" alt="image" src="https://github.com/user-attachments/assets/382497af-b68a-4a98-ac56-36e667ab3161" /><br>
<img width="579" height="637" alt="image" src="https://github.com/user-attachments/assets/836650a3-969e-4300-a7f4-b02f808c39a3" /><br>

```
lxsession -s LXDE -e LXDE
```

Установить пакет xrdp:

```
# apt-get install xrdp
```
Включить и добавить в автозапуск сервисы:

```
# systemctl enable --now xrdp xrdp-sesman
```

Права доступа пользователя:

Для доступа к терминальному сеансу — включить в группу tsusers:

```
# gpasswd -a user tsusers
```

Для проброса папки — включить в группу fuse:

```
# gpasswd -a user fuse
```
<img width="490" height="857" alt="image" src="https://github.com/user-attachments/assets/b99232df-07ea-412c-a51d-2d81be72194c" /><br>
<img width="1054" height="608" alt="image" src="https://github.com/user-attachments/assets/2b2c32ab-0a73-48ac-9b0e-8a00842e718c" /><br>
Для подключения можно использовать FreeRDP — клиент для подключения к удаленному рабочему столу по протоколу RDP.

Установить пакет xfreerdp:

```
# apt-get install xfreerdp
```

Описание некоторых параметров:

/v:<сервер>[:порт] — IP-адрес или имя сервера;
/u:<пользователь> — имя пользователя;
/p:<пароль> — пароль пользователя;
/w:<ширина> — ширина окна;
/h:<высота> — высота окна;
/f — полноэкранный режим;
/size:<ширина>x<высота> — размер окна;
/drive:<название>,<путь> — подключение каталога.

xfreerdp /drive:Epson,/home/cas/epson /v:10.4.129.129 /u:user /p:123

где:

- Epson — название папки, которая будет показываться в каталоге thinclient_drives в домашней папке терминального пользователя, у локального пользователя пробрасывается папка /home/cas/epson;
- 10.4.129.129 — адрес терминального сервера;
- user — имя терминального пользователя;
- 123 — пароль терминального пользователя.

<img width="575" height="539" alt="image" src="https://github.com/user-attachments/assets/0d06440d-e80c-4a10-91f1-6afb4600f3a4" /><br>

📥 [Xrdp](https://wiki.altlinux.org/Xrdp)

📥 [Remmina](https://www.altlinux.org/Education_applications/Remmina)

📥 [Remmina](https://www.altlinux.org/Remmina)

📥 [VNC](https://www.altlinux.org/VNC)


