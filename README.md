# Thunderbird и Firefox - использование системного хранилища сертификатов ОС

## ОС Windows

Thunderbird и Firefox не используют по умолчанию системное хранилище сертификатов ОС Windows.

Использование системного хранилища сертификатов в Mozilla Thunderbird или Firefox может быть включено следующим образом:

1. Перейти в раздел "Настройки", выбрать пункт "Редактор настроек" (самый нижний пункт меню).

2. В строке поиска параметров указать имя `security.enterprise_roots.enabled`.

3. В значении параметра указать `true`.

4. Перезапустить приложение.

### Централизованная настройка

Вы можете поместить на клиентские машины следующие конфигурационные файлы, применив любую подходящую систему управления конфигурациями:

**Для Thunderbird**

1. Создать файл `C:\Program Files\Mozilla Thunderbird\defaults\pref\autoconfig.js` со следующим содержимым:
```bash
pref("general.config.filename", "thunderbird.cfg");
pref("general.config.obscure_value", 0);
```
2. Создать файл `C:\Program Files\Mozilla Thunderbird\thunderbird.cfg` со следующим содержимым:

```bash
// ВНИМАНИЕ: Первая строка данного конфигурационного файла всегда должна быть комментарием!
// defaultPref - опции, изменяемые пользователем
// lockPref - опции, неизменяемые пользователем
//
// Конфигурация стартовой страницы:
// lockPref("mailnews.start_page.enabled", true);
// lockPref("mailnews.start_page.url", "https://linux.org.ru/");
//
// Использовать системное хранилище сертификатов (certmgr.msc / certlm.msc) (false - отключено, true - включено)
lockPref("security.enterprise_roots.enabled", true);
```

**Для Firefox**

1. Создать файл `C:\Program Files (x86)\Mozilla Firefox\defaults\pref\autoconfig.js` со следующим содержимым:

```bash
C:\Program Files (x86)\Mozilla Firefox\defaults\pref\autoconfig.js
pref("general.config.filename", "firefox.cfg");
pref("general.config.obscure_value", 0);
```


2. Создать файл `C:\Program Files (x86)\Mozilla Firefox\firefox.cfg` со следующим содержимым:

```bash
C:\Program Files (x86)\Mozilla Firefox\firefox.cfg
// ВНИМАНИЕ: Первая строка данного конфигурационного файла всегда должна быть комментарием!
// defaultPref - опции, изменяемые пользователем
// lockPref - опции, неизменяемые пользователем
//
// Конфигурация стартовой страницы:
lockPref("browser.startup.homepage", "https://astralinux.ru/");
lockPref("browser.startup", "3");
// 0 - Начать с чистой страницы (about:blank).
// 1 - Начать с домашней страницы (Значение по умолчанию).
// 2 - Начать с последней посещенной страницы.
// 3 - Восстановить последнюю сессию браузера.
//
// Использовать системное хранилище сертификатов (certmgr.msc / certlm.msc) (false - отключено, true - включено)
lockPref("security.enterprise_roots.enabled", true);
```

## OC Debian / Astra Linux Special Edition

### Общие сведения

Штатная поддержка централизованного хранилища сертификатов в Mozilla Thunderbird/Firefox имеется только для ОС Windows и macOS. В ОС Linux, по заявлениям разработчиков, на текущий момент данный функционал к реализации не запланирован.

Однако имеется возможность настроить Mozilla Thunderbird/Firefox на совместную работу в связке с библиотекой `p11-kit-trust.so`, идущей в составе пакета `p11-kit-modules`. Это позволит использовать качестве хранилища сертификатов файл `/etc/ssl/certs/ca-certificates.crt`, содержимым которого можно управлять с помощью утилиты `update-ca-certificates`.

### Настройка

**Работа с утилитой ca-certificates**

1. Скопировать требуемый сертификат УЦ в каталог `/usr/local/share/ca-certificates` (имя файла обязательно должно заканчиваться на `.crt`):

```bash
cp TEST_CA.crt /usr/local/share/ca-certificates
```

2. Назначить права:

```bash
sudo chmod 644 /usr/local/share/ca-certificates/TEST_CA.crt
```

3. Выполнить обновление хранилища сертификатов:

```bash
sudo update-ca-certificates
```

Пример успешного вывода:

```bash
Updating certificates in /etc/ssl/certs...
rehash: warning: skipping ca-certificates.crt,it does not contain exactly one certificate or CRL
1 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
```

**Подключение библиотеки "p11-kit-trust.so" в Firefox/Thunderbird**

а) В ручном режиме:

`Настройки`, `Приватность и защита`, `Устройства защиты`, `Загрузить`. Требуется указать библиотеку `/usr/lib/x86_64-linux-gnu/pkcs11/p11-kit-trust.so` и произвольное имя, например `p11-kit_TrustedCertsStorage`.

б) В автоматическом режиме:

Создать файл с описанием политик `/usr/lib/firefox/distribution/policies.json`:

```bash
sudo touch /usr/lib/firefox/distribution/policies.json
```

Содержимое файла:

```bash
/usr/lib/firefox/distribution/policies.json
{
  "policies": {
    "SecurityDevices": {
        "p11-kit_TrustedCertsStorage": "/usr/lib/x86_64-linux-gnu/pkcs11/p11-kit-trust.so"
    }
  }
}
```



> [!NOTE]
> Для применения настроек требуется перезапуск браузера.
