# yaAlphaEmu - AlphaEmu, эмулятор AlphaServer ES40 / AlphaServer ES40 emulator

*Русский текст - первым, English below.*

## Русский

### Что это

AlphaEmu - системный эмулятор AlphaServer ES40 (процессор Alpha EV6/21264,
чипсет Tsunami) для Linux. На нём работает OpenVMS Alpha V8.3. Консоль
`P00>>>` и PALcode - собственные, прошивка DEC (SRM) не нужна и не
поставляется. Автор: StarLet Squad и Ruslan R. Laishev.

Это репозиторий только с готовыми двоичными файлами; исходных текстов здесь
нет. Freeware для некоммерческого использования; коммерческая лицензия - отдельно (см. "Лицензия").

Пример вывода `--version` (сборка для x86-64):

```
AlphaEmu 1.00.45 (X01-00ECO45), DEC Alpha EV6 / AlphaServer ES40 emulator
Built for x86-64, guest code translator yes, DEC SRM console not included
53C810 SCSI adapter (tape, scsi.disk*) not included
```

Что моделируется:

* 1-4 процессора (vCPU), память гостя до 16 ГБ (`memory.bits` 26..34);
* диски на IDE (`DQA0:`, `DQB0:`) и на SCSI-адаптере QLogic ISP1040
  (`DKA0:`), образы - файлы хоста; образ компакт-диска OpenVMS - как IDE-диск;
* сеть: DE500-BA (до четырёх адаптеров, `EWA0:`-`EWD0:`), Intel 82540EM
  (`EIA0:`, jumbo-кадры), DEGXA-TA (Broadcom 5703); все - на TAP-устройствах
  хоста;
* последовательная консоль (`OPA0:`) на терминале или TCP/UNIX-сокете;
* контрольные точки (checkpoint) состояния;
* загрузка по сети (MOP) с сервера загрузки OpenVMS или InfoServer;
  кластер VMScluster из нескольких эмуляторов.

### Состав архива выпуска

Для каждой архитектуры - архив `alphaemu-<ver>-<arch>.tar.gz` и файл
`alphaemu-<ver>-<arch>.tar.gz.sha256`. Архитектуры:

* `x86_64` - статический двоичный файл, с транслятором гостевого кода (JIT);
* `aarch64` - статический двоичный файл, только интерпретатор, без JIT
  (заметно медленнее).

В архиве:

| Файл | Что это |
|---|---|
| `alphaemu` | эмулятор |
| `samples/alphaemu-native.cfg` | образец файла параметров - начните с него |
| `samples/` | другие образцы (DCL-процедуры, скрипты) |
| `alphaemu_users_guide.pdf` | руководство пользователя: устройства, параметры, консоль, эксплуатация |
| `README.txt` | краткое описание |
| `EULA.txt` | лицензионное соглашение |
| `NOTICES.txt` | компоненты сторонних авторов, товарные знаки |
| `LICENSE-Apache-2.0.txt` | лицензия Apache 2.0 (StarLet и `samples/`) |
| `LICENSE-musl.txt` | лицензия библиотеки musl |
| `SHA256SUMS` | контрольные суммы файлов комплекта |

### Требования к хосту

* 64-битный Linux: x86-64 с инструкцией CMPXCHG16B (JIT есть только здесь)
  или aarch64 (только интерпретатор);
* память хоста не меньше памяти гостя плюс запас; одно ядро хоста на один vCPU;
* для сети - TAP-устройства (нужны права root); для `mem.lock` и
  `host.rt_prio` - право блокировать память и использовать приоритеты
  реального времени;
* ничего из прошивки DEC.

### Быстрый старт

```
sha256sum -c alphaemu-<ver>-<arch>.tar.gz.sha256
tar xzf alphaemu-<ver>-<arch>.tar.gz
cd alphaemu-<ver>-<arch>
cp samples/alphaemu-native.cfg /полный/путь/alphaemu.cfg
# отредактируйте его: пути к образам дисков, TAP
./alphaemu /полный/путь/alphaemu.cfg
```

У приглашения `P00>>>`:

```
P00>>> boot dka0
```

Фрагмент образца файла параметров:

