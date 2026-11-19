# Changelog

Все заметные изменения проекта **Novus Media** документируются в этом файле.
Формат версий соответствует **SemVer (`X.Y.Z`)**:
- **X (Major)** — мажорная версия / ломающие изменения архитектуры (`0.y.z` — активная предрелизная ветка до кроссплатформенного `1.0.0`).
- **Y (Minor)** — новые пользовательские функции, разделы и подсистемы с сохранением совместимости.
- **Z (Patch)** — исправления ошибок, стабилизация и точечные оптимизации.

---

## [0.9.0] - 2026-11-19

### Added
- **Localhost HTTP Range-Chunk Audio Stream Proxy & YouTube Premium Detection**: Added a built-in localhost chunked byte-range stream proxy (`127.0.0.1:<port>/stream/<req_id>`) in `PlayerEngine` and `get_direct_stream_info()` in `YTMusicDirectClient` that streams `googlevideo.com` in 512 KB bounded chunks with `yt-dlp` headers and transparent mid-stream 403 refresh (eliminating GStreamer `souphttpsrc` `HTTP 403 Forbidden` errors), plus automatic detection of `YouTube Premium` subscription status (`is_premium`).
- **Maximum Cache Compression (Opus Ultra VBR 64k) + Full Karaoke Metadata**: Upgraded `CacheManager` with `_compress_and_tag_opus()` (`libopus -b:a 64k -vbr on -compression_level 10 -application audio`) and `optimize_existing_cache()`, reducing cached file sizes by ~50–55% (~1.2–1.5 MB per track) while embedding `videoId`, `youtube_url`, cover art, and synced karaoke lyrics (`[mm:ss.xx]`).
- **Automatic Local PC Save on Telegram Channel Export & Novus Pro Tier**: Tracks exported/published to a Telegram channel via `TelegramChannelPublisher` are now automatically saved locally to `~/Музыка` in the selected format (`.opus`, `.m4a`, `.mp3`) with cover art, tags, and `.lrc` karaoke lyrics, and immediately registered as **Downloaded** across all playlists; added **Novus Pro (Поддержка проекта)** tier badging and controls for Offline Caching, Local Downloads, and Telegram Bot Publishing.
- **Unified Window HeaderBar Controls**: Moved Compact Block Mode (`◫`) and Mini-Player (`⛶`) toggles directly into the custom window `HeaderBar` (native GTK `HeaderBar` on Linux and `.top-header` `.non-native-header-only` on Windows/macOS/Web) so mode switching is always accessible in both Standard and Compact Block Mode.
- **Complete 3-Language Localization (`ru.json`, `uk.json`, `en.json`)**: Expanded all three dictionaries to 239 keys with 100% key parity, localizing Telegram Channel Publisher settings, AI Metadata Cleanup settings, Context Menu items, Collection chips, and tooltips via `data-i18n*` attributes.
- **Anti-Ban Pacing & Thread-Safe YouTube Music Engine**: Added `_pace_api_call()` with randomized human-like jitter (`0.32s + 0.0..0.28s`), thread-safe `self._yt_lock` around `YTMusic` calls, shared lock-protected `YoutubeDL` extraction pool (`self._ydl_clients`), and cookieless-first `yt-dlp` downloads (`android_vr,web_music` before falling back to authenticated cookies).

### Changed
- **Cross-Playlist Cached & Downloaded Track Matching & Self-Healing Playback**: Added `CacheManager.resolve_cached_track(video_id, title, artist)` and `cachedNormKeySet`/`localNormKeySet` in `ui/app.js` so cached and downloaded tracks are recognized across all playlists even when YouTube Music uses alternate video IDs; reordered `_initial_data_fetch()` so local and cache indexes load before playlist enrichment; added automatic self-healing fallback from corrupted cache files to direct online streaming.
- **Eliminated Duplicate UI Controls**: Removed duplicated Brand header, Account badge, Mini-Player button, Settings button, and Block Mode button from `aside.sidebar`, `.top-header`, and `.player-right`, and removed the redundant Cache Directory section from Settings Tab 1.
- **Light (*Platinum Snow*) & Dark (*Obsidian Metallic*) Theme Parity**: Added `sync_effective_theme` IPC synchronization so the GTK `HeaderBar` follows the active Light/Dark theme even in `"system"` mode; refined Light Mode contrast in Compact Block Mode (`420×650`), Settings cards/inputs, status badges, and waveform unplayed bars; added `.progress-bar-bg` and `.progress-bar-fill` styles; made the 7 Settings navigation tabs wrap cleanly on viewports `<= 720px`.
- **Background Auto-Cache & Concurrent Download Throttling**: Added a 3.5-second playback dwell threshold before triggering background auto-caching and capped concurrent background downloads with `threading.BoundedSemaphore(2)`.
- **Repository Cleanup**: Removed legacy v1 scripts (`ym_gui.py`, `preferences_dialog.py`, `ai_music_helper.py`, `download_ym.sh`, `youtube-music-ultimate.desktop`), outdated v1 planning/docs files, unused duplicate icon assets, dead functions/methods across Python/JS/Rust, and old `.deb` build artifacts.

### Fixed
- **Encrypted Cache Reuse in Telegram Publisher**: Fixed `TelegramChannelPublisher._prepare_media_bundle()` to decrypt `.enc` cached tracks via `CacheManager.get_playback_audio_path()` instead of re-downloading from YouTube.
- **Mid-Stream HTTP 403 Resume**: Preserved playback position (`_pending_retry_seek_sec`) across mid-stream HTTP 403 stream URL refreshes in `PlayerEngine`.
- **Security & Stability Fixes**: Enforced `0o700`/`0o600` permissions on config directories, cookie/header files, local index files, and decrypted `.tmp-playback` files; added `_clean_vid()` path-traversal validation; hardened QR pairing with constant-time PIN verification (`hmac.compare_digest`), attempt limiting, and 180s auto-shutdown; fixed zombie `ffmpeg` cleanup in `_VideoTranscodeHandler`; added a 2.0s read timeout in `RustQueueBridge`; and ensured GTK thread safety via `GLib.idle_add` in `_handle_play_from_source`.

---

## [0.8.2] - 2026-11-17

### Исправлено и улучшено
- **Удаление дублей автора в названии треков для Telegram и AI-очистки (`strip_duplicate_artist_from_title`)**:
  - В [ai_metadata_cleaner.py](ai_metadata_cleaner.py) добавлена умная функция `strip_duplicate_artist_from_title` (с нормализацией `ё`/`е` и латинских диакритик при сохранении кириллической `й`), распознающая и удаляющая дубли исполнителя из названия трека:
    - групповые/коллабные префиксы вида `Потап и его команда - Море пенится` при исполнителе `Потап`,
    - вложенные повторы (`Artist - Artist & Co - Track`), мультиартистов (`IVAN LIULENOV, DAMNITSKYI - ...`), хвостовые суффиксы (`Название - Автор`, `Название (Автор)` без затрагивания `(Live)`/`Remix`), разделители `—`, `–`, `:`, `//`, кавычки `«...»`.
  - Встроена 5-уровневая защита: локальная очистка, пост-обработка ответов Gemini, автоисправление ранее закэшированных записей в `get_cached_clean`, `_prepare_track_for_export` в `ym_app_v2.py` (даже при выключенном AI) и `TelegramChannelPublisher._normalize_outgoing_track` в `telegram_publisher.py` перед записью тегов (`\xa9nam`, `TIT2`, `title`), формированием подписи и отправкой `sendAudio`.
  - В `needs_ai_cleanup` проверка перенесена с сырого `raw_title` на `local_title`, чтобы уже чисто очищенные локально треки `Автор - Название` не тратили дневную квоту Gemini API.

---

