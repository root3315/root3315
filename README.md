# Stepan Ionichev

**Linux kernel contributor · Backend & systems developer · Founder / CTO @ [Bazier.live](https://bazier.live)**

Драйверы и ядро, distributed systems, security, embedded, aerospace.
C · Python · TypeScript · Go

[Email](mailto:sozdayvek@gmail.com) · [Telegram](https://t.me/Pumpkin2008) 

---

## Linux kernel (mainline)

**34 коммита в дереве Линуса, 13 подсистем, май–июль 2026.**
Каждый прошёл ревью мейнтейнера подсистемы и публично проверяем:

→ [**Все коммиты в torvalds/linux**](https://github.com/torvalds/linux/commits?author=root3315) · [lore.kernel.org](https://lore.kernel.org/all/?q=f%3AIonichev)

Патчи приняты Greg Kroah-Hartman (serial, USB, driver core), Jonathan Cameron (IIO),
Andrew Morton (lib), Bartosz Golaszewski (GPIO), Andy Shevchenko (pinctrl, auxdisplay),
Alexandre Belloni (RTC), William Breathitt Gray (counter), Lee Jones (LEDs).

**Специализация:** memory-safety и корректность путей ошибок в драйверах —
out-of-bounds, use-after-free, NULL deref, утечки ссылок, необработанные сбои в `probe()`
и в обработчиках прерываний.

### Ключевой фикс

[**`auxdisplay: line-display: fix OOB read on zero-length message_store()`**](https://git.kernel.org/torvalds/c/a7511dcd9dd4) — `a7511dcd9dd4`, backport в stable 7.0.12

Запись нулевой длины в sysfs-атрибут `message` уводила чтение за границу буфера.
На ядре с KASAN — отчёт об out-of-bounds и паника; на стоковом — тихое чтение соседних
данных slab, и если байт оказывался `'\n'`, последующий `count--` заворачивал `ssize_t`
из 0 в `-1`, после чего значение уходило в `kmemdup_nul()`.
Триггерится из userspace обычной нулевой записью: `vfs_write()` не отсекает `count == 0`,
а `kernfs_fop_write_iter()` всё равно вызывает store-колбэк.
Затрагивал все драйверы, регистрирующиеся через `linedisp_register()`:
`ht16k33`, `max6959`, `img-ascii-lcd`, `seg-led-gpio`.

### Исправления багов

| Подсистема | Патч |
|---|---|
| `serial` | `8250_dw: unregister 8250 port if clk_notifier_register() fails` — use-after-free, backport в stable 7.0.14 |
| `serial` | `8250_dw: remove clock-notifier infrastructure` |
| `tty` | `serial: 8250: protect against NULL uart->port.dev in register` |
| `driver core` | `device property: fix fwnode reference leak in fwnode_graph_get_endpoint_by_id()` |
| `rtc` | `msc313: fix NULL deref in shared IRQ handler at probe` |
| `usb` | `gadget: goku_udc: avoid NULL deref of dev->driver in INT_USBRESET log` |
| `gpio` | `pca953x: propagate regulator_enable() error from resume` |
| `pinctrl` | `intel: move PWM base computation past feature check` |
| `leds` | `dac124s085: declare SPI command word as __le16` — endianness |
| `iio` | `light: tsl2591: return actual error from probe IRQ failure` |
| `iio` | `adc: nxp-sar-adc: harden buffer ISR against per-channel read failure` |
| `iio` | `resolver: ad2s1210: notify trigger and clear state on fault read error` |
| `iio` | `proximity: vl53l0x: notify trigger and clear IRQ on error paths` |
| `iio` | `chemical: scd30: reject (response=NULL, size>0) in scd30_i2c_command()` |
| `iio` | `gyro: bmg160: wait full startup time after mode change at probe` |
| `iio` | `gyro: bmg160: bail out when bandwidth/filter is not in table` |
| `iio` | `adc: qcom-spmi-iadc: balance enable_irq_wake() on driver unbind` |
| `iio` | `adc: ad7192: fix GPOCON register access annotation` |

### Тесты, ресурсы, документация

- `lib/uuid_kunit: add tests for the four random UUID/GUID generators` — KUnit-покрытие четырёх генераторов
- `counter: interrupt-cnt` / `counter: ftm-quaddec` — переход на `devm_mutex_init()`
- `iio: temperature: tmp006` — переход на `devm_iio_trigger_register()`
- `iio` × 4 — замена `usleep_range()` на `fsleep()` (`adxl355`, `ad7192`, `adrf6780`, `ad7793`)
- Документация IIO, Kconfig `auxdisplay`, правки комментариев в `xhci`, `i2c/bcm-kona`, `wcn36xx`, `bno055`, `blinkm`

---

## Избранные проекты

### [UniSat](https://github.com/root3315/unisat) — платформа спутникового ПО

Одна кодовая база → 14 форм-факторов: CanSat, CubeSat 1U–12U, HAB, ракета, дрон, ровер.

- Протокольный стек **AX.25 v2.2 + CCSDS Space Packet**, аутентификация команд через HMAC-SHA256
- **FDIR** на 12 классов отказов с persistent-логированием
- 471 тест, покрытие 85%+ по C и Python
- Готовые профили миссий: UzCanSat, NASA CanSat, CubeSat Design, IREC

`C` `Python` `STM32` `FreeRTOS` → [документация](https://root3315.github.io/unisat/)

### [binary-protocol-fuzzer](https://github.com/root3315/binary-protocol-fuzzer) — фаззер бинарных протоколов

Поиск падений и уязвимостей в разборщиках бинарных форматов — тот же класс дефектов,
что и в патчах выше, только со стороны инструмента.

`C++`

### [ExoVision](https://github.com/root3315/TIC-id) — анализ экзопланет по данным NASA

Классификация кандидатов TESS / Kepler с ML-оценкой потенциальной обитаемости.

- Интеграция с NASA API, визуализация кривых блеска
- Опциональный локальный инференс через Ollama
- FastAPI + Motor (async MongoDB), React + TypeScript

`Python` `FastAPI` `MongoDB` `React`

### [Bazier.live](https://bazier.live) — real-time социальная платформа

Роль: **CTO** — архитектура, CI/CD, техническая часть продукта.

- WebSocket-слой на Socket.io, посты и комментарии, поиск и фильтрация
- JWT + OAuth, инструменты модерации
- Оптимизации горячих путей, снизившие latency и нагрузку на инфраструктуру

`Node.js` `Express` `MongoDB` `Socket.io` `AWS`

### [excel-auto-parser](https://github.com/root3315/excel-auto-parser) — zero-config парсер таблиц

Находит таблицы в `.xlsx / .xlsm / .xls / .xlsb / .csv` без указания диапазонов и ключевых слов.

- Экспорт в JSON / JSONL / CSV, встроенный веб-просмотрщик
- Потоковый режим для больших файлов без переполнения памяти

`Python` `pandas` `CLI`

---

## Стек

**Ядро и embedded** — C, драйверы Linux, IIO, GPIO, serial/tty, USB, STM32, FreeRTOS, KUnit, KASAN
**Языки** — C, Python, TypeScript / Node.js, Go, C++
**Бэкенд** — FastAPI, Express, Flask, Socket.io
**Данные** — PostgreSQL, MongoDB, Redis, MySQL
**Инфра** — Docker, Kubernetes, AWS, Nginx, GitHub Actions, Linux
**Безопасность** — фаззинг, анализ бинарных протоколов, memory-safety аудит

---

Открыт к задачам в драйверах, embedded, системном бэкенде и аудите безопасности.
Связь — почтой или в Telegram.