```
memory.bits      = 30                       # 1 ГБ
cpu.count        = 2
smp.threads      = 1
isp.disk0        = /srv/alphaemu/vmssys.img # системный диск, DKA0:
# disk.dqa0      = /srv/alphaemu/AXPVMS83.raw  # образ CD, "boot dqa0"
# net.tap        = tap0                     # DE500-BA, EWA0:
console.nvram    = /srv/alphaemu/console.nvram
jit.enable       = 1                        # только x86-64
```

### Чего в комплекте нет

Нужны ваши собственные носители OpenVMS Alpha V8.3 и лицензии (PAK) на
OpenVMS и на слойные продукты. Ничего из этого не поставляется.

### Ограничения

* В этот выпуск не входят адаптер SCSI 53C810 (поэтому нет ленты/BACKUP на
  ленту) и консоль DEC SRM.
* Проверена только OpenVMS Alpha V8.3; другие системы не проверялись.
* Нет видеокарты, клавиатуры и мыши, дискеты, USB, ATAPI CD-ROM; DECwindows
  работает на X-сервере хоста.
* Не более четырёх процессоров; контрольная точка восстанавливается только
  той же сборкой эмулятора с тем же числом CPU и объёмом памяти и не содержит
  данных дисков.
* aarch64: только интерпретатор, скорость много ниже, чем на x86-64.
* aarch64: SMP (`cpu.count` больше 1) - экспериментальный; для работы берите 1 CPU.
* Собственная консоль умеет загрузку, SET/SHOW переменных и список устройств;
  диагностики и обновления прошивки в ней нет.

### Сообщения об ошибках

