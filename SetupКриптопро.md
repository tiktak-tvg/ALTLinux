#### Устанавливаем Криптопро в ручную.

Перед установкой запустите обновление пакетов, зависимостей.

```
epm update && epm upgrade  или apt-get update && apt-get upgrade 
```

Скачиваем с сайта ``https://cryptopro.ru/`` в ручную.

- КриптоПро ЭЦП Browser plug-in
- КриптоПро CSP<br>
с расширением *.rpm

Или заходим на сайт<br>
*Проверка создания электронной подписи CAdES-BES*<br>
```
https://www.cryptopro.ru/sites/default/files/products/cades/demopage/cades_bes_sample.html
```

и на нем автоматически предлагается загрузить не достающие плагины. 

<img width="992" height="472" alt="image" src="https://github.com/user-attachments/assets/9417c3b7-c2bd-48f2-8f5b-6b3f912fec8c" />

Переходим в папку куда скачали ахив для установки ``КриптоПро ЭЦП Browser plug-in`` ``КриптоПро CSP``<br>
Нам понадобяться три папки с такими названиями  ``cades-linux-amd64``, ``linux-amd64`` и предлагаемый браузер ``chromium-gost``.<br>
Ссылка на закачку ``https://cryptopro.ru/products/chromium-gost`` для Линукс с расширение ``*.rpm``.
Распакуем их.<br>

<img width="1348" height="259" alt="image" src="https://github.com/user-attachments/assets/2cd5959a-9976-413d-a04e-a0b280f53626" />


Далее открываем сайт: ссылка такая ``https://www.altlinux.org/%D0%9A%D1%80%D0%B8%D0%BF%D1%82%D0%BE%D0%9F%D1%80%D0%BE``, если не откроется найдите по названию:

<img width="791" height="306" alt="image" src="https://github.com/user-attachments/assets/bbe73afb-0f31-477f-b1e4-08df02d0d1bd" />


Переходим в терменале ``term`` где распакована папка ``linux-amd64``<br>
```
cd /home/user/linux-amd64/
```
<img width="1304" height="745" alt="image" src="https://github.com/user-attachments/assets/739dcc7b-bc62-4bce-b1a3-538e1311f280" />



И с сайта который открыли копируем команды:

**установите базовые пакеты:** 

``apt-get install cprocsp-curl* lsb-cprocsp-base* lsb-cprocsp-capilite* lsb-cprocsp-kc1-64* lsb-cprocsp-rdr-64*``

<img width="1327" height="374" alt="image" src="https://github.com/user-attachments/assets/0498a231-b53c-4f57-85ae-72dbe4aed710" />

**установите пакеты для поддержки токенов (Рутокен S и Рутокен ЭЦП):**

``apt-get install cprocsp-rdr-gui-gtk* cprocsp-rdr-rutoken* cprocsp-rdr-pcsc* lsb-cprocsp-pkcs11* pcsc-lite-rutokens pcsc-lite-ccid``

<img width="1324" height="737" alt="image" src="https://github.com/user-attachments/assets/b89f97c7-f841-45ed-a5a7-01d948cf13b4" />

**Для установки сертификатов Главного удостоверяющего центра:**

``apt-get install lsb-cprocsp-ca-certs*``

<img width="1326" height="235" alt="image" src="https://github.com/user-attachments/assets/8bb42d57-0b40-4000-b11f-f6f2727635bd" />

**Если есть потребность в установке графических Инструментов КриптоПро:** 

``apt-get install cprocsp-cptools*``

<img width="1324" height="239" alt="image" src="https://github.com/user-attachments/assets/a7e11ec1-1dbf-4743-be1f-38839c3a7931" />


Далее, открываем папку /home/user/cades-linux-amd64/ удаляем файлы с расширением *.deb и запускаем установку трёх файлов с расширением *.rpm

<img width="526" height="144" alt="image" src="https://github.com/user-attachments/assets/9a48abea-e099-4e1a-81df-0db007ac673a" />


*P.s.В папке home/user/linux-arm64/./install_gui.sh предлагают автоматическую установку через скрипт, на этом же сайте есть инструкция, но мы и так всё уже установили.
Поэтому запускать не будем.*

Теперь проверяем установку.<br>
Проверка создания электронной подписи CAdES-BES<br>
```
https://www.cryptopro.ru/sites/default/files/products/cades/demopage/cades_bes_sample.html
```

<img width="1156" height="502" alt="image" src="https://github.com/user-attachments/assets/eab7d29d-c41b-48ca-ac62-2ead52fa1d1b" />


Так будет выглядеть с ключиком ``Контур`` например 

<img width="983" height="1020" alt="image" src="https://github.com/user-attachments/assets/980a67c2-d7c9-4a65-b557-bc0187d6d126" />


> [!Warning]
> В OS Windows ключ находит КриптоПро сам, в OS Linux его нужно добавить в КриптоПро вручную, чтобы ЭЦП появился на электронной площадке

##### Ошибки и исправления.

```
https://www.altlinux.org/%D0%9A%D1%80%D0%B8%D0%BF%D1%82%D0%BE%D0%9F%D1%80%D0%BE#%D0%98%D0%B7%D0%B2%D0%B5%D1%81%D1%82%D0%BD%D1%8B%D0%B5_%D0%BE%D1%88%D0%B8%D0%B1%D0%BA%D0%B8_%D0%B8_%D0%BC%D0%B5%D1%82%D0%BE%D0%B4%D1%8B_%D0%B8%D1%81%D0%BF%D1%80%D0%B0%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D1%8F
```

Не забудьте добавить в настройке список доверенных сайтов.

```
https://chrome-extension://epebfcehmdedogndhlcacafjaacknbcm/trusted_sites.html
```
<img width="1326" height="600" alt="image" src="https://github.com/user-attachments/assets/238fadc3-c483-4c7b-9a0d-10deb30a9b93" />