## [0.8.1] - 2026-11-17

### Исправлено и ускорено
- **Мгновенный старт выгрузки плейлиста в Telegram без зависания на 3% (JIT AI Lookahead)**:
  - Ранее при активной ИИ-очистке публикация большого плейлиста (1500+ треков) сначала очищала *весь* список пакетами с паузами соблюдения RPM и 3-кратными ретраями при `429` до отправки 1-го трека, из-за чего прогресс надолго замирал на `3%`.
  - Теперь `YMModernApp._handle_publish_playlist_telegram` очищает метаданные скользящим окном **не более 20 ближайших неотправленных треков** (`tracks[idx : min(total, idx + 20)]`) с `fail_fast_on_429=True` (`max_attempts=1`). Первая песня начинает отправляться в Telegram за 1–2 секунды, уже опубликованные треки пропускаются без расхода квоты Gemini, а при `429` (исчерпании RPM/RPD) выгрузка не ждёт и не тормозит — треки мгновенно очищаются локальными правилами (`RegEx`), пока AI-пакетная очистка остаётся доступной без ожидания.
- **Прозрачный статус пауз `429` в `ai_metadata_cleaner.py`**:
  - `_call_gemini_batch` получил параметры `max_attempts` и `progress_cb`, чтобы при ручной/фоновой полной очистке сразу показывать пользователю статус ожидания квоты `429` вместо молчаливого простоя, а `clean_playlist_tracks(..., fail_fast_on_429=False)` поддерживает быстрый выход при фоновой потоковой публикации в Telegram.

---

## [0.8.0] - 2026-11-17

### Added
- **Умная AI- и RegEx-очистка метаданных треков (`ai_metadata_cleaner.py`, `ym_app_v2.py`, `ui/index.html`, `ui/app.js`)**:
  - Новая вкладка **Настройки → AI Очистка**: шифрованное хранение `Gemini API Key` (`Fernet`, `0o600`), автоматическое подтягивание актуального списка моделей напрямую из Google API (`GET /v1beta/models`), классификация и сортировка по бесплатным квотам (`RPM / TPM / RPD`), с возможностью скрыть модели с нулевой бесплатной квотой (`0/0`).
  - Двухуровневая очистка названий и исполнителей: мгновенные локальные правила без API (хэштеги `#...`, URL, эмодзи, `_NEWS`/`VEVO`/`- Topic`, `ВЖИВУЮ`/`Official Video`, дубликаты исполнителя и капс) + пакетная нормализация через Gemini батчами (до 28 треков за запрос для Flash-Lite / по 10 для Gemma 4) с соблюдением лимитов `RPM` и обработкой `429 RetryDelay`.
  - Постоянный локальный кеш очищенных названий (`ai_clean_cache.json`): чистые `Исполнитель` и `Название` подставляются в интерфейс плеера (`ai_apply_in_ui`) и автоматически применяются при отправке в **Telegram-канал** (`ai_auto_clean_export`) без изменения оригиналов на сервере.
  - В контекстное меню трека добавлен пункт **«Очистить название (AI)»**.

---

## [0.7.3] - 2026-11-17

### Added & Fixed
- **Умное определение постов, удалённых вручную из Telegram-канала (`telegram_publisher.py`, `ym_app_v2.py`, `ui/index.html`, `ui/app.js`)**:
  - При каждой успешной публикации трека теперь сохраняется его `message_id` (`{ key: message_id }`) в `telegram_posted.json`.
  - При проверке дубликатов перед пропуском трека бот проверяет актуальность поста в канале (`editMessageReplyMarkup` с пустой разметкой без каких-либо изменений в канале): если пост был удалён пользователем (`message to edit not found`), запись автоматически очищается и трек **повторно публикуется в канал**.
  - Кнопка **«Сбросить»** рядом с «Пропускать дубликаты в канале» показывает число отслеживаемых постов (`Сбросить (N)`) и сразу очищает журнал для всех alias-идентификаторов канала (`-100...` и `@username`).

---

## [0.7.2] - 2026-11-17

### Fixed
- **Выбор плейлиста и формата в Настройках → Telegram (`ui/index.html`, `ui/app.js`, `ui/style.css`)**:
  - Исправлен вызов `toggleCustomDropdown(...)` в `#tg-playlist-custom-select` и `#tg-format-custom-select` (ранее передавался лишний аргумент `event`, из-за чего меню выбора плейлиста и формата не открывалось).
  - Добавлена поддержка обоих форматов вызова в `toggleCustomDropdown` и автоматическое открытие списка вверх (`.open-up`), если элемент расположен у нижнего края окна настроек.

---

## [0.7.1] - 2026-11-17

### Added & Fixed
- **Поддержка приватных Telegram-каналов (`telegram_publisher.py`, `ym_app_v2.py`, `ui/index.html`, `ui/app.js`)**:
  - Автоматическая конвертация ссылки на любое сообщение приватного канала (`https://t.me/c/123456789/5` → `-100123456789`).
  - Добавлено автообнаружение приватного канала через события бота (`getUpdates`: добавление бота в канал, пост в канале или пересланное боту сообщение) — срабатывает автоматически при вставке invite-ссылки `t.me/+...` или по кнопке **«Найти авто»**, автоматически подставляя и сохраняя `-100...` ID канала.
  - Понятная инструкция при неразрешимых invite-ссылках `t.me/+...` вместо сырого `Bad Request: chat not found`.

---

## [0.7.0] - 2026-11-17

### Added
- **Интеграция с Telegram-ботом и публикация в Telegram-канал (`telegram_publisher.py`, `ym_app_v2.py`, `ui/index.html`, `ui/app.js`, `ui/style.css`)**:
  - Новая вкладка **Настройки → Telegram**: настройка `Bot Token` (@BotFather) и `@username` / `-100...` канала с кнопкой **«Проверить подключение»** (`getMe`, `getChat`, `getChatMember` проверка прав администратора канала на публикацию).
  - Посты отправляются от имени самого канала (если бот добавлен администратором с правом публикации и в канале отключена «Подпись авторов») через `sendAudio` с прикреплённой квадратной JPEG-миниатюрой (`<= 320×320`), исполнителем (`performer`), названием (`title`), длительностью, шаблоном подписи (`{artist}`, `{title}`, `{album}`, `{url}`) и вшитыми метаданными + обложкой внутри файла.
  - Выбор формата для публикации в Telegram: **M4A (AAC • Рекомендуется)**, **Opus (Компактный + обложка)** или **MP3 (320 kbps)** с автоматической адаптацией размера в лимит Bot API (50 МБ).
  - Поддержка **публикации одного трека** через контекстное меню (`⋮` → *Опубликовать в Telegram-канал*) и **пакетной выгрузки выбранного плейлиста** прямо из Настроек → Telegram (с паузой, остановкой, защитой от `429 Retry-After` и пропуском уже отправленных дубликатов).
  - Токен Telegram-бота шифруется на диске с помощью `cryptography.fernet` (права `0o600`), маскируется в интерфейсе и санитизируется в сообщениях об ошибках.

### Fixed
- **Оптимизация размера локально сохраняемых файлов `.opus` (`ym_app_v2.py`, `cache_manager.py`)**:
  - Убрана лишняя перекодировка вверх `--audio-quality 0` (~270 kbps) и вшивание огромной raw PNG-обложки `1280×720` при локальном сохранении `.opus` — теперь используется нативный поток YouTube Opus (`251` / `128K`) и сжатая квадратная обложка JPEG до 600px с сохранением всех тегов (название, исполнитель, текст, обложка и ссылка на YouTube Music).

---

## [0.6.5] - 2026-11-17

