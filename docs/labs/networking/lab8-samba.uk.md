---
author: Wale Soyinka
contributors: Ganna Zhyrnova
tested on: Всі версії
tags:
  - samba
  - cifs
  - smbd
  - nmbd
  - smb.conf
  - smbpasswd
  - файлова система мережі
---

# Лабораторна робота 8: Samba

## Завдання

Виконавши цю лабораторну роботу, ви зможете

- встановити та налаштувати Samba
- обмінюватися файлами та каталогами між системами Linux за допомогою Samba
- використовувати звичайні утиліти Samba

Приблизний час виконання цієї лабораторної роботи: 40 хвилин

## Вступ

Samba дозволяє обмінюватися файлами та виконувати служби друку між системами Unix/Linux і Windows.

Samba — це реалізація «Загальної файлової системи Інтернету» (CIFS) із відкритим кодом. CIFS також називають серверним блоком повідомлень (SMB), менеджером локальної мережі або протоколом NetBIOS.
Сервер Samba складається з двох основних демонов – smbd і nmbd.

_smbd_: цей демон надає служби файлів і друку клієнтам SMB, таким як комп’ютери з різними операційними системами Microsoft.

_nmbd_: цей демон забезпечує обслуговування імен NETBIOS і підтримку перегляду.

Вправи в цій лабораторній роботі зосереджені на налаштуванні Samba як сервера та клієнта на сервері Rocky Linux.

## Завдання 1

### Встановіть Samba та налаштуйте базовий спільний каталог

#### Щоб встановити серверну програму Samba

0. Використовуйте утиліту dnf, щоб установити сервер Samba та пакет клієнта на вашому сервері.
   Впишіть:
   ```bash
   sudo dnf install -y samba
   ```

#### Щоб налаштувати Samba

1. Створіть каталог під назвою samba-share у папці /tmp, до якої потрібно надати спільний доступ. Впишіть:

   ```bash
   mkdir /tmp/samba-share
   ```

2. Давайте створимо базову конфігурацію Samba для спільного використання папки /tmp/samba-share.
   Зробіть це, створивши нове визначення спільного ресурсу у файлі конфігурації Samba:

   ```bash
   sudo tee -a /etc/samba/smb.conf << 'EOF'
   [Shared]
   path = /tmp/samba-share
   browsable = yes
   writable = yes
   EOF
   ```

#### Щоб запустити та включити службу Samba

1. Запустіть і ввімкніть служби Samba:

   ```bash
   sudo systemctl start smb nmb
   sudo systemctl enable smb nmb
   ```

2. Переконайтеся, що демони, які використовує служба Samba, запущені:

   ```bash
   sudo systemctl status smb nmb
   ```

## Завдання 2

### Користувачі Samba

Важливим і поширеним адміністративним завданням для керування сервером Samba є створення користувачів і паролів для користувачів, яким потрібен доступ до спільних ресурсів.

Ця вправа показує, як створювати користувачів Samba та налаштовувати облікові дані доступу для користувачів.

#### Щоб створити користувача Samba та пароль Samba

1. Спочатку створіть звичайного системного користувача з іменем sambarockstar. Впишіть:

   ```bash
   sudo useradd sambarockstar
   ```

2. Переконайтеся, що користувача було створено правильно. Впишіть:
   ```bash
   id sambarockstar
   ```

3. Додайте нового користувача системи sambarockstar до бази даних користувачів Samba та одночасно встановіть пароль для користувача Samba:

   ```bash
   sudo smbpasswd -a sambarockstar
   ```

   Коли буде запропоновано, введіть вибраний пароль і натисніть ENTER після кожного введення.

4. Перезапустіть служби Samba:
   ```bash
   sudo systemctl restart smb nmb
   ```

## Завдання 3

### Доступ до Samba Share (локальний тест)

У цій вправі ми спробуємо отримати доступ до нового ресурсу Samba з тієї ж системи. Це означає, що ми будемо використовувати той самий хост як сервер і клієнт.

#### Для встановлення клієнтських інструментів Samba

0. Встановіть клієнтські утиліти, запустивши:

   ```bash
   sudo dnf -y install cifs-utils
   ```

#### Щоб створити точку монтування Samba

0. Створіть точку монтування:
   ```bash
   mkdir ~/samba-client
   ```

#### Для локального монтування файлової системи SMB

1. Встановіть Samba Share локально:

   ```bash
   sudo mount -t cifs //localhost/Shared ~/samba-client -o user=sambarockstar
   ```

2. Використовуйте команду `mount`, щоб отримати список усіх змонтованих файлових систем типу CIFS. Впишіть:
   ```bash
   mount -t cifs
   ```
   Вихід
   ```bash
   //localhost/Shared on ~/samba-client type cifs (rw,relatime,vers=3.1.1,cache=strict,username=sambarockstar....
   ...<SNIP>...
   ```