Через Issues этого репозитория на Codeberg
(https://codeberg.org/SysManOne/yaAlphaEmu). Укажите версию (`--version`),
архитектуру, файл параметров и фрагмент журнала.

### Лицензия

Бесплатно для некоммерческого использования (личное и учебное, некоммерческие
исследования, оценка перед покупкой). Для коммерческого использования нужна
отдельная лицензия автора. Условия - в `EULA.txt`, сторонние компоненты - в
`NOTICES.txt`. Исходные тексты не распространяются. Включать AlphaEmu в
коммерческий продукт или услугу можно только с письменного разрешения автора.
Вопросы о лицензии и коммерческая лицензия: zator@yandex.ru.

### Товарные знаки

DEC, Alpha, AlphaServer, OpenVMS и другие названия - товарные знаки их
владельцев; см. `NOTICES.txt`. AlphaEmu не связан с их владельцами.

### Сопутствующие проекты / Companion projects

| Проект | Что это | Состояние | URL |
|---|---|---|---|
| yaVTERM | шифрованный терминал и передача файлов для OpenVMS | готов | https://gitlab.com/SysMan-One/yavterm |
| yaDCX | сборщик комплектов (kits) для OpenVMS, выпуск X-02-00ECO01 | готов | https://codeberg.org/SysManOne/yaDCX |
| yaDX | DX, полноэкранный файловый менеджер для OpenVMS; комплекты в выпусках | готов | https://codeberg.org/SysManOne/yaDX |

---

## English

### What it is

AlphaEmu is a system emulator of the AlphaServer ES40 (Alpha EV6/21264
processor, Tsunami chipset) for Linux. It runs OpenVMS Alpha V8.3. The
`P00>>>` console and the PALcode are its own: no DEC firmware (SRM) is needed
or shipped. By StarLet Squad and Ruslan R. Laishev.

This repository holds binaries only; there are no sources here. The program
is freeware for non-commercial use; commercial licence separately (see "Licence").

Sample `--version` output (x86-64 build):

```
AlphaEmu 1.00.45 (X01-00ECO45), DEC Alpha EV6 / AlphaServer ES40 emulator
Built for x86-64, guest code translator yes, DEC SRM console not included
53C810 SCSI adapter (tape, scsi.disk*) not included
```

What is modelled:

* 1 to 4 processors (vCPUs), guest memory up to 16 GB (`memory.bits`
  26..34);
* disks on IDE (`DQA0:`, `DQB0:`) and on the QLogic ISP1040 SCSI adapter
  (`DKA0:`); the images are host files; the OpenVMS CD image is used as an
  IDE disk;
* network: DE500-BA (up to four adapters, `EWA0:` to `EWD0:`), Intel 82540EM
  (`EIA0:`, jumbo frames), DEGXA-TA (Broadcom 5703); all on host TAP devices;
* a serial console (`OPA0:`) on the terminal or on a TCP/UNIX socket;
* checkpoints of the machine state;
* network boot (MOP) from an OpenVMS boot server or an InfoServer; a
  VMScluster of several emulators.

### What is in a release archive

For each architecture there is an archive `alphaemu-<ver>-<arch>.tar.gz` and
a file `alphaemu-<ver>-<arch>.tar.gz.sha256`. The architectures:

* `x86_64` - a static binary with the guest code translator (JIT);
* `aarch64` - a static binary, interpreter only, no JIT (much slower).

Inside the archive:

| File | What it is |
|---|---|
| `alphaemu` | the emulator |
| `samples/alphaemu-native.cfg` | a sample parameter file - start from it |
| `samples/` | other samples (DCL procedures, scripts) |
| `alphaemu_users_guide.pdf` | the User's Guide: devices, parameters, console, operation |
| `README.txt` | a short description |
| `EULA.txt` | the End User License Agreement |
| `NOTICES.txt` | third-party components, trademarks |
| `LICENSE-Apache-2.0.txt` | the Apache 2.0 licence (StarLet and `samples/`) |
| `LICENSE-musl.txt` | the licence of the musl library |
| `SHA256SUMS` | checksums of the kit's files |

### Host requirements

* 64-bit Linux: x86-64 with the CMPXCHG16B instruction (the JIT is only
  there) or aarch64 (interpreter only);
* host memory of at least the guest memory plus some reserve; one host core
  per vCPU;
* for networking, TAP devices (root is needed); for `mem.lock` and
  `host.rt_prio`, the right to lock memory and to use real-time priorities;
* no DEC firmware.

### Quick start

```
sha256sum -c alphaemu-<ver>-<arch>.tar.gz.sha256
tar xzf alphaemu-<ver>-<arch>.tar.gz
cd alphaemu-<ver>-<arch>
cp samples/alphaemu-native.cfg /full/path/alphaemu.cfg
# edit it: disk image paths, TAP
./alphaemu /full/path/alphaemu.cfg
```

At the `P00>>>` prompt:

```
P00>>> boot dka0
```

An excerpt of the sample parameter file is shown in the Russian part above.

### What you must bring

Your own OpenVMS Alpha V8.3 media and licences (PAKs) for OpenVMS and the
layered products you use. None of it is provided.

### Limitations

* Not in this release: the 53C810 SCSI adapter (so no tape or BACKUP to
  tape) and the DEC SRM console.
* Only OpenVMS Alpha V8.3 is verified; other systems are not.
* No graphics card, keyboard, mouse, floppy, USB or ATAPI CD-ROM; DECwindows
  runs on an X server of the host.
* At most four CPUs; a checkpoint is restored only by the same build with the
  same CPU count and memory, and holds no disk contents.
* aarch64: interpreter only, much slower than x86-64.
* aarch64: SMP (`cpu.count` above 1) is experimental; use 1 CPU for real work.
* The own console boots, sets and shows its variables and lists devices; it
  has no diagnostics and no firmware update.

### Bug reports

Use the Issues of this repository on Codeberg
(https://codeberg.org/SysManOne/yaAlphaEmu). Give the version (`--version`),
the architecture, your parameter file and a piece of the log.

### Licence

Free for non-commercial use (private and hobby use, teaching and learning,
non-profit research, evaluation before a commercial licence). Commercial use
needs a separate licence from the author. The terms are in `EULA.txt`;
third-party components are in `NOTICES.txt`. Sources are not distributed.
Bundling AlphaEmu in a commercial product or service needs the author's
written permission. Licence questions and commercial licences: zator@yandex.ru.

### Trademarks

DEC, Alpha, AlphaServer, OpenVMS and other names are trademarks of their
owners; see `NOTICES.txt`. AlphaEmu is not affiliated with them.

### Companion projects

See the table "Сопутствующие проекты / Companion projects" above: yaVTERM
(encrypted terminal and file transfer for OpenVMS) - ready; yaDCX (OpenVMS
kit builder, X-02-00ECO01) - ready; yaDX (the DX file manager for OpenVMS and
its kits) - ready.