### Performance & Reliability
- **Пул DOM-строк с O(1) виртуализацией вместо пересоздания таблицы при прокрутке (`ui/app.js`, `ui/style.css`)**:
  - Заменено разрушительное `rowsHost.innerHTML = ''` при каждом шаге скролла на фиксированный рециклируемый пул из ~25–30 DOM-строк с точным шагом (`stride = height + 4px`), `transform: translate3d(...)`, `contain: strict` и `overflow-anchor: none` — устранён бесконечный цикл сдвига высоты и переполнения памяти при прокрутке больших плейлистов (1500+ треков).
  - Добавлены делегированные обработчики событий на контейнер таблицы вместо сотен подписок на каждую строку.
  - Во время активного скролла (`body.is-scrolling`) временно отключаются дорогостоящие `backdrop-filter`, CSS-переходы и `pointer-events`, а загрузка удалённых обложек выполняется после кратковременной стабилизации скролла (`85ms`).
- **Компактные превью обложек для списков (`ytmusic_client.py`, `ui/app.js`)**:
  - Для таблиц плейлистов и поиска выбираются компактные миниатюры (~120px вместо 544px+), а для активного трека в плеере автоматически подгружается обложка высокого разрешения.
- **O(1) обновление статусов треков и оконная очередь (`ui/app.js`)**:
  - Массовые пересчёты статусов кеша и локальных файлов заменены на O(1) `Set`-индексы и обновление только видимых строк; длинная очередь (1500+ треков) рендерится окном вокруг текущего трека.
- **Защита WebKitGTK от зависаний и крашей на Wayland (`ym_app_v2.py`)**:
  - Для Linux Wayland включён `WEBKIT_DISABLE_DMABUF_RENDERER=1` по умолчанию (устраняет зависание/краш `WebKitWebProcess` с пустым экраном при интенсивном скролле), подключён `CacheModel.DOCUMENT_VIEWER`, настроен `WebKit2.MemoryPressureSettings`, политика `ON_DEMAND` и автоматическое самовосстановление UI (`web-process-terminated`) с сохранением воспроизведения звука и текущего плейлиста.
  - Повторно открываемые уже закешированные плейлисты загружаются мгновенно из памяти.

### Security
- Усилена защита локального QR-сервера сопряжения (`hmac.compare_digest` для PIN, ротация после 8 неверных попыток, `html.escape`, заголовки `X-Frame-Options: DENY` / `X-Content-Type-Options: nosniff`) и санитизация SVG / URL обложек на фронтенде (`renderSafeQrSvg`, `sanitizeImageUrl`).
- Ограничен размер словаря локальных токенов обложек (`OrderedDict` до 4096 записей), добавлены таймауты системных вызовов (`lsblk`) и глобальный UI-перехватчик ошибок.

---

## [0.6.4] - 2026-11-16

### Added
- **Адаптивные настройки для узких, модульных (~420px) и мобильных экранов (`ui/style.css`, `ui/index.html`)**:
  - На экранах `<= 720px` левая навигация настроек автоматически превращается в аккуратную горизонтальную панель вкладок сверху, а карточки и кнопки Rclone выстраиваются без вертикального сжатия и переносов внутри кругов.
  - В блоке Rclone и окне QR-сопряжения добавлено заметное уведомление о том, что ПК и телефон/ТВ должны находиться в **одной Wi-Fi сети**; поля ручного ввода Client ID / Token скрыты в сворачиваемый аккордеон `<details>`.
- **Обновлённая панель управления воспроизведением (`ui/index.html`, `ui/app.js`, `ui/style.css`)**:
  - Добавлена кнопка **Стоп** (`#btn-stop`, `#mnp-btn-stop`) для остановки воспроизведения со сбросом позиции в `00:00` без потери текущего трека и очереди.
  - Правые кнопки (`Очередь`, `Текст`, `Звук`) приведены к единому стилю с иконками-плитками транспортных контролов (`.btn-deck-tile`).
  - Нажатие на кнопку звука в компактном/модульном виде открывает интерактивную стеклянную плашку громкости (`#volume-popover`) с перетаскиваемым слайдером, процентами и быстрым отключением звука.
- **Системный трей и сворачивание при закрытии (`tray_manager.py`, `ym_app_v2.py`, `MainWindow.cs`, `AppController.cs`)**:
  - Добавлен кроссплатформенный трей (`NovusTrayManager` через AyatanaAppIndicator3 / pure `Gio.DBus` `StatusNotifierItem` + `com.canonical.dbusmenu` / `Gtk.StatusIcon` на Linux и `NotifyIcon` на Windows) с управлением треком (`Пауза`, `Стоп`, `Пред`, `След`, `Показать/Свернуть`, `Выход`) и переключателем в настройках.

### Changed
- Убран фильтр по плейлистам (`#collection-playlist-custom-select`) в разделе «Коллекция» (`Кеш` и `Локально`); привязка очереди выбранного плейлиста в «Медиатеке» сохранена.
- Переключатель трея `#setting-tray-row` скрыт на мобильных платформах.

### Fixed
- **Перемотка на паузе (`player_engine.py`, `WindowsAudioPlayerService.cs`, `ui/app.js`)**:
  - Исправлена перемотка трека во время паузы (`Gst.SeekFlags.FLUSH | Gst.SeekFlags.ACCURATE` и немедленный emit `on_position_change` на Linux, прямой вызов `PositionChanged` на Windows и мгновенная перерисовка вейвформа/ползунка в UI).

---

## [0.6.3] - 2026-11-16

