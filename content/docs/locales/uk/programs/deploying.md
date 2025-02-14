---
title: "Розгортання програм"
description:
  Розгортання програм на ланцюгу можна здійснити за допомогою Solana CLI,
  використовуючи Upgradable BPF loader для завантаження скомпільованого
  байт-коду на блокчейн Solana.
sidebarSortOrder: 2
---

Програми Solana зберігаються в "виконуваних" акаунтах в мережі. Ці акаунти
містять скомпільований байт-код програми, який визначає інструкції, які
користувачі викликають для взаємодії з програмою.

## Команди CLI

Цей розділ призначений як довідник для базових команд CLI для створення та
розгортання програм Solana. Для покрокового посібника зі створення вашої першої
програми почніть з [Розробки програм на Rust](/docs/programs/rust).

### Збірка програми

Щоб побудувати вашу програму, використовуйте команду `cargo build-sbf`.

```shell
cargo build-sbf
```

Ця команда:

1. Скомпілює вашу програму
2. Створить директорію `target/deploy`
3. Згенерує файл `<program-name>.so`, де `<program-name>` відповідає імені вашої
   програми у файлі `Cargo.toml`

Вивантажений файл `.so` містить скомпільований байт-код вашої програми, який
буде збережено в акаунті Solana під час розгортання вашої програми.

### Розгортання програми

Щоб розгорнути вашу програму, використовуйте команду `solana program deploy`,
вказавши шлях до файлу `.so`, який був створений командою `cargo build-sbf`.

```shell
solana program deploy ./target/deploy/your_program.so
```

Під час періодів завантаження є кілька додаткових прапорців, які можна
використовувати для полегшення розгортання програми.

- `--with-compute-unit-price`: Встановіть ціну за обчислювальні одиниці для
  транзакції, в increments 0.000001 лампортів (мікро-лампорти) за обчислювальну
  одиницю.
- `--max-sign-attempts`: Максимальна кількість спроб підписати або повторно
  підписати транзакції після закінчення терміну дії blockhash. Якщо будь-які
  транзакції, надіслані під час розгортання програми, залишаються
  непідтвердженими після закінчення терміну дії початково вибраного останнього
  blockhash, ці транзакції будуть повторно підписані з новим blockhash і
  відправлені знову. Використовуйте цей параметр для налаштування максимальної
  кількості спроб підписання транзакцій. Кожен blockhash є дійсним близько 60
  секунд, що означає, що використання значення за замовчуванням 5 призведе до
  надсилання транзакцій щонайменше 5 хвилин або до тих пір, поки всі транзакції
  не будуть підтверджені, залежно від того, що відбудеться раніше. [за
  замовчуванням: 5]
- `--use-rpc`: Надсилайте транзакції запису до налаштованого RPC замість TPU
  валідатора. Цей прапорець вимагає RPC з урахуванням ставки.

Ви можете використовувати ці прапорці окремо або поєднувати їх разом. Наприклад:

```shell
solana program deploy ./target/deploy/your_program.so --with-compute-unit-price 10000 --max-sign-attempts 1000 --use-rpc
```

