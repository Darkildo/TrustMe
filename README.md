<p align="center">
  <img src="assets/logo.png" width="112" alt="TrustMe">
</p>

<h1 align="center">TrustMe</h1>

<p align="center">
  <b>Приватный мессенджер с открытым сервером и сетью доверия.</b><br>
  Без номера телефона, до 8 устройств на аккаунт, свои ноды.
</p>

<p align="center">
  <a href="https://github.com/Darkildo/TrustMe-releases/releases/latest"><img alt="Приложение" src="https://img.shields.io/github/v/release/Darkildo/TrustMe-releases?label=%D0%BF%D1%80%D0%B8%D0%BB%D0%BE%D0%B6%D0%B5%D0%BD%D0%B8%D0%B5&sort=semver"></a>
  <a href="https://github.com/Darkildo/trustme_backend/tags"><img alt="Нода" src="https://img.shields.io/github/v/tag/Darkildo/trustme_backend?label=%D0%BD%D0%BE%D0%B4%D0%B0&sort=semver"></a>
  <a href="https://github.com/Darkildo/trustme_admin/tags"><img alt="Админка" src="https://img.shields.io/github/v/tag/Darkildo/trustme_admin?label=%D0%B0%D0%B4%D0%BC%D0%B8%D0%BD%D0%BA%D0%B0&sort=semver"></a>
  <a href="README.en.md"><img alt="English" src="https://img.shields.io/badge/lang-English-informational"></a>
</p>

---

Это обзорный репозиторий: здесь собраны все части TrustMe со ссылками. Кода в нём нет.

TrustMe состоит из приложения, серверной ноды и панели управления нодами. Сообщения, файлы, звонки и местоположение
шифруются на устройстве и расшифровываются только у собеседника; нода пересылает зашифрованные конверты и не может их прочитать.
Общим сервером пользоваться не обязательно: ноду можно поднять у себя одной командой.

## Карта проекта

| Репозиторий | Что внутри | Стек | Состояние |
| --- | --- | --- | --- |
| [**TrustMe-releases**](https://github.com/Darkildo/TrustMe-releases) | Приложение: сборки для Android, Windows и Linux (iOS через TestFlight), описание возможностей, установка, проверка подлинности файлов, история версий | Flutter | Релизы выходят регулярно, [Releases](https://github.com/Darkildo/TrustMe-releases/releases) |
| [**trustme_backend**](https://github.com/Darkildo/trustme_backend) | Нода: шифрованный канал Noise поверх TCP, доставка конвертов между пользователями и их устройствами, хранение недоставленного и файлов, пробуждение офлайн-устройств пушами без содержимого. Протокол, схемы protobuf, инструкции по развёртыванию | Rust | Код открыт, [LICENSING.md](https://github.com/Darkildo/trustme_backend/blob/master/LICENSING.md) |
| [**trustme_admin**](https://github.com/Darkildo/trustme_admin) | Панель управления нодами: пробы снаружи, отчёты `tmnode doctor` с хоста, установка новых нод по SSH, лимиты без перезапуска, STUN/TURN для звонков, пользователи с ролями и TOTP | Rust + React | Код открыт, [LICENSING.md](https://github.com/Darkildo/trustme_admin/blob/master/LICENSING.md) |
| Клиент и криптографическое ядро | Исходный код приложения и библиотеки шифрования, на которой оно построено | Flutter, Rust + Dart | Готовятся к публикации |

## Как части связаны

```mermaid
flowchart LR
    subgraph devices["Устройства одного аккаунта, до 8"]
        direction TB
        A["Android"]
        I["iOS"]
        W["Windows"]
        L["Linux"]
    end
    subgraph nodes["Пул нод пользователя, до 3"]
        direction TB
        N1[("Нода 1\ntrustme_backend")]
        N2[("Нода 2")]
    end
    devices -- "Noise поверх TCP,\nзашифрованные конверты" --> nodes
    nodes -. "пуш без содержимого" .-> P["FCM / APNs"]
    P -.-> devices
    nodes --- T["STUN / TURN\nдля звонков"]
    ADM["trustme_admin"] -- "пробы, tmnode,\nлимиты, TURN" --> nodes
```

- Приложение держит сессии на всех нодах своего пула; главная нода принимает сообщения, остальные подстраховывают.
- Свои устройства синхронизируются между собой через те же ноды, в зашифрованном виде.
- Файлы загружаются на ноду одним зашифрованным контейнером, собеседнику уходит только указатель.
- Панель управления общается с нодами через агент `tmnode`, которому не нужен ни один входящий порт на сервере ноды.

## С чего начать

**Хочу пользоваться.** Скачайте приложение на странице [TrustMe-releases](https://github.com/Darkildo/TrustMe-releases#readme):
там же описаны возможности, установка на каждую платформу и проверка контрольных сумм.

**Хочу свою ноду.** В [trustme_backend](https://github.com/Darkildo/trustme_backend#readme) есть запуск через Docker Compose,
[описание развёртывания](https://github.com/Darkildo/trustme_backend/blob/master/docs/deployment.md) и
[runbook'и](https://github.com/Darkildo/trustme_backend/tree/master/docs). В приложении достаточно указать адрес ноды и её ключ.
Для парка нод поставьте [trustme_admin](https://github.com/Darkildo/trustme_admin#readme): он сам проверяет ноды и ставит новые по SSH.

**Хочу разобраться в протоколе.** Формат обмена описан в
[server-contract.md](https://github.com/Darkildo/trustme_backend/blob/master/server-contract.md),
схемы в [schemas/](https://github.com/Darkildo/trustme_backend/tree/master/schemas),
архитектура ноды в [docs/architecture.md](https://github.com/Darkildo/trustme_backend/blob/master/docs/architecture.md).

## Безопасность в двух словах

- Аккаунт: пара ключей, созданная на устройстве из движения пальцем. Номер телефона и почта не нужны.
- Сообщения, медиа, геолокация и профиль: X25519, AES-256-GCM, Ed25519. Звонки: DTLS-SRTP. Канал до ноды: Noise.
- По желанию в личных чатах: защита прошлых сообщений на основе Double Ratchet.
- Сеть доверия: сверка отпечатков при встрече, рекомендации контактов, права для каждого контакта отдельно.
- Нода видит метаданные (кто, кому, когда, какого размера), но не содержимое. Подробнее в разделе
  [«Безопасность и приватность»](https://github.com/Darkildo/TrustMe-releases#безопасность-и-приватность).

Об уязвимостях пишите по адресу из SECURITY.md соответствующего репозитория, а не в публичные issue.

## Лицензии

Приложение бесплатно и распространяется готовыми сборками. Код ноды и панели управления открыт; условия использования,
включая коммерческое, описаны в файлах LICENSING.md каждого репозитория.

## Связь

Вопросы и идеи: [Discussions](https://github.com/Darkildo/TrustMe-releases/discussions) в репозитории приложения.
Ошибки: issues в репозитории той части, где они возникли. Остальное: didakostya@gmail.com.