### Fixed
- **Критическое исправление инициализации интерфейса (`ui/app.js`)**:
  - Устранена ошибка JS Temporal Dead Zone (`ReferenceError: Cannot access 'serverDeviceType' before initialization`), возникавшая при первом вызове `syncPlatformClasses()` до объявления глобальных переменных состояния ниже по файлу и блокировавшая загрузку обработчиков интерфейса и IPC `ui_ready`.
  - Убрана дублирующая строка `Аккаунт` в левой боковой панели [ui/index.html](file:///home/kyryll/Scripts/Novus%20Media/ui/index.html) (вход в раздел остаётся через кружок профиля в левом верхнем углу окна и карточку профиля).

---

## [0.6.2] - 2026-11-16

### Added
- **Кружок аккаунта и настройки в левом углу шапки окна (`ym_app_v2.py`, `ui/index.html`, `ui/app.js`, `ui/style.css`)**:
  - На Linux в левую часть `Gtk.HeaderBar` рядом с `Novus Media` добавлены круглая кнопка аккаунта (с инициалами и индикацией авторизации) и кнопка настроек; на Windows / macOS / Web / Mobile — кнопки `#btn-header-account` и `#btn-header-settings` в левом углу `.top-header`. Вкладка `Аккаунт` убрана из полосы вкладок `#modular-compact-nav`.
- **Дисклеймер авторских прав для локального скачивания (`ui/index.html`, `ui/app.js`)**:
  - Включение свитч-переключателя `#setting-enable-local-mode` теперь требует подтверждения в модальном окне `#modal-local-disclaimer` с предупреждением о защите авторских прав и запрете распространения.
- **Пакет UX-улучшений (`ui/app.js`, `ui/style.css`, `ui/index.html`)**:
  - Единые горячие клавиши на ПК (`Space`, `J`/`←`, `L`/`→`, `M`, `Q`, `Ctrl+F`, `Esc`).
  - Постоянно видимый кликабельный бейдж активного таймера сна (`#player-sleep-badge`, `#mnp-sleep-timer-pill`, точка-индикатор на кнопке `⋮`).
  - Горизонтальный адаптивный layout `#mobile-now-playing-sheet` для альбомной ориентации телефонов и Split View на планшетах.
  - Режим навигации 10-foot UI / D-Pad для Smart TV и ТВ-приставок (`.platform-tv`, `:focus-visible`).
  - Shimmer-скелетоны загрузки плейлистов, запоминание позиции скролла и кнопка быстрого перекеширования в тосте при ошибке воспроизведения.

### Fixed & Changed
- **Восстановлена панель управления и горизонтальная волна в блочном режиме (`ui/style.css`)**:
  - Устранено конфликтное правило `flex-wrap: wrap` и `flex: 1 1 100%`, из-за которого элементы управления, переключатель режимов и полоса волновой диаграммы вытеснялись за правый край блока плеера.
- **Единый стиль Novus Glass для всех выпадающих списков и форм (`ui/index.html`, `ui/app.js`, `ui/style.css`)**:
  - Все оставшиеся нативные `<select>` (`#collection-playlist-filter`, `#video-remote-select`, `#account-playlist-sort`, `#rclone-cfg-type`) заменены на кастомный компонент `.custom-select-wrap` в тёмном/светлом стиле Novus Media.
- **Более компактная «Медиатека» (`ui/style.css`)**:
  - Уменьшены высота `#library-hero` и ширина/отступы карточек плейлистов в сетке и модульном списке.

---

## [0.6.1] - 2026-11-16

### Added
- **Таймер сна в меню «⋮» расширенного плеера и треков (`ym_app_v2.py`, `AppController.cs`, `WindowsAudioPlayerService.cs`, `ui/index.html`, `ui/app.js`)**:
  - Добавлен пункт **«Таймер сна»** в `#custom-context-menu` (доступен через кнопку `⋮` в `#mobile-now-playing-sheet` и у любого трека) с модальным окном `#modal-sleep-timer` (Выкл, До конца трека, 10/15/30/45/60 мин и произвольные минуты) и живым обратным отсчётом.
- **Фильтр по плейлистам в разделе «Коллекция» (`ym_app_v2.py`, `AppController.cs`, `ui/index.html`, `ui/app.js`)**:
  - Добавлено сохранение индекса `playlists_track_map.json` и селектор `#collection-playlist-filter` для фильтрации вкладок **«Кеш»** и **«Локально»** по любому плейлисту пользователя.
- **Иконочные статусы 6 форм-факторов устройств (`ui/app.js`, `ui/style.css`)**:
  - Локально скачанные треки отображаются салатовым цветом без текста с автоопределением устройства: Ноутбук, ПК, Телефон, Планшет, Телевизор и ТВ-Приставка (`detectDeviceFormFactor`).

### Changed
- **Выравнивание таблицы треков и минималистичная шапка плейлиста (`ui/index.html`, `ui/style.css`, `ui/app.js`)**:
  - Кнопки «Слушать всё», «Перемешать» и круговая кнопка «Кеш» с динамическим кольцом прогресса (без цифр и процентов в самой кнопке) перенесены к названию плейлиста; в `#pl-meta` оставлено только количество треков.
  - Кнопка скачивания плейлиста удалена из шапки плейлиста; колонки `#`, `Композиция`, `Альбом`, `Статус`, `Длительность` и `⋮` переведены на единую CSS Grid-сетку для точного совпадения заголовков и строк.
- **Оптимизация нижней панели плеера и расширенного Now Playing (`ui/index.html`, `ui/style.css`, `ui/app.js`)**:
  - Удалена дублирующая кнопка переключения режима окна `#btn-header-modular` из верхней шапки; оставлен единый переключатель `#btn-player-layout-toggle` в панели плеера.
  - Убран дублирующий текст исполнителя под контролами при отсутствии синхронизированных текстов; время и караоке-строка вынесены в правую часть блока трека, а все кнопки управления, очередь, текст, громкость, переключение режима и разворачивание размещены на одной строке.
  - В расширенном плеере `#mobile-now-playing-sheet` переключатель `[Плеер | Очередь | Текст]` выровнен по центру между `[⌄]` и `[⋮]`, индикатор «Сейчас играет» перенесён под переключатель, удалена нижняя строка `.mnp-quick-actions`, а иконка статуса рядом с названием/исполнителем при онлайн-стриминге (`YouTube Music`) стала кликабельной для ручного кеширования трека.

### Fixed
- **Фиксация очереди выбранного плейлиста (`ym_app_v2.py`, `AppController.cs`, `ui/app.js`)**:
  - Добавлен счётчик последовательности `_playlist_load_seq` / `_playlistLoadSeq` и пиннинг `playlist_id` + `video_id` при воспроизведении, устраняющий переключение на «Мне нравится» при запуске треков из другого открытого плейлиста.

---

## [0.6.0] - 2026-11-16

### Added
- **Иконочная кнопка переключения режима окна (`Блочный вид ↔ Полный вид`)**:
  - Верхняя кнопка `#btn-header-modular` переведена в компактный иконочный формат без лишнего текста, динамически обновляя иконку и всплывающую подсказку (`title`) при наведении.
- **Контекстно-приоритетный поиск без лишних фильтров (`ym_app_v2.py`, `AppController.cs`, `ui/app.js`)**:
  - Удалена дублирующая строка поиска внутри чипа «Кеш»; добавлена фоновая индексация всех плейлистов пользователя для единой поисковой строки.
  - Из раздела **«Медиатека / Плейлисты»** поиск идёт строго в порядке: **1) Текущий плейлист → 2) Все плейлисты → 3) YouTube Music (Онлайн)**.
  - Из раздела **«Коллекция»** поиск идёт в порядке: **1) Активный чип («Кеш» или «Локально») → 2) Плейлисты → 3) YouTube Music (Онлайн)**.
- **Модульная активация «Локально» и «Видео (Rclone)» + QR-подключение для ТВ/смартфонов (`ym_app_v2.py`, `rclone_video_manager.py`, `WindowsRcloneService.cs`, `ui/index.html`)**:
  - Чип **«Локально»**, кнопки скачивания в открытые файлы и вкладка **«Видео (Rclone)»** скрыты по умолчанию и включаются переключателями в Настройках.
  - В Настройках добавлены форма ввода `Client ID / Secret / OAuth Token` для Rclone, импорт из локального конфига, OAuth-авторизация через браузер, пошаговая инструкция и встроенный **QR-код сервер в локальной сети (LAN)** для быстрого подключения ТВ-приставок, Smart TV, телефонов и ПК.
  - Добавлен кроссплатформенный выбор папок для защищённого кеша («Кеш») и открытых аудиофайлов («Локально»).

### Changed
- **Минималистичная коллекция («Кеш» | «Локально») и обновлённый «Аккаунт» (`ui/index.html`, `ui/style.css`)**:
  - Чип «Кеш» перенесён на первое место; кнопки управления («Слушать», «Автокеш», «Папка», «Очистить», «Скачать», «M3U8», «Обновить») размещены иконками с tooltip на одной строке с чипами.
  - Экран **«Аккаунт»** переработан в компактную карточку профиля, 3 ключевых индикатора (Синхр., Плейлисты, Обновлено) и аккуратную панель действий.
- **Фирменная шапка окна Novus Media (`ym_app_v2.py`, `DwmWindowHelper.cs`)**:
  - На Linux реализован кастомный тёмный/светлый `Gtk.HeaderBar` с логотипом, версией и стилизованными кнопками `−`, `□`, `×`.
  - На Windows добавлена нативная покраска рамки и заголовка окна через DWM (`DWMWA_CAPTION_COLOR`, `DWMWA_BORDER_COLOR`) в палитре Novus Media.

---

## [0.5.9] - 2026-11-16

