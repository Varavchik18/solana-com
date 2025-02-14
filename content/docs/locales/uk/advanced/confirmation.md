---
sidebarSortOrder: 1
sidebarLabel: "Підтвердження та закінчення строку дії"
title: "Підтвердження транзакції та закінчення строку дії"
seoTitle: "Підтвердження транзакції та закінчення строку дії"
description:
  "Дізнайтеся, як працює підтвердження транзакцій у Solana і коли транзакція
  завершує дію (включаючи перевірку останніх blockhash)."
altRoutes:
  - /docs/uk/advanced
  - /docs/uk/core/transactions/confirmation
---

Проблеми, пов'язані з
[підтвердженням транзакції](/docs/uk/terminology.md#transaction-confirmations),
є поширеними серед нових розробників під час створення застосунків. Ця стаття
має на меті підвищити загальне розуміння механізму підтвердження, який
використовується в блокчейні Solana, включаючи деякі рекомендовані найкращі
практики.

## Короткий вступ до транзакцій

Перед тим як розглянути, як працює підтвердження та закінчення строку дії
транзакцій у Solana, давайте коротко розглянемо кілька основ:

- що таке транзакція,
- життєвий цикл транзакції,
- що таке blockhash,
- і коротке ознайомлення з Proof of History (PoH) і тим, як це стосується
  blockhash.

### Що таке транзакція?

Транзакції складаються з двох компонентів:
[повідомлення](/docs/uk/terminology.md#message) і
[списку підписів](/docs/uk/terminology.md#signature). Повідомлення транзакції
містить основну інформацію та складається з чотирьох компонентів:

- **заголовок** з метаданими про транзакцію,
- **список інструкцій** для виконання,
- **список акаунтів**, які потрібно завантажити,
- та **“недавній blockhash”**.

У цій статті ми зосередимося на
[недавньому blockhash](/docs/uk/terminology.md#blockhash), оскільки він відіграє
важливу роль у підтвердженні транзакції.

### Життєвий цикл транзакції

Нижче наведено основні етапи життєвого циклу транзакції. У цій статті
розглядаються всі етапи, крім 1 і 4.

1. Створення заголовка та списку інструкцій разом із переліком акаунтів, які
   потрібно прочитати або записати.
2. Отримання недавнього blockhash та його використання для підготовки
   повідомлення транзакції.
3. Симуляція транзакції, щоб переконатися, що вона поводиться очікувано.
4. Запит користувача на підписання підготовленого повідомлення транзакції за
   допомогою його приватного ключа.
5. Відправка транзакції до RPC-вузла, який намагається передати її поточному
   виробнику блоків.
6. Очікування, що виробник блоку перевірить і додасть транзакцію до свого блоку.
7. Підтвердження того, що транзакція або була включена в блок, або закінчився її
   термін дії.

### Що таке Blockhash?

[“Blockhash”](/docs/uk/terminology.md#blockhash) означає останній Proof of
History (PoH) хеш для [“слота”](/docs/uk/terminology.md#slot) (опис нижче).
Оскільки Solana використовує PoH як довірений годинник, недавній blockhash
транзакції можна розглядати як **мітку часу**.

### Коротке нагадування про Proof of History

Механізм Proof of History у Solana використовує довгий ланцюг рекурсивних хешів
SHA-256 для створення довіреного годинника. Назва “історія” походить від того,
що виробники блоків додають хеш ідентифікаторів транзакцій у потік, щоб
зафіксувати, які транзакції були оброблені в їхньому блоці.

[Обчислення PoH-хешу](https://github.com/anza-xyz/agave/blob/aa0922d6845e119ba466f88497e8209d1c82febc/entry/src/poh.rs#L79):
`next_hash = hash(prev_hash, hash(transaction_ids))`.

PoH може використовуватися як довірений годинник, оскільки кожен хеш має бути
створений послідовно. Кожен згенерований блок містить blockhash і список
контрольних точок хешів, званих “ticks,” які дозволяють валідаторам перевіряти
весь ланцюг хешів паралельно та доводити, що певний час дійсно минув.

## Закінчення строку дії транзакції

За замовчуванням усі транзакції Solana закінчують строк дії, якщо їх не включено
в блок протягом певного періоду часу. **Переважна більшість** проблем із
підтвердженням транзакцій пов'язана з тим, як RPC-вузли та валідатори виявляють
і обробляють **прострочені** транзакції. Чітке розуміння механізму закінчення
строку дії транзакції допоможе вам діагностувати більшість проблем із
підтвердженням транзакцій.

### Як працює закінчення строку дії транзакції?

Кожна транзакція містить “недавній blockhash,” який використовується як часовий
штамп PoH і закінчується, коли цей blockhash більше не вважається “достатньо
недавнім.”

Коли кожен блок завершується (тобто досягається максимальна висота тиків,
[визначена](https://github.com/anza-xyz/agave/blob/0588ecc6121ba026c65600d117066dbdfaf63444/runtime/src/bank.rs#L3269-L3271),
до "кордону блоку"), фінальний хеш блоку додається до `BlockhashQueue`, яка
зберігає максимум
[300 найновіших blockhash](https://github.com/anza-xyz/agave/blob/e0b0bcc80380da34bb63364cc393801af1e1057f/sdk/program/src/clock.rs#L123-L126).
Під час обробки транзакцій валідатори Solana перевіряють, чи зберігається
недавній blockhash транзакції серед останніх 151 записаних хешів (так званий
"максимальний вік обробки"). Якщо недавній blockhash транзакції
[старший за цей вік](https://github.com/anza-xyz/agave/blob/cb2fd2b632f16a43eff0c27af7458e4e97512e31/runtime/src/bank.rs#L3570-L3571),
транзакція не обробляється.

> Завдяки поточному
> [максимальному віку обробки, що становить 150](https://github.com/anza-xyz/agave/blob/cb2fd2b632f16a43eff0c27af7458e4e97512e31/sdk/program/src/clock.rs#L129-L131)
> і тому, що "вік" blockhash у черзі є
> [нульовим індексом](https://github.com/anza-xyz/agave/blob/992a398fe8ea29ec4f04d081ceef7664960206f4/accounts-db/src/blockhash_queue.rs#L248-L274),
> фактично є 151 blockhash, які вважаються "достатньо недавніми" і дійсними для
> обробки.

Оскільки [слоти](/docs/uk/terminology.md#slot) (тобто періоди часу, протягом
яких валідатор може створити блок) налаштовані на тривалість близько
[400 мс](https://github.com/anza-xyz/agave/blob/cb2fd2b632f16a43eff0c27af7458e4e97512e31/sdk/program/src/clock.rs#L107-L109),
але можуть коливатися між 400 мс і 600 мс, певний blockhash можна
використовувати в транзакціях приблизно протягом 60–90 секунд, перш ніж він
вважатиметься простроченим у середовищі виконання.

### Приклад закінчення строку дії транзакції

Розгляньмо швидкий приклад:

1. Валідатор активно створює новий блок для поточного слота.
2. Валідатор отримує транзакцію від користувача з недавнім blockhash `abcd...`.
3. Валідатор перевіряє цей blockhash `abcd...` у списку недавніх blockhash у
   `BlockhashQueue` і виявляє, що він був створений 151 блок тому.
4. Оскільки йому рівно 151 блок, транзакція ще не прострочена та може бути
   оброблена.
5. Але зачекайте: перед обробкою транзакції валідатор завершує створення
   наступного блоку та додає його до `BlockhashQueue`. Потім валідатор починає
   створювати блок для наступного слота (валідатори можуть створювати блоки
   протягом 4 послідовних слотів).
6. Валідатор перевіряє ту ж транзакцію ще раз і виявляє, що їй тепер 152 блоки.
   Вона відхиляється, оскільки занадто стара. :(

## Чому транзакції закінчують строк дії?

Це робиться для того, щоб валідатори могли уникнути повторної обробки тієї ж
транзакції.

Наївний підхід для запобігання повторній обробці полягав би в перевірці кожної
нової транзакції з усією історією транзакцій блокчейна. Але завдяки тому, що
транзакції закінчують строк дії за короткий період часу, валідаторам потрібно
перевіряти лише відносно невеликий набір **недавніх** транзакцій.

### Other blockchains

Solana's approach to prevent double processing is quite different from other
blockchains. For example, Ethereum tracks a counter (nonce) for each transaction
sender and will only process transactions that use the next valid nonce.

Ethereum's approach is simple for validators to implement, but it can be
problematic for users. Many people have encountered situations when their
Ethereum transactions got stuck in a _pending_ state for a long time and all the
later transactions, which used higher nonce values, were blocked from
processing.

### Інші блокчейни

Підхід Solana до запобігання повторній обробці транзакцій значно відрізняється
від інших блокчейнів. Наприклад, Ethereum використовує лічильник (nonce) для
кожного відправника транзакцій і обробляє лише транзакції, які використовують
наступний дійсний nonce.

Підхід Ethereum є простим для реалізації валідаторами, але може викликати
проблеми для користувачів. Багато хто стикався із ситуаціями, коли їхні
транзакції Ethereum залишалися в стані **очікування** протягом тривалого часу, а
всі подальші транзакції з більшими значеннями nonce блокувалися.

### Переваги на Solana

Solana має кілька переваг у цьому підході:

1. Один платник комісії може надсилати кілька транзакцій одночасно, які можуть
   оброблятися в будь-якому порядку. Це може статися, якщо ви використовуєте
   кілька додатків одночасно.
2. Якщо транзакція не була додана до блоку та закінчила строк дії, користувачі
   можуть спробувати знову, знаючи, що їхня попередня транзакція **ніколи** не
   буде оброблена.

Відсутність використання лічильників робить досвід роботи з Solana більш
зрозумілим для користувачів, оскільки вони швидше отримують результат — успіх,
невдачу або закінчення строку дії — та уникають тривалого стану очікування.

### Недоліки на Solana

Звісно, є й недоліки:

1. Валідатори повинні активно відстежувати набір усіх оброблених ідентифікаторів
   транзакцій, щоб запобігти їх повторній обробці.
2. Якщо час закінчення строку дії надто короткий, користувачі можуть не
   встигнути надіслати свої транзакції до того, як вони стануть недійсними.

Ці недоліки показують компроміс у тому, як налаштовано строк дії транзакцій.
Якщо збільшити час закінчення строку дії, валідаторам доведеться використовувати
більше пам’яті для відстеження транзакцій. Якщо ж скоротити строк дії,
користувачам буде важче встигнути надіслати свої транзакції.

Наразі кластери Solana вимагають, щоб транзакції використовували blockhash, який
не старший за 151 блок.

> У цьому [питанні GitHub](https://github.com/solana-labs/solana/issues/23582)
> надані розрахунки, які оцінюють, що валідаторам mainnet-beta потрібно близько
> 150 МБ пам’яті для відстеження транзакцій. У майбутньому це можна
> оптимізувати, не скорочуючи строк дії транзакцій, як зазначено в цьому
> обговоренні.

## Поради щодо підтвердження транзакцій

Як зазначалося раніше, blockhash закінчують строк дії через період, який може
тривати всього **одну хвилину**, коли слоти обробляються з цільовим часом 400
мс.

Одна хвилина — це не багато, враховуючи, що клієнту потрібно отримати недавній
blockhash, дочекатися підписання транзакції користувачем, а потім сподіватися,
що надіслана транзакція досягне лідера, який погодиться її обробити. Розглянемо
кілька порад, які допоможуть уникнути невдач підтвердження через закінчення
строку дії транзакцій.

### Отримуйте blockhash із відповідним рівнем підтвердження

З огляду на короткий період закінчення строку дії, клієнти та додатки повинні
допомагати користувачам створювати транзакції з blockhash, який є максимально
недавнім.

Коли ви отримуєте blockhash, рекомендованим RPC API є
[`getLatestBlockhash`](/docs/uk/rpc/http/getLatestBlockhash.mdx). За
замовчуванням цей API використовує рівень підтвердження `finalized`, щоб
повернути blockhash останнього фіналізованого блоку. Проте ви можете змінити цю
поведінку, встановивши параметр `commitment` на інший рівень підтвердження.

**Рекомендація**

Рівень підтвердження `confirmed` майже завжди слід використовувати для запитів
RPC, оскільки він зазвичай лише на кілька слотів відстає від рівня `processed` і
має дуже низьку ймовірність належати до відхиленого
[форку](https://docs.anza.xyz/consensus/fork-generation).

Але ви також можете розглянути інші варіанти:

- Вибір `processed` дозволяє отримати найбільш недавній blockhash порівняно з
  іншими рівнями підтвердження, що дає більше часу на підготовку і обробку
  транзакції. Однак через поширеність форків у блокчейні Solana близько 5%
  блоків не потрапляють у фіналізований кластер, що створює ризик того, що ваша
  транзакція використовує blockhash, що належить до відкинутого форку.
  Транзакції з такими blockhash ніколи не будуть вважатися недавніми у
  фіналізованому блокчейні.
- Використання
  [рівня підтвердження за замовчуванням](/docs/uk/rpc#default-commitment)
  `finalized` виключає ризик того, що вибраний blockhash належатиме до
  відкинутого форку. Однак це має компроміс: зазвичай є щонайменше 32 слоти
  різниці між найбільш недавнім підтвердженим блоком і найбільш недавнім
  фіналізованим блоком. Це значно зменшує строк дії ваших транзакцій приблизно
  на 13 секунд, що може бути ще більш суттєвим за нестабільних умов кластеру.

### Використовуйте відповідний рівень підтвердження для preflight

Якщо ваша транзакція використовує blockhash, отриманий з одного RPC-вузла, але
надсилається або симулюється на іншому, можуть виникнути проблеми через затримку
одного з вузлів.

Коли RPC-вузли отримують запит `sendTransaction`, вони намагаються визначити
строк дії транзакції, використовуючи найбільш недавній фіналізований блок або
блок, вибраний параметром `preflightCommitment`. **Дуже поширена проблема**
виникає, якщо blockhash транзакції створений після блоку, який використовується
для обчислення строку дії. У такому випадку RPC-вузол лише один раз пересилає
транзакцію, а потім **видаляє** її.

Аналогічно, при отриманні запиту `simulateTransaction` RPC-вузли симулюють
транзакцію, використовуючи найбільш недавній фіналізований блок або блок,
вибраний параметром `preflightCommitment`. Якщо вибраний блок для симуляції
старіший за blockhash транзакції, симуляція завершиться помилкою “blockhash not
found”.

**Рекомендація**

Навіть якщо ви використовуєте `skipPreflight`, **завжди** встановлюйте параметр
`preflightCommitment` на той самий рівень підтвердження, який використовувався
для отримання blockhash транзакції, як для запитів `sendTransaction`, так і
`simulateTransaction`.

### Уникайте відставання RPC-вузлів під час надсилання транзакцій

Якщо ваш додаток використовує сервіс пулу RPC або якщо кінцева точка RPC
відрізняється між створенням транзакції та її надсиланням, будьте обережні зі
сценаріями, коли один RPC-вузол відстає від іншого. Наприклад, якщо blockhash
транзакції отримано з одного RPC-вузла, а транзакція надсилається на інший для
обробки або симуляції, другий RPC-вузол може відставати.

**Рекомендація**

Для запитів `sendTransaction` клієнти повинні повторно надсилати транзакцію до
RPC-вузла на частих інтервалах, щоб у випадку, якщо RPC-вузол трохи відстає від
кластеру, він врешті-решт виявив транзакцію та її строк дії.

Для запитів `simulateTransaction` клієнти повинні використовувати параметр
[`replaceRecentBlockhash`](/docs/uk/rpc/http/simulateTransaction.mdx), щоб
інструктувати RPC-вузол замінювати blockhash симульованої транзакції на
blockhash, який завжди буде дійсним для симуляції.

### Уникайте повторного використання застарілих blockhash

Навіть якщо ваш додаток отримав дуже недавній blockhash, переконайтеся, що ви не
використовуєте його надто довго. Ідеальний сценарій — отримати свіжий blockhash
безпосередньо перед підписанням транзакції.

**Рекомендація для додатків**

Часто запитуйте нові blockhash, щоб забезпечити готовність вашого додатка до
створення транзакції, коли користувач виконує дію.

**Рекомендація для гаманців**

Часто запитуйте нові blockhash та оновлюйте blockhash транзакції перед
підписанням, щоб забезпечити її актуальність.

### Використовуйте надійні RPC-вузли для отримання blockhash

RPC-вузли можуть відставати від кластеру через навантаження або інші причини,
повертаючи blockhash, який майже вичерпав строк дії.

**Рекомендація**

Моніторте стан RPC-вузлів, використовуючи такі методи:

1. Викликайте API [`getSlot`](/docs/uk/rpc/http/getSlot.mdx) з рівнем
   підтвердження `processed` для отримання останнього обробленого слота вузла та
   порівняйте його з
   [`getMaxShredInsertSlot`](/docs/uk/rpc/http/getMaxShredInsertSlot.mdx), щоб
   оцінити відставання вузла.
2. Використовуйте RPC API `getLatestBlockhash` з рівнем підтвердження
   `confirmed` на кількох різних RPC-вузлах та обирайте blockhash від вузла,
   який повертає найвищий слот для свого
   [контекстного слоту](/docs/uk/rpc/index.mdx#rpcresponse-structure).

### Зачекайте достатньо довго перед закінченням терміну дії

**Рекомендація**

При виклику RPC API
[`getLatestBlockhash`](/docs/uk/rpc/http/getLatestBlockhash.mdx) для отримання
останнього blockhash вашої транзакції зверніть увагу на `lastValidBlockHeight` у
відповіді.

Потім викликайте RPC API
[`getBlockHeight`](/docs/uk/rpc/http/getBlockHeight.mdx) з рівнем підтвердження
`confirmed`, поки він не поверне висоту блоку, більшу за попередньо отриманий
`lastValidBlockHeight`.

### Розгляньте використання "довготривалих" транзакцій

Іноді уникнути проблем із закінченням терміну дії транзакції дуже важко
(наприклад, при офлайн-підписанні або нестабільності кластера). Якщо попередні
поради не підходять для вашого випадку використання, ви можете перейти на
використання довготривалих транзакцій (вони вимагають трохи налаштувань).

Щоб почати використовувати довготривалі транзакції, користувач спочатку повинен
подати транзакцію, яка
[викликає інструкції для створення спеціального "nonce" облікового запису на блокчейні](https://docs.rs/solana-program/latest/solana_program/system_instruction/fn.create_nonce_account.html)
і зберігає в ньому "довготривалий blockhash". У будь-який момент у майбутньому
(доки nonce-аккаунт ще не використаний) користувач може створити довготривалу
транзакцію, дотримуючись двох правил:

1. Список інструкцій повинен починатися з
   [системної інструкції "advance nonce"](https://docs.rs/solana-program/latest/solana_program/system_instruction/fn.advance_nonce_account.html),
   яка завантажує їх nonce-аккаунт.
2. Blockhash транзакції повинен дорівнювати довготривалому blockhash,
   збереженому nonce-аккаунтом на блокчейні.

Ось як Solana обробляє ці довготривалі транзакції:

1. Якщо blockhash транзакції більше не є "недавнім", система перевіряє, чи
   починається список інструкцій транзакції з інструкції "advance nonce".
2. Якщо так, система завантажує nonce-аккаунт, вказаний у цій інструкції.
3. Потім перевіряє, чи збережений довготривалий blockhash відповідає blockhash
   транзакції.
4. Нарешті, система оновлює blockhash nonce-аккаунту до останнього недавнього
   blockhash, щоб гарантувати, що ця ж транзакція не може бути оброблена
   повторно.

Детальніше про роботу цих довготривалих транзакцій ви можете дізнатися з
[оригінальної пропозиції](https://docs.anza.xyz/implemented-proposals/durable-tx-nonces)
та
[ознайомитися з прикладом](/content/guides/advanced/introduction-to-durable-nonces.md)
у документації Solana.
