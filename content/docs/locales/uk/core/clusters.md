---
sidebarLabel: Кластери та точки доступу RPC
title: Кластери та публічні точки доступу RPC
sidebarSortOrder: 8
description:
  Дізнайтеся про кластери мережі Solana (Devnet, Testnet і Mainnet Beta), їхні
  публічні точки доступу RPC, обмеження швидкості та випадки використання.
  Дізнайтеся, як підключатися до різних мереж Solana для розробки, тестування та
  виробничого середовища.
---

Блокчейн Solana має кілька різних груп валідаторів, відомих як
[Кластери](/docs/uk/core/clusters.md). Кожен з них обслуговує різні цілі в межах
загальної екосистеми та містить виділені вузли API для виконання
[JSON-RPC](/docs/uk/rpc/index.mdx) запитів для своїх відповідних кластерів.

Індивідуальні вузли в межах кластера належать та управляються третіми сторонами,
з публічною точкою доступу, доступною для кожного.

## Публічні точки доступу RPC Solana

Організація Solana Labs керує публічною точкою доступу RPC для кожного кластера.
Кожна з цих публічних точок доступу має обмеження швидкості, але доступна для
користувачів та розробників для взаємодії з блокчейном Solana.

> Обмеження швидкості публічних точок доступу можуть змінюватися. Зазначені
> обмеження швидкості в цьому документі можуть бути не найактуальнішими.

### Використання експлорерів з різними кластерами

Багато популярних експлорерів блокчейну Solana підтримують вибір будь-якого
кластера, часто дозволяючи досвідченим користувачам додавати користувацьку/
приватну точку доступу RPC.

Прикладами таких експлорерів є:

- [http://explorer.solana.com/](https://explorer.solana.com/).
- [http://solana.fm/](https://solana.fm/).
- [http://solscan.io/](https://solscan.io/).
- [http://solanabeach.io/](http://solanabeach.io/).
- [http://validators.app/](http://validators.app/).

## Основні відомості

- Mainnet: Живе виробниче середовище для розгорнутих додатків.
- Devnet: Тестування з публічним доступом для розробників, які експериментують
  зі своїми додатками.
- Testnet: Стрес-тестування для оновлень мережі та продуктивності валідаторів.

**Приклади використання**: Можливо, ви захочете налагодити нову програму на
Devnet або перевірити метрики продуктивності на Testnet перед розгортанням на
Mainnet.

| **Кластер** | **Точка доступу**                     | **Призначення**                 | **Примітки**                    |
| ----------- | ------------------------------------- | ------------------------------- | ------------------------------- |
| Mainnet     | `https://api.mainnet-beta.solana.com` | Живе виробниче середовище       | Потребує SOL для транзакцій     |
| Devnet      | `https://api.devnet.solana.com`       | Публічне тестування та розробка | Безкоштовний SOL для тестування |
| Testnet     | `https://api.testnet.solana.com`      | Тестування валідаторів          | Може мати періодичні простої    |

## Devnet

Devnet слугує тестовим майданчиком для будь-кого, хто хоче ознайомитися з Solana
як користувач, власник токенів, розробник додатків або валідатор.

- Розробники додатків повинні орієнтуватися на Devnet.
- Потенційні валідатори повинні спочатку орієнтуватися на Devnet.
- Основні відмінності між Devnet і Mainnet Beta:
  - Токени Devnet **не реальні**.
  - Devnet включає крани для отримання токенів для тестування додатків.
  - Devnet може піддаватися скиданням журналу.
  - Devnet зазвичай працює на тій самій гілці випуску програмного забезпечення,
    що й Mainnet Beta, але може працювати на новішій мінорній версії.
- Точка доступу Gossip для Devnet: `entrypoint.devnet.solana.com:8001`

### Точка доступу Devnet

- `https://api.devnet.solana.com` - єдиний вузол API, розміщений Solana Labs; з
  обмеженнями швидкості

#### Приклад конфігурації командного рядка `solana`

Щоб підключитися до кластеру `devnet` за допомогою CLI Solana:

```shell
solana config set --url https://api.devnet.solana.com
```

### Обмеження швидкості Devnet

- Максимальна кількість запитів на 10 секунд на IP: 100
- Максимальна кількість запитів на 10 секунд на IP для одного RPC: 40
- Максимальна кількість одночасних з'єднань на IP: 40
- Максимальна швидкість з'єднань на 10 секунд на IP: 40
- Максимальна кількість даних на 30 секунд: 100 МБ

## Testnet

Testnet - це місце, де основні учасники Solana стрес-тестують функції останніх
випусків у живому кластері, особливо зосереджуючись на продуктивності мережі,
стабільності та поведінці валідаторів.

- Токени Testnet **не реальні**.
- Testnet може піддаватися скиданням журналу.
- Testnet включає крани для отримання токенів для тестування додатків.
- Testnet зазвичай працює на новішій гілці випуску програмного забезпечення, ніж
  Devnet і Mainnet Beta.
- Точка доступу Gossip для Testnet: `entrypoint.testnet.solana.com:8001`

### Точка доступу Testnet

- `https://api.testnet.solana.com` - єдиний вузол API, розміщений Solana Labs; з
  обмеженнями швидкості

#### Приклад конфігурації командного рядка `solana`

Щоб підключитися до кластеру `testnet` за допомогою CLI Solana:

```shell
solana config set --url https://api.testnet.solana.com
```

### Обмеження швидкості Testnet

- Максимальна кількість запитів на 10 секунд на IP: 100
- Максимальна кількість запитів на 10 секунд на IP для одного RPC: 40
- Максимальна кількість одночасних з'єднань на IP: 40
- Максимальна швидкість з'єднань на 10 секунд на IP: 40
- Максимальна кількість даних на 30 секунд: 100 МБ

## Mainnet beta

Безперешкодний, постійний кластер для користувачів Solana, розробників,
валідаторів та власників токенів.

- Токени, випущені на Mainnet Beta, є **реальними** SOL.
- Точка доступу Gossip для Mainnet Beta:
  `entrypoint.mainnet-beta.solana.com:8001`

### Точка доступу Mainnet beta

- `https://api.mainnet-beta.solana.com` - кластер вузлів API, розміщених Solana
  Labs, із підтримкою балансувальника навантаження; з обмеженнями швидкості

#### Приклад конфігурації командного рядка `solana`

Щоб підключитися до кластеру `mainnet-beta` за допомогою CLI Solana:

```shell
solana config set --url https://api.mainnet-beta.solana.com
```

### Обмеження швидкості Mainnet beta

- Максимальна кількість запитів на 10 секунд на IP: 100
- Максимальна кількість запитів на 10 секунд на IP для одного RPC: 40
- Максимальна кількість одночасних з'єднань на IP: 40
- Максимальна швидкість з'єднань на 10 секунд на IP: 40
- Максимальна кількість даних на 30 секунд: 100 МБ

> Публічні точки доступу RPC не призначені для виробничих додатків. Будь ласка,
> використовуйте виділені/приватні сервери RPC під час запуску додатка, випуску
> NFT тощо. Публічні сервіси піддаються зловживанням, і обмеження швидкості
> можуть змінюватися без попередження. Також, вебсайти з високим трафіком можуть
> бути заблоковані без попередження.

## Загальні HTTP-коди помилок

- 403 -- Ваша IP-адреса або вебсайт було заблоковано. Настав час запустити
  власні сервери RPC або знайти приватний сервіс.
- 429 -- Ваша IP-адреса перевищує обмеження швидкості. Зменшіть швидкість!
  Використовуйте
  [Retry-After](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Retry-After)
  HTTP-заголовок відповіді, щоб визначити, як довго потрібно чекати перед
  повторною спробою.