### Added
- **Единый раздел «Коллекция» с двумя чипами (`Локально` | `Кеш`) и мини-модалкой скачивания (`ui/index.html`, `ui/style.css`, `ui/app.js`)**:
  - Разделы «Коллекция» и «Офлайн-музыка» объединены в единый раздел «Коллекция» (`#view-local`) с мгновенным переключением чипами **«Локально»** (`#chip-col-local`) и **«Кеш»** (`#chip-col-cache`).
  - Раздел «Скачивания» перенесён внутрь `Коллекция → Локально` в виде кнопки **«Скачать»**, открывающей компактную мини-модалку выбора плейлиста, аудиоформата (`Opus` / `AAC`) и правового предупреждения (`#modal-download-warning`).
- **Приоритетный поиск по активному плейлисту в верхней поисковой строке (`ui/index.html`, `ui/app.js`)**:
  - Внутренний дублирующий фильтр в открытом плейлисте убран; верхняя строка поиска (`#global-search-input`) теперь мгновенно ищет в порядке приоритета: **1) В активном плейлисте** (`#search-group-playlist-wrap`), **2) В локальной коллекции и кеше**, **3) В YouTube Music (онлайн)**.

### Changed
- **Компактные горизонтальные карточки плейлистов и иконочная панель открытого плейлиста (`ui/index.html`, `ui/style.css`, `ui/app.js`)**:
  - Большие квадратные карточки плейлистов заменены на компактные горизонтальные строки (`54px` высотой) с обложкой `40×40`, метаданными и кнопкой быстрого запуска.
  - Панель открытого плейлиста (`#library-hero`) сжата в одну компактную строку (`56px`) с кнопкой возврата `[←]`, мини-обложкой и лаконичными иконочными кнопками (`▶`, `🔀`, `Кеш`, `ПК`, `Аудит`, `⚙`), не занимающими полезную площадь списка треков.

### Fixed
- **Стабильная работа «Блочного вида» и музыкальной волны при сжатии окна на ПК (`ui/app.js`, `ui/style.css`)**:
  - Исправлена функция `isMobileViewport()` в `ui/app.js`, из-за которой сужение десктопного окна уже `768px` (включая блочный режим `420×650`) ошибочно активировало мобильный класс `.platform-mobile`, скрывало музыкальную диаграмму воспроизведения (`#track-waveform-canvas`) и убирало кнопку переключения «Полный вид / Блочный вид» (`#btn-header-modular`).
  - Навигационная панель модульного режима (`.modular-compact-nav`) переведена на 4-колоночную сетку (`Медиатека | Коллекция | Видео | Аккаунт`) без обрезания вкладок даже при минимальной ширине окна `360px`.
  - Полностью исключены любые упоминания сторонних брендов плееров в коде, стилях и локализациях.

---

## [0.5.8] - 2026-11-16

### Added
- **Трансформация всего окна плеера в компактный модульный режим Novus (`ym_app_v2.py`, `MainWindow.cs`, `AppController.cs`, `ui/index.html`, `ui/style.css`, `ui/app.js`)**:
  - Снято жёсткое ограничение минимального размера окна (`860×540` → `360×460` на Linux и Windows).
  - При нажатии на кнопку **«Блочный вид»** (`#btn-header-modular` или `#btn-player-layout-toggle`) всё окно приложения перестраивается в компактный вертикальный блок `420×650`: верхняя мастер-дека с обложкой, музыкальной волной, караоке и транспортом, горизонтальная панель вкладок (`#modular-compact-nav`) и плотный список треков.
- **Верифицированный поиск текстов песен и кнопка «↻ Обновить текст» (`ym_app_v2.py`, `cache_manager.py`, `WindowsYouTubeService.cs`, `ui/app.js`)**:
  - Добавлена фонетическая сверка исполнителя и названия (включая транслитерацию `Zaychik ↔ Зайчик`) через API `lrclib.net` + проверка длительности трека и инвалидация старого неверифицированного кеша текстов (`lyrics_source = "verified_v2"`).

### Changed
- **Уплотнён и очищен от визуального шума основной интерфейс (`ui/index.html`, `ui/style.css`)**:
  - Шапка плейлиста (`#library-hero`) объединена с кнопкой «← Все плейлисты» и сжата по высоте почти в 3 раза, освобождая экран под список треков даже в небольшом окне.
  - Убраны дублирующиеся иконки в верхней панели при открытом сайдбаре и лишние кнопки в полноэкранном Split View.

### Fixed
- **Отображение музыкальной диаграммы (Waveform) в основном окне и полноэкранном плеере (`ui/style.css`, `ui/app.js`)**:
  - Исправлена ошибка, из-за которой `uiAutoPowerSaverActive` / `body.ui-power-saver` отключали `visualizerEnabled` и скрывали `.track-waveform-canvas`, оставляя плоскую линию вместо волны. Добавлен `ResizeObserver` для мгновенной перерисовки волны.
- **Кнопка меню «⋮» (троеточие) в полноэкранном плеере и строках треков (`ui/app.js`)**:
  - Добавлен `e.stopPropagation()` в `openContextForCurrentTrack` и `openContextMenu`, устраняющий мгновенное закрытие меню глобальным обработчиком клика, и повышен `z-index` контекстного меню до `2200`.

---

## [0.5.7] - 2026-11-16

### Added
- **Модульный блочный режим плеера (`body.player-modular-blocks`, `ui/index.html`, `ui/style.css`, `ui/app.js`)**:
  - Добавлен переключатель стиля панели плеера в 1 клик (`#btn-player-layout-toggle`) и в настройках (`#setting-player-layout-select`): **Стандартный** (высота `64px`) или **Блочный (Модульный)** с отдельными модулями трека, пульта управления с цифровым таймером и спектральным скраббером-рельсой сверху.
- **Единый движок аудио-волны `ui/waveform.js` + интеграция в плавающий мини-плеер (`ui/mini.html`, `ui/mini.js`)**:
  - Мини-плеер теперь использует ту же живую диаграмму прокрутки трека (`#mini-waveform-canvas`), поддерживает клик-перемотку (`seek`, включая проброс payload в Windows `MiniPlayerWindow.cs`) и компактный счётчик времени.

### Changed
- **Чистая навигация по плейлистам без дублирования (`ui/index.html`, `ui/app.js`)**:
  - Убран список плейлистов из левого сайдбара и дублирующая полоса чипсов. В «Медиатеке» сначала отображается сетка плейлистов (`#library-playlists-hub`), а при выборе плейлиста верхние карточки скрываются и открывается детальный экран плейлиста (`#library-playlist-detail`) с кнопкой возврата «Все плейлисты».

### Fixed
- **Видимость кнопок «Мини-плеер» и «Настройки» на любых размерах окна (`ui/index.html`, `ui/style.css`)**:
  - Исправлен flex-overflow левого сайдбара (`.sidebar-top { min-height: 0; overflow-y: auto; }` + закреплённый `.sidebar-footer`), а также добавлены быстрые кнопки «Мини-плеер» и «Настройки» в правую часть верхней шапки на ПК.

---

## [0.5.6] - 2026-11-16

### Added
- **Аудио-волна (Waveform Scrubber) прямо в полосе прокрутки трека (`ui/index.html`, `ui/style.css`, `ui/app.js`)**:
  - Спектральная диаграмма трека интегрирована непосредственно в верхний интерактивный скраббер мини-панели (`#scrubber-track` / `#track-waveform-canvas`) и в полосу прокрутки полноэкранного режима `Now Playing` (`#mnp-scrubber-track` / `#mnp-waveform-canvas`).
  - Для каждого трека строится детерминированный сглаженный профиль волны (по метаданным трека), с живым битом у текущей позиции воспроизведения, предпросмотром перемотки при наведении курсора и автоматической адаптацией к тёмной и светлой темам.
  - При отключении визуализатора в настройках или в режиме энергосбережения полоса аккуратно переключается на классический линейный скраббер (`#scrubber-progress` / `#mnp-progress`).

