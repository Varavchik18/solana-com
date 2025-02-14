---
sidebarSortOrder: 4
title: Стиснення стану (State Compression)
description:
  'Стиснення стану - це метод дешевого та безпечного збереження "відбитків"
  даних поза мережею в реєстрі Solana замість дорогих облікових записів.'
---

У Solana [стиснення стану](/docs/uk/advanced/state-compression.md) є методом
створення "відбитку" (або гешу) даних поза мережею та збереження цього відбитку
в мережі для безпечної перевірки. Цей процес використовує безпеку реєстру Solana
для гарантування цілісності даних поза мережею, забезпечуючи їх незмінність.

Цей метод "стиснення" дозволяє програмам та децентралізованим додаткам (dApps)
використовувати дешевий простір у [реєстрі](/docs/uk/terminology.md#ledger)
блокчейну, замість більш дорогого простору
[облікових записів](/docs/uk/terminology.md#account), для безпечного зберігання
даних.

Це досягається за допомогою спеціальної структури двійкового дерева, відомого як
[конкурентне мерклеве дерево](#what-is-a-concurrent-merkle-tree), яке створює
геш кожного фрагмента даних (названого `листком`), об'єднує ці геші і зберігає
тільки фінальний геш у мережі.

## Що таке стиснення стану?

Простіше кажучи, стиснення стану використовує структури "**_дерев_**" для
криптографічного хешування даних поза мережею детермінованим способом, щоб
обчислити один кінцевий геш, який зберігається у мережі.

Ці _дерева_ створюються таким "_детермінованим_" процесом:

- Береться будь-який фрагмент даних.
- Створюється геш цих даних.
- Цей геш зберігається як `листок` на нижньому рівні дерева.
- Кожна пара `листків` об'єднується в геш, створюючи `гілку`.
- Кожна `гілка` також хешується разом.
- Процес повторюється, поки не буде обчислено фінальний `кореневий геш`.

Цей `кореневий геш` зберігається в мережі як верифіковане **_підтвердження_**
для всіх даних у кожному листку. Це дозволяє криптографічно перевіряти всі дані
поза мережею, використовуючи мінімальну кількість даних у мережі. Таким чином,
значно знижуються витрати на зберігання/перевірку великих обсягів даних завдяки
"стисненню стану".

## Мерклеві дерева та конкурентні мерклеві дерева

Стиснення стану у Solana використовує спеціальний тип
[мерклевого дерева](#what-is-a-merkle-tree), який дозволяє виконувати кілька
змін у дереві, зберігаючи його цілісність і валідність.

Це спеціальне дерево, відоме як
"[конкурентне мерклеве дерево](#what-is-a-concurrent-merkle-tree)", зберігає
"журнал змін" дерева в мережі. Це дозволяє виконувати кілька змін до дерева
(наприклад, у межах одного блоку), не порушуючи підтвердження.

### Що таке мерклеве дерево?

[Мерклеве дерево](https://uk.wikipedia.org/wiki/Мерклеве_дерево), або "дерево
гешів", — це двійкова структура, у якій кожен `листок` є криптографічним гешем
даних. Всі вузли, які **не** є листками, називаються `гілками` і є гешами їхніх
дочірніх листків.

Кожна гілка також хешується разом, поступово піднімаючись вгору, поки не
залишиться один геш. Цей фінальний геш, званий `кореневим гешем`, можна
використовувати разом із "шляхом підтвердження" для перевірки будь-яких даних,
збережених у листковому вузлі.

### Що таке Конкурентне Мерклеве дерево?

У високопродуктивних застосунках, таких як
[середовище виконання Solana](/docs/uk/core/fees.md), запити на зміну ончейн
_традиційного мерклевого дерева_ можуть надходити до валідаторів досить швидко
(наприклад, у межах одного слота). У таких випадках кожна зміна даних у листках
повинна виконуватися послідовно. Це призводить до невдачі наступних запитів на
зміну, оскільки кореневий геш і підтвердження стають недійсними після
попередньої зміни в слоті.

Рішенням цієї проблеми є Конкурентні Мерклеві дерева.

**Конкурентне Мерклеве дерево** зберігає **захищений журнал змін**, який містить
останні зміни, їх кореневий геш і підтвердження для його обчислення. Цей буфер
змін ("changelog buffer") зберігається ончейн у спеціальному акаунті для кожного
дерева, з обмеженням на максимальну кількість записів у журналі змін
(`maxBufferSize`).

Коли валідатори отримують кілька запитів на зміну даних у листках у межах одного
слота, ончейн _конкурентне мерклеве дерево_ може використовувати цей буфер змін
як джерело правдивої інформації для більш прийнятних підтверджень. Це дозволяє
виконувати до `maxBufferSize` змін для одного дерева в межах одного слота, що
значно підвищує пропускну здатність.

## Розмір Конкурентного Мерклевого дерева

При створенні такого ончейн дерева існує 3 параметри, які визначають розмір
дерева, вартість його створення і кількість одночасних змін:

1. Максимальна глибина (max depth)
2. Розмір буфера змін (max buffer size)
3. Глибина крони дерева (canopy depth)

### Максимальна глибина

"Максимальна глибина" дерева — це **максимальна кількість** переходів від
будь-якого `листка` даних до `кореня` дерева.

Оскільки мерклеві дерева є двійковими, кожен листок з'єднаний лише з **одним**
іншим листком; вони утворюють `пару листків`.

Таким чином, `maxDepth` дерева використовується для визначення максимальної
кількості вузлів (тобто елементів даних або `листків`), які можна зберігати у
дереві, за допомогою простої формули:

```text
nodes_count = 2 ^ maxDepth
```

Оскільки глибину дерева потрібно встановити під час його створення, ви повинні
визначити, скільки елементів даних ви хочете зберігати у своєму дереві. Потім,
використовуючи просту формулу вище, ви можете визначити найменше значення
`maxDepth`, яке дозволить зберігати ваші дані.

#### Приклад 1: Мінтинг 100 NFT

Якщо ви хочете створити дерево для зберігання 100 стиснутих NFT, вам знадобиться
щонайменше "100 листків" або "100 вузлів".

```text
// maxDepth=6 -> 64 nodes
2^6 = 64

// maxDepth=7 -> 128 nodes
2^7 = 128
```

Ми повинні використовувати значення `maxDepth` рівне `7`, щоб забезпечити
можливість зберігати всі наші дані.

#### Приклад 2: Мінтинг 15000 NFT

Якщо ви хочете створити дерево для зберігання 15000 стиснутих NFT, вам
знадобиться щонайменше "15000 листків" або "15000 вузлів".

```text
// maxDepth=13 -> 8192 nodes
2^13 = 8192

// maxDepth=14 -> 16384 nodes
2^14 = 16384
```

Ми повинні використовувати `maxDepth` рівне `14`, щоб забезпечити можливість
зберігати всі наші дані.

#### Чим більша максимальна глибина, тим вища вартість

Значення `maxDepth` є одним із основних чинників вартості під час створення
дерева, оскільки ви оплачуєте цю вартість наперед при створенні дерева. Чим
більша глибина дерева, тим більше даних (відбитків або хешів) можна зберігати,
але тим вища вартість.

---

### Максимальний розмір буфера (maxBufferSize)

"Максимальний розмір буфера" — це максимальна кількість змін, які можуть бути
внесені до дерева, доки кореневий хеш (`root hash`) залишається дійсним.

Оскільки кореневий хеш є єдиним хешем для всіх даних листків, зміна будь-якого
окремого листка інвалідовує proof, потрібний для всіх наступних спроб змінити
будь-який інший листок у звичайному дереві.

У випадку [Concurrent Tree](#що-таке-concurrent-merkle-tree), існує журнал змін
для цих proof, який задається під час створення дерева через параметр
`maxBufferSize`.

---

### Глибина козирка (canopyDepth)

"Глибина козирка" або "розмір козирка" визначає кількість рівнів proof, які
кешуються або зберігаються on-chain для даного proof-шляху.

Коли виконується дія оновлення для `leaf` (наприклад, передача права власності),
**повний** proof-шлях повинен бути використаний для верифікації початкового
права власності на листок. Це здійснюється шляхом обчислення поточного
`root hash`.

Для великих дерев потрібно більше proof-вузлів, щоб виконати цю перевірку.
Наприклад, якщо `maxDepth` дорівнює `14`, потрібно `14` proof-вузлів. З
використанням козирка частина цих вузлів зберігається on-chain, зменшуючи
кількість proof-вузлів, які потрібно включити в транзакції.

Наприклад, дерево з `maxDepth` рівне `14` із козирком розміром `10`
потребуватиме лише `4` proof-вузли на транзакцію.

![Глибина козирка 1 для Concurrent Merkle Tree з максимальною глибиною 3](/assets/docs/compression/canopy-depth-1.png)

---

#### Чим більша глибина козирка, тим вища вартість

Значення `canopyDepth` також є одним із основних чинників вартості створення
дерева. Чим більше proof-вузлів зберігається on-chain, тим вища вартість.

#### Низький козирок обмежує композитність

Низьке значення `canopyDepth` вимагає більше proof-вузлів у кожній транзакції
оновлення. Це обмежує можливості для інтеграції вашого дерева з іншими
Solana-програмами або dApps.

Наприклад, дерево, яке використовується для стиснутих NFT з низьким
`canopyDepth`, може дозволяти лише базові дії, як-от передача, але не
підтримувати розширені функції, такі як система ставок.

---

## Вартість створення дерева

Вартість створення Concurrent Merkle Tree залежить від його параметрів:
`maxDepth`, `maxBufferSize`, і `canopyDepth`. Ці параметри визначають необхідний
простір (у байтах) для дерева.

Використовуючи метод
[`getMinimumBalanceForRentExemption`](/docs/uk/rpc/http/getminimumbalanceforrentexemption),
можна дізнатися вартість (у лампортах) для виділення цього простору on-chain.

---

### Розрахунок вартості дерева у JavaScript

У пакеті
[`@solana/spl-account-compression`](https://www.npmjs.com/package/@solana/spl-account-compression)
можна використовувати функцію
[`getConcurrentMerkleTreeAccountSize`](https://solana-labs.github.io/solana-program-library/account-compression/sdk/docs/modules/index.html#getConcurrentMerkleTreeAccountSize)
для розрахунку необхідного простору для дерева з заданими параметрами.

Далі, за допомогою функції
[`getMinimumBalanceForRentExemption`](https://solana-labs.github.io/solana-web3.js/v1.x/classes/Connection.html#getMinimumBalanceForRentExemption),
можна визначити остаточну вартість (у лампортах) для створення дерева, подібно
до будь-якого іншого акаунта.

```ts
// calculate the space required for the tree
const requiredSpace = getConcurrentMerkleTreeAccountSize(
  maxDepth,
  maxBufferSize,
  canopyDepth,
);

// get the cost (in lamports) to store the tree on-chain
const storageCost =
  await connection.getMinimumBalanceForRentExemption(requiredSpace);
```

### Приклади вартості

Нижче наведено кілька прикладів вартості для дерев різного розміру, включаючи
кількість можливих листків:

**Приклад №1: 16,384 вузлів, вартість 0.222 SOL**

- максимальна глибина: `14`, максимальний розмір буфера: `64`
- максимальна кількість листків: `16,384`
- глибина козирка: `0`, вартість створення приблизно `0.222 SOL`

**Приклад №2: 16,384 вузлів, вартість 1.134 SOL**

- максимальна глибина: `14`, максимальний розмір буфера: `64`
- максимальна кількість листків: `16,384`
- глибина козирка: `11`, вартість створення приблизно `1.134 SOL`

**Приклад №3: 1,048,576 вузлів, вартість 1.673 SOL**

- максимальна глибина: `20`, максимальний розмір буфера: `256`
- максимальна кількість листків: `1,048,576`
- глибина козирка: `10`, вартість створення приблизно `1.673 SOL`

**Приклад №4: 1,048,576 вузлів, вартість 15.814 SOL**

- максимальна глибина: `20`, максимальний розмір буфера: `256`
- максимальна кількість листків: `1,048,576`
- глибина козирка: `15`, вартість створення приблизно `15.814 SOL`

---

## Стиснуті NFT

Стиснуті NFT є одним із найпопулярніших варіантів використання стиснення стану
на Solana. Завдяки стисненню колекцію з одного мільйона NFT можна створити за
`~50 SOL`, у порівнянні з `~12,000 SOL` для її нестиснутої еквівалентної
колекції.

Якщо ви зацікавлені в створенні стиснутих NFT, ознайомтеся з нашим посібником
для розробників:
[створення та передача стиснутих NFT](/content/guides/javascript/compressed-nfts.md).