3. Подібним чином скористайтеся командою `df`, щоб переконатися, що змонтований спільний ресурс доступний. Впишіть:

   ```bash
   df -t cifs
   ```

   Вихід

   ```
   Filesystem         1K-blocks     Used Available Use% Mounted on
   //localhost/Shared  73364480 17524224  55840256  24% ~/samba-client
   ```

4. Далі перелічіть вміст підключеного спільного ресурсу. Впишіть:

   ```bash
   ls ~/samba-client
   ```

5. Створіть тестовий файл у Share:

   ```bash
   touch ~/samba-client/testfile.txt
   ```

## Завдання 4

### Змінення дозволів на спільний доступ

#### Щоб налаштувати дозволи на спільний доступ

1. Зробити визначення спільних ресурсів samba «Shared» доступним лише для читання. Це можна зробити, змінивши значення параметра для запису з «так» на «ні» у файлі конфігурації smb.con. Давайте використаємо `sed` onliner, щоб виконати це, виконавши:

   ```bash
   sudo  sed -i'' -E \
    '/\[Shared\]/,+3 s/writable =.*$/writable = no/'  /etc/samba/smb.conf
   ```

2. Перезапустіть служби Samba:
   ```bash
   sudo systemctl restart smb nmb
   ```

3. Тепер перевірте запис до спільного ресурсу, спробувавши створити файл на змонтованому спільному ресурсі:

   ```bash
   touch ~/samba-client/testfile2.txt
   ```

## Завдання 5

### Використання Samba для певних груп користувачів

У цій вправі описано обмеження доступу до спільних ресурсів Samba через членство користувача в локальній групі. Це забезпечує зручний механізм для того, щоб зробити спільні ресурси доступними лише для певних груп користувачів.

#### Щоб створити нову групу для користувача Samba

1. Скористайтеся утилітою groupadd, щоб створити нову системну групу під назвою rockstars. Ми будемо використовувати цю групу в нашому прикладі для користувачів житлової системи, які мають доступ до певного ресурсу. Впишіть:
   ```bash
   sudo groupadd rockstars
   ```
2. Додайте існуючого користувача системи/Samba до групи. Впишіть:
   ```bash
   sudo usermod -aG rockstars sambarockstar
   ```

#### Щоб налаштувати дійсних користувачів у конфігурації Samba

1. Скористайтеся утилітою sed, щоб додати новий дійсний параметр користувача до визначення спільного доступу у файлі конфігурації Samba. Впишіть:
   ```bash
   sudo sed -i '/\[Shared\]/a valid users = @sambagroup' /etc/samba/smb.conf
   ```
2. Перезапустіть служби Samba:
   ```bash
   sudo systemctl restart smb nmb
   ```
3. Тепер протестуйте доступ до спільного ресурсу за допомогою sambarockstar і перевірте доступ.

## Завдання 6

Ця вправа імітує реальний сценарій, у якому ви діятимете як адміністратор клієнтської системи, а потім тестуватимете доступ до служби Samba на віддаленій системі (serverHQ), до якої у вас немає адміністративного доступу чи привілеїв.  Будучи студентом, ви налаштуєте клієнт Samba на своїй машині (serverXY) для доступу до служби Samba, розміщеної на іншій машині (serverHQ). Це відображає стандартні налаштування робочого місця.

Припущення:

- Ви не маєте кореневого доступу до serverHQ.
- Спільний ресурс Samba на serverHQ уже налаштований і доступний.

#### Щоб налаштувати клієнт Samba на сервері XY

Налаштуйте свою машину (serverXY) як клієнт Samba для доступу до спільного каталогу на окремому хості (serverHQ).

1. Переконайтеся, що необхідні утиліти клієнта Samba встановлені у вашій локальній системі.
   За потреби встановіть їх, виконавши:

   ```bash
   sudo dnf install samba-client cifs-utils -y
   ```

2. Створіть точку монтування на serverXY:

   ```bash
   mkdir ~/serverHQ-share
   ```

#### Щоб підключити Samba Share із serverHQ

Вам знадобиться IP-адреса або ім’я хоста serverHQ, ім’я спільного доступу та ваші облікові дані Samba.

Замініть serverHQ, sharedFolder і yourUsername на фактичні значення.

````
```bash
sudo mount -t cifs //serverHQ/sharedFolder ~/serverHQ-share -o user=yourUsername
```
````

#### Для перевірки та доступу до змонтованого спільного ресурсу

1. Перевірте, чи спільний каталог із serverHQ успішно змонтовано на вашій машині:

   ```bash
   ls ~/serverHQ-share
   ```

2. Спробуйте отримати доступ і змінити файли в підключеному спільному ресурсі. Наприклад, щоб створити новий файл:

   ```bash
   touch ~/serverHQ-share/newfile.txt
   ```

#### Щоб відключити віддалений спільний доступ

Після цього відключіть спільний доступ:

````
```bash
sudo umount ~/serverHQ-share
```
````