### Changed
- **Упрощена правая зона нижней панели плеера**: удалена отдельная 5-полосная диаграмма `#visualizer-canvas` между кнопкой «Текст» и регулятором громкости.

---

## [0.5.5] - 2026-11-16

### Added
- **Кроссплатформенный UI/UX премиум-уровня для ПК, планшетов и смартфонов (`ui/index.html`, `ui/style.css`, `ui/app.js`, `locales/*.json`)**:
  - **Hero-шапка плейлиста** с обложкой, автором, длительностью, счётчиком и мгновенным фильтром `#playlist-inline-filter` внутри больших плейлистов (`1 500+` треков).
  - **Карусель быстрого доступа (`#library-quick-cards`)** с обложками плейлистов и кнопкой быстрого запуска.
  - **Адаптивный полноэкранный режим `Now Playing Split View`** (2-колоночный на ПК/планшетах: обложка + вкладки `Далее` / `Текст`, и 3-вкладочный `Плеер / Далее / Текст` на Android и iOS).
  - Сворачиваемый Rail-сайдбар (`#btn-toggle-sidebar`), полноширинный верхний скраббер-прогрессбар плеера, колонка «Альбом» и мини-шапка выбранного трека + поиск по артисту в контекстном меню.

### Changed
- **Загрузка полных плейлистов без лимита `500/300` (`ytmusic_client.py`, `ym_app_v2.py`, `WindowsYouTubeService.cs`)**: снято ограничение `limit=500`/`300` (теперь `limit=None` на Linux + continuation-пагинация до 50 страниц на Windows), благодаря чему плейлисты на `1 500+` треков загружаются целиком (устранён обрыв на `683` треках) и передают метаданные обложки/длительности в Hero-шапку.

### Fixed
- **Приоритет воспроизведения «Скачано > Кеш» (`local_music_manager.py`, `player_engine.py`, `ym_app_v2.py`, Windows `Storage`/`Player`/`AppController`)**:
  - Введена мгновенная регистрация скачанных файлов (`local_yt_map.json` + `--print after_move:filepath`) и поиск локального файла не только по `[videoId]` в имени, но и по нормализованному названию и артисту.
  - Локально скачанные треки всегда воспроизводятся с диска, их кеш в `.ymc_vault` принудительно удаляется, а ручное или фоновое кеширование уже скачанных треков не создаёт дубликат и очищает оставшийся кеш.
- **Ошибки GStreamer при быстром переключении треков (`player_engine.py`)**: добавлены сброс очереди сообщений шины (`_flush_bus`) и проверка `_active_bus_req_id == _play_req_id` в `_on_bus_message`, исключающие всплывающие ошибки от предыдущего потока.

---

## [0.5.4] - 2026-11-16

### Added
- **Полноценный Windows-бэкенд (`platform/windows/src/NovusMedia/`)**:
  - Нативный аудиодвижок `WindowsAudioPlayerService` на `Windows.Media.Playback.MediaPlayer` + `SystemMediaTransportControls` (SMTC медиа-клавиши, экран блокировки, авто-кеширование и координация очереди через Rust `ym_core`).
  - `WindowsYouTubeService` (InnerTube HTTP-клиент с `SAPISIDHASH`, поиск, загрузка плейлистов/треков, редактирование/удаление треков, резолвер аудиопотоков через `yt-dlp.exe` / InnerTube `ANDROID_VR`, синхронные тексты LRCLIB).
  - `AuthWebViewDialog` для встроенного входа в YouTube Music с захватом cookie, `MiniPlayerWindow` (плавающий мини-плеер `mini.html`), `WindowsStorageService` (локальная медиатека, `DriveInfo` диски, экспорт `.m3u8`, хранилище `.ymc_vault`), `WindowsRcloneService` (`rclone lsjson` / `serve http` / кеш видео) и `AppController` с полным покрытием IPC-команд UI (73/73).
- **Изоляция и безопасность Windows**: шифрование сессии через Windows DPAPI (`DataProtectionHelper`), Win32 Job Objects (`Win32JobObject`) для авто-завершения дочерних процессов `ym_core_worker.exe` и `rclone.exe`, тёмная рамка окна через DWM (`DwmWindowHelper`) и виртуальный HTTPS-хост WebView2 `https://app.novusmedia.local` с навигационным фильтром.

### Changed & Optimized
- **Сборка и CI Windows (`NovusMedia.csproj`, `build_windows.sh`, `.github/workflows/windows-build.yml`)**: включены `<UseWindowsForms>true</UseWindowsForms>`, `<ApplicationHighDpiMode>PerMonitorV2</ApplicationHighDpiMode>`, `<EnableWindowsTargeting>true</EnableWindowsTargeting>`, исправлены 4-уровневые относительные пути до `.ico`, `ui/`, `VERSION` и `CHANGELOG.md`, CI переведён на `.NET 9.0.x` + Rust toolchain со сборкой release-бинарников `ym_core_cli.exe`/`ym_core_worker.exe`.
- **Мультиформатные иконки `.ico` (`assets/icons/novus-media.ico`, `ui/assets/logo.ico`, `ui/assets/favicon.ico`)**: пересобраны из официальных PNG со слоями `16–128px` в 32-bit BGRA BMP DIB (`wPlanes=1, bpp=32`) и `256px` в PNG для 100% совместимости с WinForms `System.Drawing.Icon` и Windows PE ресурсами.

### Fixed
- **Компиляция и потокобезопасность `Program.cs`, `BridgeFileLogger.cs`, `MainWindow.cs`, `WorkerProcessHost.cs`, `CliProcessRunner.cs`, `BridgeFacadeService.cs`**: устранён конфликт пространств имён `NovusMedia.Application` и `System.Windows.Forms.Application`, вход переведён на синхронный `[STAThread] static void Main` без потери STA, логи/WebView2 перенесены в `%LOCALAPPDATA%\NovusMedia`, исправлены десериализация плоских IPC-сообщений WebView2, проброс флага `shuffle` в `calc_next`/`calc_prev`, `CultureInfo.InvariantCulture` в `plan_cache_evict`, блокировка `SemaphoreSlim` и рестарт зависшего worker-процесса.

---

## [0.5.3] - 2026-11-16

### Added
- **Единый модуль протокола Rust Core (`protocol.rs`) и перенос JNI-моста (`android_jni.rs`)**: консолидированы CLI/Worker парсеры команд (убрано ~300 строк дублирования) и подключён `android_jni` в `src/lib.rs` для корректного экспорта символов на Android (`autobins = false` для беспроблемной работы `cargo build/test`).
- **Полноценная поддержка локализации `i18n` (`RU` / `UK` / `EN`)**: добавлены переводы разделов «Видео (Rclone)», «Аккаунт», «Скачивания», настроек, контекстного меню, мобильного плеера и мини-плеера (`mini.js`), а также подключены подсказки `data-i18n-title` и адаптивные индикаторы `shuffle_state` / `repeat_state`.

### Changed & Optimized
- **Облегчённая и чистая сборка `.deb` (`build_deb.sh` + `build-deb.yml`)**: исключены 402 МБ сборочных артефактов `shared-core/rust-core/target/`, в пакет включаются только собранные release-бинарники `ym_core_cli` и `ym_core_worker` без runtime `cargo build` в `/usr`, добавлены системные зависимости (`gstreamer1.0-gtk3`, `gstreamer1.0-gl`, `python3-cryptography`, `python3-requests`), убран `pip` из `/usr/bin/novus-media`, удалён дублирующий `.desktop`-ярлык, а CI получил инструмент сборки `cargo`/`rustc` и ограниченные права.
- **Ускорение Rust-моста, LRU-эвикта и синхронизации**: эвикт кэша в `CacheEngine` переведён с `O(N²)` на `O(N log N)`, убрано `O(N²)` пересканирование коллекции на каждом треке в `_sync_worker` и пересчёт кэша в `_cache_worker`, синхронные вызовы (`get_rclone_sources`, `get_system_disks`, `rescan_local`, `remove_track_from_playlist`) вынесены в фоновые потоки, добавлена поддержка тегов/обложек `.flac`.
- **Контраст светлой темы (WCAG) и готовность UI к iOS/Mobile**: заменены низкоконтрастные цвета кнопок/спиннера в `[data-theme="light"]` на переменные палитры, разрешено выделение текста в полях ввода, уточнён размер шрифта `16px` и расчёт `100dvh - safe-area-inset-top` для мобильных браузеров и WebView.