- Використовуйте
  [Priority Fee API від Helius](https://docs.helius.dev/guides/priority-fee-api)
  для отримання оцінки пріоритетної плати, яку потрібно встановити за допомогою
  прапорця `--with-compute-unit-price`.

- Отримайте
  [RPC з урахуванням ставки](https://solana.com/developers/guides/advanced/stake-weighted-qos)
  від [Helius](https://www.helius.dev/) або [Triton](https://triton.one/) для
  використання з прапорцем `--use-rpc`. Прапорець `--use-rpc` повинен
  використовуватись тільки з RPC з урахуванням ставки.

Щоб оновити ваш за умовчанням RPC URL за допомогою власної точки доступу RPC,
використовуйте команду `solana config set`.

```shell
solana config set --url <RPC_URL>
```

Ви можете переглянути список програм, які ви розгорнули, використовуючи
підкоманду `program show`:

```shell
solana program show --programs
```

Example output:

```
Program Id                                   | Slot      | Authority                                    | Balance
2w3sK6CW7Hy1Ljnz2uqPrQsg4KjNZxD4bDerXDkSX3Q1 | 133132    | 4kh6HxYZiAebF8HWLsUWod2EaQQ6iWHpHYCz8UcmFbM1 | 0.57821592 SOL
```

### Оновлення програми

Авторизація на оновлення програми може змінювати існуючу програму Solana,
розгортаючи новий файл `.so` на той самий ID програми.

Щоб оновити існуючу програму Solana:

- Змініть вихідний код вашої програми
- Виконайте команду `cargo build-sbf`, щоб згенерувати оновлений файл `.so`
- Виконайте команду `solana program deploy ./target/deploy/your_program.so`, щоб
  розгорнути оновлений файл `.so`

Авторизацію на оновлення можна змінити за допомогою підкоманди
`set-upgrade-authority` наступним чином:

```shell
solana program set-upgrade-authority <PROGRAM_ADDRESS> --new-upgrade-authority <NEW_UPGRADE_AUTHORITY>
```

### Незмінна програма

Програму можна зробити незмінною, видаливши її авторизацію на оновлення. Це
незворотна дія.

```shell
solana program set-upgrade-authority <PROGRAM_ADDRESS> --final
```

Ви можете вказати, що програма має бути незмінною при розгортанні, встановивши
прапорець `--final` під час розгортання програми.

```shell
solana program deploy ./target/deploy/your_program.so --final
```

### Закрити програму

Ви можете закрити свою програму Solana, щоб повернути SOL, виділені для акаунту.
Закриття програми є незворотним, тому це слід робити обережно. Щоб закрити
програму, використовуйте підкоманду `program close`. Наприклад:

```shell filename="Terminal"
solana program close 4Ujf5fXfLx2PAwRqcECCLtgDxHKPznoJpa43jUBxFfMz
--bypass-warning
```

Example output:

```
Closed Program Id 4Ujf5fXfLx2PAwRqcECCLtgDxHKPznoJpa43jUBxFfMz, 0.1350588 SOL
reclaimed
```

Зверніть увагу, що після закриття програми її ID програми не можна буде
використовувати знову. Спроба розгорнути програму з раніше закритим ID програми
призведе до помилки.

```
Error: Program 4Ujf5fXfLx2PAwRqcECCLtgDxHKPznoJpa43jUBxFfMz has been closed, use
a new Program Id
```

Якщо вам потрібно повторно розгорнути програму після її закриття, ви повинні
згенерувати новий ID програми. Щоб згенерувати нову ключову пару для програми,
виконайте наступну команду:

```shell filename="Terminal"
solana-keygen new -o ./target/deploy/your_program-keypair.json --force
```

Альтернативно, ви можете видалити існуючий файл ключової пари і знову виконати
команду `cargo build-sbf`, що згенерує новий файл ключової пари.

### Акаунти буфера програми

Розгортання програми вимагає кількох транзакцій через обмеження в 1232 байти для
транзакцій на Solana. Проміжним кроком процесу розгортання є запис байт-коду
програми в тимчасовий "акаунт буфера".

Цей акаунт буфера автоматично закривається після успішного розгортання програми.
Однак, якщо розгортання не вдалося, акаунт буфера залишається, і ви можете:

- Продовжити розгортання, використовуючи існуючий акаунт буфера
- Закрити акаунт буфера, щоб повернути виділений SOL (оренду)

Ви можете перевірити, чи є відкриті акаунти буфера, використовуючи підкоманду
`program show`, наступним чином:

```shell
solana program show --buffers
```

Example output:

```
Buffer Address                               | Authority                                    | Balance
5TRm1DxYcXLbSEbbxWcQbEUCce7L4tVgaC6e2V4G82pM | 4kh6HxYZiAebF8HWLsUWod2EaQQ6iWHpHYCz8UcmFbM1 | 0.57821592 SOL
```

Ви можете продовжити до розгортання за допомогою підкоманди `program deploy`
наступним чином:

```shell
solana program deploy --buffer 5TRm1DxYcXLbSEbbxWcQbEUCce7L4tVgaC6e2V4G82pM
```

Expected output on successful deployment:

```
Program Id: 2w3sK6CW7Hy1Ljnz2uqPrQsg4KjNZxD4bDerXDkSX3Q1

Signature: 3fsttJFskUmvbdL5F9y8g43rgNea5tYZeVXbimfx2Up5viJnYehWe3yx45rQJc8Kjkr6nY8D4DP4V2eiSPqvWRNL
```

To close buffer accounts, use the `program close` subcommand as follows:

```shell
solana program close --buffers
```

### ELF Dump

Внутрішні дані SBF shared object можна вивести в текстовий файл, щоб отримати
більше інформації про склад програми та те, що вона може виконувати під час
виконання. Вивантаження містить як ELF інформацію, так і список всіх символів та
інструкцій, що їх реалізують. Деякі з повідомлень журналу помилок BPF loader
будуть посилатися на конкретні номери інструкцій, де сталася помилка. Ці
посилання можна знайти у вивантаженні ELF, щоб ідентифікувати помилкову
інструкцію та її контекст.

```shell
cargo build-bpf --dump
```

Файл буде виведено до `/target/deploy/your_program-dump.txt`.

## Процес розгортання програми

Розгортання програми на Solana вимагає кількох транзакцій через максимальний
ліміт розміру транзакцій у 1232 байти. Solana CLI надсилає ці транзакції за
допомогою підкоманди `solana program deploy`. Процес можна розділити на наступні
3 фази:

1. [Ініціалізація буфера](https://github.com/solana-labs/solana/blob/7409d9d2687fba21078a745842c25df805cdf105/cli/src/program.rs#L2113):
   Спочатку CLI надсилає транзакцію, яка
   [створює акаунт буфера](https://github.com/solana-labs/solana/blob/7409d9d2687fba21078a745842c25df805cdf105/cli/src/program.rs#L1903)
   достатнього розміру для байт-коду, що розгортається. Також викликається
   [інструкція ініціалізації буфера](https://github.com/solana-labs/solana/blob/7409d9d2687fba21078a745842c25df805cdf105/programs/bpf_loader/src/lib.rs#L320),
   щоб встановити право власності на буфер і обмежити записи на вибрану адресу
   розробника.
2. [Запис буфера](https://github.com/solana-labs/solana/blob/7409d9d2687fba21078a745842c25df805cdf105/cli/src/program.rs#L2129):
   Після ініціалізації акаунту буфера CLI
   [розбиває байт-код програми](https://github.com/solana-labs/solana/blob/7409d9d2687fba21078a745842c25df805cdf105/cli/src/program.rs#L1940)
   на шматки розміром близько 1 КБ і
   [відправляє транзакції зі швидкістю 100 транзакцій за секунду](https://github.com/solana-labs/solana/blob/7409d9d2687fba21078a745842c25df805cdf105/client/src/tpu_client.rs#L133),
   щоб записати кожен шматок за допомогою
   [інструкції запису буфера](https://github.com/solana-labs/solana/blob/7409d9d2687fba21078a745842c25df805cdf105/programs/bpf_loader/src/lib.rs#L334).
   Ці транзакції надсилаються безпосередньо до порту обробки транзакцій (TPU)
   поточного лідера і обробляються паралельно. Після того, як всі транзакції
   будуть надіслані, CLI
   [опитує RPC API партіями підписів транзакцій](https://github.com/solana-labs/solana/blob/7409d9d2687fba21078a745842c25df805cdf105/client/src/tpu_client.rs#L216),
   щоб переконатися, що кожен запис був успішним і підтвердженим.
3. [Фіналізація](https://github.com/solana-labs/solana/blob/7409d9d2687fba21078a745842c25df805cdf105/cli/src/program.rs#L1807):
   Після завершення записів CLI
   [надсилає фінальну транзакцію](https://github.com/solana-labs/solana/blob/7409d9d2687fba21078a745842c25df805cdf105/cli/src/program.rs#L2150)
   для того, щоб
   [розгорнути нову програму](https://github.com/solana-labs/solana/blob/7409d9d2687fba21078a745842c25df805cdf105/programs/bpf_loader/src/lib.rs#L362)
   або
   [оновити існуючу програму](https://github.com/solana-labs/solana/blob/7409d9d2687fba21078a745842c25df805cdf105/programs/bpf_loader/src/lib.rs#L513).
   В будь-якому випадку байт-код, записаний в акаунт буфера, буде скопійовано в
   акаунт даних програми і перевірено.

## Оновлювана програма BPF Loader

Програма BPF loader є програмою, яка "володіє" всіма виконуваними акаунтами на
Solana. Коли ви розгортаєте програму, власник акаунту програми встановлюється на
програму BPF loader.

### Акаунти стану

Оновлювана програма BPF loader підтримує три різні типи акаунтів стану:

1. [Акаунт програми](https://github.com/solana-labs/solana/blob/master/sdk/program/src/bpf_loader_upgradeable.rs#L34):
   Це основний акаунт програми на ланцюгу, і його адреса зазвичай називається
   "ID програми". ID програми — це те, на що посилаються інструкції транзакцій
   для виклику програми. Акаунти програми незмінні після розгортання, тому ви
   можете вважати їх проксі-акаунтами для байт-коду та стану, що зберігаються в
   інших акаунтах.
2. [Акаунт даних програми](https://github.com/solana-labs/solana/blob/7409d9d2687fba21078a745842c25df805cdf105/sdk/program/src/bpf_loader_upgradeable.rs#L39):
   Цей акаунт зберігає виконуваний байт-код програми на ланцюгу. Коли програма
   оновлюється, дані цього акаунту оновлюються новим байт-кодом. Крім байт-коду,
   акаунти даних програми також відповідають за зберігання слоту, коли вони були
   востаннє змінені, та адреси єдиного акаунту, авторизованого для зміни акаунту
   (ця адреса може бути очищена, щоб зробити програму незмінною).
3. [Акаунти буфера](https://github.com/solana-labs/solana/blob/7409d9d2687fba21078a745842c25df805cdf105/sdk/program/src/bpf_loader_upgradeable.rs#L27):
   Ці акаунти тимчасово зберігають байт-код під час активного розгортання
   програми через серію транзакцій. Вони також зберігають адресу єдиного
   акаунту, який авторизований для виконання записів.

### Інструкції

Акаунти стану, перераховані вище, можуть бути змінені лише за допомогою однієї з
наступних інструкцій, що підтримуються програмою Upgradeable BPF Loader:

1. [Ініціалізація буфера](https://github.com/solana-labs/solana/blob/7409d9d2687fba21078a745842c25df805cdf105/sdk/program/src/loader_upgradeable_instruction.rs#L21):
   Створює акаунт буфера і зберігає адресу авторизації, яка дозволена змінювати
   буфер.
2. [Запис](https://github.com/solana-labs/solana/blob/7409d9d2687fba21078a745842c25df805cdf105/sdk/program/src/loader_upgradeable_instruction.rs#L28):
   Записує байт-код на вказаний байтовий офсет в акаунті буфера. Записи
   обробляються маленькими шматками через обмеження транзакцій Solana,
   максимальний розмір яких складає 1232 байти.
3. [Розгортання](https://github.com/solana-labs/solana/blob/7409d9d2687fba21078a745842c25df805cdf105/sdk/program/src/loader_upgradeable_instruction.rs#L77):
   Створює акаунт програми та акаунт даних програми. Він заповнює акаунт даних
   програми, копіюючи байт-код, що зберігається в акаунті буфера. Якщо байт-код
   є дійсним, акаунт програми буде встановлений як виконуваний, дозволяючи його
   викликати. Якщо байт-код недійсний, інструкція не вдасться, і всі зміни
   будуть скасовані.
4. [Оновлення](https://github.com/solana-labs/solana/blob/7409d9d2687fba21078a745842c25df805cdf105/sdk/program/src/loader_upgradeable_instruction.rs#L102):
   Заповнює існуючий акаунт даних програми, копіюючи виконуваний байт-код з
   акаунта буфера. Подібно до інструкції розгортання, вона буде успішною лише в
   тому випадку, якщо байт-код є дійсним.
5. [Встановлення авторизації](https://github.com/solana-labs/solana/blob/7409d9d2687fba21078a745842c25df805cdf105/sdk/program/src/loader_upgradeable_instruction.rs#L114):
   Оновлює авторизацію акаунта даних програми або акаунта буфера, якщо поточний
   власник акаунту підписав транзакцію, що обробляється. Якщо авторизація буде
   видалена без заміни, її можна буде встановити лише один раз і акаунт більше
   не можна буде закрити.
6. [Закриття](https://github.com/solana-labs/solana/blob/7409d9d2687fba21078a745842c25df805cdf105/sdk/program/src/loader_upgradeable_instruction.rs#L127):
   Очищає дані акаунта програми або акаунта буфера та повертає SOL, використані
   для депозиту звільнення від оренди.
