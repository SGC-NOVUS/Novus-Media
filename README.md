<div align="center">
  <img src="assets/logo-256.png" width="116" height="116" alt="Novus Media Logo" />
  <h1>Novus Media</h1>
  <p><b>Кроссплатформенный аудио- и медиаплеер нового поколения для YouTube Music, локальных фонотек и облачного медиа</b></p>

  <p>
    <a href="https://github.com/SGC-NOVUS/Novus-Media/releases"><img src="https://img.shields.io/github/v/release/SGC-NOVUS/Novus-Media?label=Download%20Release&color=0ea5e9&style=for-the-badge" alt="Download Release" /></a>
  </p>
  <p>
    <img src="https://img.shields.io/badge/Linux-DEB%20(amd64)-38bdf8?style=flat-square&logo=linux&logoColor=white" alt="Linux" />
    <img src="https://img.shields.io/badge/Windows-10%20%2F%2011%20(x64)-38bdf8?style=flat-square&logo=windows&logoColor=white" alt="Windows" />
    <img src="https://img.shields.io/badge/Android-APK%20(arm64%20%2F%20x86__64)-38bdf8?style=flat-square&logo=android&logoColor=white" alt="Android" />
    <img src="https://img.shields.io/badge/macOS%20%2F%20iOS-Supported%20UI-38bdf8?style=flat-square&logo=apple&logoColor=white" alt="Apple" />
  </p>
</div>

---

## О приложении

**Novus Media** — быстрый, лёгкий и безопасный медиаплеер, разработанный в экосистеме **SGC-NOVUS (NOVUS-OS)**. Плеер объединяет прямое потоковое воспроизведение **YouTube Music** (с поддержкой аккаунтов **YouTube Premium** без рекламы), локальную музыкальную коллекцию, синхронное караоке с таймкодами, ИИ-очистку названий треков и защищённый офлайн-кеш с максимальным сжатием **Opus Ultra VBR**.

> В данном публичном репозитории публикуются **официальная информация о проекте, история обновлений ([CHANGELOG.md](CHANGELOG.md)) и готовые установочные сборки в разделе [Releases](https://github.com/SGC-NOVUS/Novus-Media/releases)**.

---

## Скачать Novus Media

Перейдите в раздел **[Releases](https://github.com/SGC-NOVUS/Novus-Media/releases/latest)** и выберите установочный пакет для вашей операционной системы:

| Платформа | Формат пакета | Установка |
| :--- | :--- | :--- |
| **Linux** (Ubuntu / Debian / NOVUS-OS / Mint) | `novus-media_<version>_amd64.deb` | `sudo apt install ./novus-media_<version>_amd64.deb` |
| **Windows** (Windows 10 / 11 x64) | `novus-media_<version>_win-x64.zip` | Распакуйте архив и запустите `NovusMedia.exe` |
| **Android** (Смартфоны / Планшеты / Android TV) | `novus-media-<version>.apk` | Установите `.apk` файл на устройство |

---

## Ключевые возможности

### 1. Безопасная прямая трансляция YouTube Music & YouTube Premium
- **Прямой онлайн-стриминг с защитой аккаунта**: интеллектуальное управление потоками и автоматическое бесшовное переподключение при обрывах сети без остановки воспроизведения.
- **Поддержка YouTube Premium**: автоматическое определение активной подписки YouTube Premium / Music Premium для потокового воспроизведения без рекламы в максимальном качестве (`Opus` / `AAC`).
- **Умный приоритет источников**: если трек уже скачан на устройство или сохранён в офлайн-кеш, в любом плейлисте он отображается соответствующей иконкой (**Скачано** / **В кеше**) и воспроизводится мгновенно с диска без лишних сетевых запросов.

### 2. Караоке и тексты песен с таймкодами
- Автоматический поиск синхронизированных текстов песен с таймкодами `[mm:ss.xx]` (**LRCLIB Karaoke**, официальные тексты YouTube Music и локальные `.lrc` файлы).
- Быстрая перемотка композиции кликом по любой строке текста в режиме караоке.

### 3. Novus Pro — Поддержка проекта
- **Зашифрованный офлайн-кеш (Opus Ultra VBR 64k, AES-256-GCM)**:
  - Максимальное сжатие аудио без потери воспринимаемого качества — треки занимают **в 2 раза меньше места** (~1.2–1.5 МБ на композицию).
  - В каждый закешированный файл автоматически сохраняются обложка, ссылка на YouTube Music и синхронизированный текст песни с таймкодами для офлайн-караоке.
  - Защита ресурса системного SSD/NVMe с возможностью размещения кеша на внешнем HDD или отдельном диске.
- **Экспорт и синхронизация плейлистов на ПК (`~/Музыка`)**:
  - Сохранение треков и плейлистов в `.opus`, `.m4a` или `.mp3` с вшитой обложкой, чистыми тегами и `.lrc` караоке.
- **Публикация в Telegram-канал через бота**:
  - Автоматическая выгрузка треков и плейлистов в публичные и приватные Telegram-каналы от имени канала с обложкой и контролем дубликатов.
  - Все выгружаемые в Telegram треки автоматически сохраняются локально на ПК в том же формате.

### 4. AI Очистка метаданных (Google Gemini API + локальный фильтр)
- Автоматическая очистка названий треков от хештегов, ссылок, эмодзи, суффиксов каналов (`_NEWS`, `- Topic`, `Official Video`) и дублей имени исполнителя.

### 5. Светлый (*Platinum Snow*) и Тёмный (*Obsidian Metallic*) режимы
- Единая шапка окна с быстрым переключением в **Компактный блочный режим (`420×650`)** и **Плавающий мини-плеер**.
- Полная локализация интерфейса: **Русский**, **Украинский**, **Английский**.

---

## Авторство и поддержка

Разработано командой **SGC-NOVUS** (`Kyryll Selantiev`).
- История релизов: [CHANGELOG.md](CHANGELOG.md)
- Скачать последнюю версию: [GitHub Releases](https://github.com/SGC-NOVUS/Novus-Media/releases)