### Fixed
- **Устранена 100% нагрузка CPU на старте**: `_bind_vlc_to_draw_area()` переведён с бесконечного polling `GLib.idle_add` до инициализации окна на сигнал `"realize"`.
- **Починен Shuffle, сохранение метаданных и удаление из очереди (`PlayerEngine` + `QueueEngine`)**: исправлен немедленный повтор текущего трека при переходе цикла в `Shuffle + Repeat All`, поддержан `Shuffle` в `prev_index`, сохраняются полные метаданные треков при `enqueue_next`/`enqueue_end` и корректно удаляется последний/текущий трек без сбоя индекса.
- **Безопасность WebView и UI (DOM XSS)**: экранированы все динамические строки/URL, inline `onclick`/`onerror` заменены на безопасную привязку обработчиков, в WebKit2 отключён `allow_universal_access_from_file_urls`, добавлен фильтр навигации `decide-policy`, а секреты и конфиги (`config.json`, `auth_headers.json`, `cookies.sqlite`) защищены правами `0o600`/`0o700`.
- **Исправления YouTube Music, кэша и подпроцессов**: убраны захардкоженные пути `/home/kyryll/...` и `/media/kyryll/...`, восстановлено сохранение `cache_dir`, `cache_quota_gb` (включая `0` — без ограничений), `auto_cache`, `volume`, `replaygain` и `hidden_playlists`, разблокировано ручное кэширование при выключенном автокэше, исправлена передача dict-элементов при удалении треков из плейлистов (`ytmusic_client.py`), устранены утечки pipe (`RustQueueBridge`, `ffmpeg`, `rclone`), неэкранированные `file://` URI, ложные подстроковые срабатывания «Скачано», краши/ошибки `mpris_service.py` и `confirmCacheInsteadOfDownload()`.

---

## [0.5.2] - 2026-11-16

### Added
- **Умный аппаратный мост для AV1 (`libdav1d` -> Intel `h264_vaapi` / `x264` fallback)**: автоматическое определение видеопотоков `av1` (где системный `libaom av1dec` работает в 0.39× реального времени) и потоковое ускорение через SIMD `libdav1d` + GPU VA-API (`24+ FPS`, `0` потерянных кадров) с сохранением всех аудиодорожек, субтитров, паузы и перемотки.

### Fixed
- **Отключение субтитров в видеоплеере (`Отключено`)**: при выборе `-1` теперь снимается флаг `GST_PLAY_FLAG_TEXT` (`0x00000004`) на `playbin` (ранее `current-text=-1` означало auto-select и оставляло первую дорожку субтитров видимой).
- **Убрана дублирующая кнопка очереди в нижней панели плеера на ПК**: исправлен порядок каскада CSS (`button.mobile-mini-queue-btn { display: none; }` после `.btn-icon { display: flex; }`).
- **Синхронизация модального окна текста песни (`#lyrics-modal`)**: при переключении трека с открытым окном текста сразу обновляются заголовок, исполнитель и содержимое, игнорируются запоздалые ответы от предыдущего `videoId`, а при отсутствии текста больше не остаётся вечное «Загрузка текста...».

---

## [0.5.1] - 2026-11-16

### Added
- **Кинотеатр Novus Media (Obsidian Cinema HUD)** в встроенном видеоплеере (`ym_app_v2.py`):
  - Темный `Gtk.Overlay` интерфейс с плавающими верхним и нижним стеклянными HUD-барами, автоскрытием элементов и курсора через 2.8 с, центральным OSD-уведомлением и темным стилизованным оформлением селекторов, скраббера и кнопок.
  - Навигация по сериям/списку папки и видеокеша прямо из плеера (`⏮`/`⏭`, `PageUp`/`PageDown`, `N`, автопереход на следующую серию), переключение скорости воспроизведения (`0.5×`–`2.0×`, `[`/`]`), кнопка `Mute` (`M`), быстрый переход во внешний плеер (`↗ VLC`), управление двойным кликом/колесом мыши и расширенными горячими клавишами (`Space`/`K`, `J`/`L`, `←`/`→`, `Shift+←`/`→`, `A`, `S`, `F`/`F11`, `P`).

### Changed & Optimized
- **Аппаратный GPU OpenGL видеорендеринг (`gtkglsink` + `glsinkbin`)**: перевод встроенного GStreamer-видеоплеера с программного CPU `gtksink` на аппаратный OpenGL `glsinkbin(gtkglsink)` с безопасным авто-откатом на `gtksink` при ошибках GL.
- **Потоковая буферизация и стриминг Rclone**: добавлены расширенные буферы `playbin` (`buffer-duration=8s`, `buffer-size=24MB`, `ring-buffer-max-size=64MB`, `deinterlace`, `subtitle-font-desc="Sans Bold 21"`), а `rclone serve http` оптимизирован под потоковый просмотр (`--vfs-read-chunk-size 8M --buffer-size 64M --vfs-read-wait 10ms`).

### Fixed
- **Устранено падение картинки до ~1 FPS при нормальном звуке**: отключен агрессивный `qos` и расширен `max-lateness` (`120 мс`) у GTK video sink (ранее дефолт `max-lateness=5мс + qos=True` на программном `gtksink` сбрасывал все delta-кадры до следующего keyframe), добавлен debounce/фиксация перемотки при отпускании ползунка (устранен шторм HTTP range-запросов) и асинхронная обработка `BUFFERING` через `GLib.idle_add` только для HTTP-потоков.

---

## [0.5.0] - 2026-11-15

### Added
- **Адаптивный мобильный UI/UX (Android, iOS, Touch и узкие экраны)**:
  - Единый 5-вкладочный нижний док-бар (`Медиатека`, `Поиск`, `Коллекция`, `Офлайн`, `Аккаунт`) с безопасными отступами `env(safe-area-inset-*)`, плавающий мини-плеер с верхней полосой прогресса и полноэкранный выдвижной плеер (`#mobile-now-playing-sheet`) с обложкой, скраббером, караоке-строкой и быстрыми действиями.
  - Мобильная кнопка настроек (`#mobile-settings-btn`) и быстрого перехода к видео в верхней шапке, горизонтально прокручиваемые чипсы плейлистов, кнопка меню `⋮` на каждой строке трека, long-press и iOS/Android bottom action sheet (`#custom-context-menu` + `#mobile-sheet-backdrop`).
  - Автоматическое определение мобильного вьюпорта (`syncPlatformClasses()` для Android WebView, iOS Safari/WKWebView и ширины `<= 768px`).
- **Переработка раздела настроек и единая система профилей**:
  - Убран неработающий фиктивный раздел AI/Gemini без бэкенда; все параметры сгруппированы в 5 реальных функциональных вкладок (`Медиатека`, `Кеш и диски`, `Аудио и плеер`, `Внешний вид`, `О программе`) с живой сводкой кеша, сохранением настроек караоке и визуализатора в `localStorage`, синхронизацией переключателя совместимости видео (`Wayland / Linux`) и кнопкой сброса UI (`resetUiPreferences()`).
- **Синхронизация официальных иконок `assets/icons` для всех платформ**:
  - Сгенерированы все плотности Android `mipmap-{mdpi,hdpi,xhdpi,xxhdpi,xxxhdpi}/{ic_launcher,ic_launcher_round}.png`, обновлены adaptive-иконки (`#06101F`, безопасный отступ `16dp`, `ic_launcher_round.xml`, `AndroidManifest.xml`).
  - Подключены `<link rel="icon">` / `apple-touch-icon` (`logo.svg`, `favicon.ico`, `logo-256.png`) в `ui/index.html` и `ui/mini.html`, обновлен Windows `.csproj` и `MainWindow.LoadAppIcon()` для нативных `.ico`-ресурсов.

### Changed
- **Enterprise-тема «Obsidian Metallic & Platinum Snow Glass»**: обновлена цветовая палитра, контрастные стальные границы, градиентные основные кнопки (`.btn-primary`) и оформление карточки «О программе» в едином стиле с официальной иконкой Novus Media.

### Fixed
- Исправлена вертикальная поломка строк папок в «Видео (Rclone)» за счет явного flex-макета `.rclone-folder-row` / `.rclone-folder-icon` и свернута техническая диагностика Rclone в компактный `<details>`.
- Исправлена пустая правая ячейка в сетке статистики аккаунта на десктопе через адаптивный `.account-stats-grid` (`repeat(auto-fit, minmax(150px, 1fr))`).
- Устранены теснота и обрезание верхних строк в модальном окне «Параметры плеера» (`min(860px, 94vw)` × `min(620px, 86vh)`, однострочное меню слева, горизонтальные пилюли вкладок на мобильных).

---

## [0.4.0] - 2026-11-15

### Added
- **Автономная авторизация YouTube Music без кеша Google Chrome**: добавлено встроенное изолированное окно входа `WebKit2.WebView` (`webkit_auth/cookies.sqlite`), импорт сессии напрямую из браузера приложения (`YTMusicDirectClient.import_browser_cookies()`), а также поддержка импорта Netscape `cookies.txt` и raw-заголовков `Cookie:`.
- **Полноценный облачный проводник и стриминг Rclone (`Видео (Rclone)`)**:
  - Навигация по папкам и подпапкам (`RcloneVideoManager.list_entries()`), кликабельные «хлебные крошки», кнопка `⬆ Вверх`, фильтр внутри папки и рекурсивный поиск видео.
  - Прямое воспроизведение видео без монтирования диска в ОС (`rclone mount`) через локальный loopback `rclone serve http` (`get_stream_source()`, `stop_serve()`).
  - Встроенный Wayland-совместимый видеоплеер на GStreamer `playbin + gtksink` (с fallback на `libVLC` для X11 и внешний плеер) и автоматической паузой аудиоплеера.
  - Live-прогресс кеширования видео (`--use-json-log`), кнопка отмены, удаление отдельных видео из видеокеша, полная очистка видеокеша и синхронизация корня видеокеша при смене диска в настройках.
- **Локальный HTTP-сервер обложек**: `_start_cover_server()` с RAM LRU на 160 обложек (`/cover/cache/<vid>`, `/cover/local/<tok>`).
- **Единое версионирование и встроенный Changelog**: добавлены корневой файл `VERSION`, `CHANGELOG.md`, скрипт синхронизации `./bump_version.sh` и отображение актуальной версии и истории изменений во вкладке «О программе».

### Changed & Optimized
- **Быстрый запуск онлайн-аудио**: разрешение аудиопотоков переведено с холодного CLI `yt-dlp` (~8.5 с) на потоковый TTL-кеш `_stream_cache` + переиспользуемый in-process `yt_dlp.YoutubeDL` (`fast_android_vr` -> `auth_node`) с предзагрузкой следующего трека очереди.
- **Оптимизация IPC, памяти и UI**:
  - Из списков треков и очереди (`_enrich_track(..., inline_cover=False)`) убраны многомегабайтные base64 `data:image` обложки; изображения загружаются лениво (`loading="lazy" decoding="async"`).
  - Обновление очереди отправляет только `queue_index_changed` при неизменном списке и не перестраивает `#queue-body`, пока drawer закрыт.
  - В `CacheManager` добавлен O(1)-индекс в RAM (`_cached_index`, `has_cover()`, `invalidate_index()`).
  - Остановлены холостые `requestAnimationFrame`-циклы в визуализаторе спектра и UI-мониторе производительности при паузе, скрытом окне и в режиме энергосбережения; добавлен `contain: layout style paint` на строки треков.

### Fixed
- Исправлены зависания при клике на онлайн-трек, добавлены спиннер загрузки (`loading_state`), вывод ошибок воспроизведения (`player_error`), токен отмены гонок `_play_req_id` и однократный авто-ретрай протухших потоков.
- Исправлен запуск неверного трека при клике на локальные/кешированные результаты поиска (`source: 'search_local'` + защита `seq`).
- Исправлено чтение белого текста на белом фоне в выпадающих списках `<select>`/`<option>` на WebKitGTK и безопасно экранированы селекторы через `CSS.escape()`.
- Исправлен вызов `cm.cache_track(...)` при синхронном кешировании плейлиста.

---

## [0.3.0] - 2026-11-15

### Added
- **Видеоплеер Rclone и локальный видеокеш**: добавлен модуль `rclone_video_manager.py`, вкладка «Видео (Rclone)», импорт токенов `rclone.conf`, режим PiP и совместимости видео.
- **Центр аккаунта и мультиаккаунт Google**: добавлена вкладка `#view-account`, переключатель профилей `AuthUser`, исправлено определение реального активного аккаунта Chrome вместо постороннего профиля.
- **Нативный OAuth и плеер в Android (`MainActivity.kt`)**: полноценная интеграция входа в Google, синхронизация плейлистов и адаптивный режим интерфейса `uiAdaptiveMode` + виртуализация длинных списков.

---

## [0.2.0] - 2026-11-12

### Added
- **100% миграция core-логики в Rust**: операции очереди (`queue`), LRU-политика кеша (`cache_policy`) и классификация ошибок (`error_policy`) переведены на единое Rust-ядро (`shared-core/rust-core`) с тестами паритета.
- **Кроссплатформенные клиенты (MVP)**: добавлен проект для Windows (`WinUI 3 / WebView2`), Android-приложение с JNI-биндингом Rust (`platform/android`), мобильная адаптация UI `.platform-android` и GitHub Actions CI для `.deb`, `.exe`, `.apk`.

---

## [0.1.0] - 2026-11-11

### Added
- **Ребрендинг Novus Media и новая архитектура v2 (`ym_app_v2.py`)**: современный интерфейс на WebKitGTK + GStreamer, HTML/CSS/JS фронтенд в стиле Obsidian/Snow Glass.
- **Защищённый офлайн-кеш на HDD (`cache_manager.py`)**: шифрованные контейнеры `.ymc` / `.ymcc` (AES-GCM), квота с LRU-очисткой для защиты системного NVMe SSD, автоматическая дедупликация с `~/Музыка`.
- **Инфраструктура плеера**: 4-уровневое синхронизированное караоке (YouTube Music, LRCLIB, локальные `.lrc`), плавающий мини-плеер, MPRIS 2 D-Bus и системные медиа-клавиши, сборка пакета `novus-media_*.deb`.

---

## [0.0.0] - 2026-11-09

### Added
- **Инициализация проекта (`YouTube Music Ultimate`)**: базовая версия на Python / GTK (`ym_gui.py`), прямое воспроизведение, загрузка и сжатие в Opus (`download_ym.sh`), синхронизация плейлистов, таймер сна, нормализация громкости ReplayGain и экспорт `.m3u8`.
