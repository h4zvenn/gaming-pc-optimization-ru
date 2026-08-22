# 09. PowerShell: проверка и диагностика

> Этот раздел содержит преимущественно безопасные команды для просмотра информации о ПК, процессах, питании, дисках и системных событиях. Они не «разгоняют» компьютер и не отключают компоненты Windows — они помогают понять, где находится проблема.

## Как открыть PowerShell

### Обычный режим

1. Нажмите `Win`.
2. Введите `Windows Terminal` или `PowerShell`.
3. Откройте приложение.

Для большинства команд в этом гайде права администратора не нужны.

### От имени администратора

Права администратора нужны только там, где это прямо указано.

1. Нажмите `Win`.
2. Введите `Windows Terminal`.
3. Нажмите правой кнопкой → `Запуск от имени администратора`.

> [!WARNING]
> Не запускайте неизвестные команды от администратора. Администраторские права позволяют изменять системные настройки, службы, реестр и файлы Windows.

---

## 1. Информация о процессоре

```powershell
Get-CimInstance Win32_Processor |
Select-Object Name, Manufacturer, NumberOfCores, NumberOfLogicalProcessors,
CurrentClockSpeed, MaxClockSpeed
```

### Что показывает

- `Name` — модель CPU
- `Manufacturer` — производитель
- `NumberOfCores` — физические ядра
- `NumberOfLogicalProcessors` — логические потоки
- `CurrentClockSpeed` — значение частоты, которое сообщает Windows
- `MaxClockSpeed` — указанная максимальная частота

> [!NOTE]
> `CurrentClockSpeed` и `MaxClockSpeed` из WMI не всегда показывают точный boost под игровой нагрузкой. Для подробного мониторинга частот, температур и лимитов используйте HWiNFO.

---

## 2. Информация о видеокарте и драйвере

```powershell
Get-CimInstance Win32_VideoController |
Select-Object Name, VideoProcessor, DriverVersion, DriverDate,
CurrentHorizontalResolution, CurrentVerticalResolution, CurrentRefreshRate
```

### Что показывает

- модель GPU;
- версию и дату драйвера;
- текущее разрешение;
- частоту обновления дисплея.

> [!TIP]
> Если `CurrentRefreshRate` показывает неожиданный результат, дополнительно проверьте: `Параметры → Система → Дисплей → Дополнительный дисплей`.

---

## 3. Информация об оперативной памяти

```powershell
Get-CimInstance Win32_PhysicalMemory |
Select-Object Manufacturer, PartNumber, Capacity, Speed, ConfiguredClockSpeed
```

### Что смотреть

- `Capacity` — объём одного модуля в байтах;
- `Speed` — заявленная скорость, которую сообщает модуль;
- `ConfiguredClockSpeed` — частота, на которой память настроена сейчас.

### Посчитать общий объём RAM в GB

```powershell
[math]::Round(
    (Get-CimInstance Win32_PhysicalMemory |
    Measure-Object -Property Capacity -Sum).Sum / 1GB,
    0
)
```

> [!NOTE]
> Если частота ниже ожидаемой, проверьте XMP/EXPO в BIOS. Не включайте профиль, если система с ним нестабильна.

---

## 4. Диски и свободное место

### Свободное место на дисках

```powershell
Get-PSDrive -PSProvider FileSystem |
Select-Object Name,
@{Name='UsedGB';Expression={[math]::Round($_.Used / 1GB, 1)}},
@{Name='FreeGB';Expression={[math]::Round($_.Free / 1GB, 1)}}
```

### Физические накопители

```powershell
Get-PhysicalDisk |
Select-Object FriendlyName, MediaType, BusType, HealthStatus,
OperationalStatus, Size
```

### Что смотреть

- `MediaType` — SSD или HDD, если Windows определила тип;
- `BusType` — NVMe, SATA, USB и т.д.;
- `HealthStatus` — общий статус, который сообщает Windows;
- `FreeGB` — свободное место на разделе.

> [!TIP]
> Статус `Healthy` не заменяет полный SMART-анализ. Если есть подозрение на проблему SSD, используйте утилиту производителя накопителя.

---

## 5. Активный план питания

### Текущая схема

```powershell
powercfg /getactivescheme
```

### Все доступные схемы

```powershell
powercfg /list
```

### Отчёт об энергопотреблении

Откройте PowerShell **от имени администратора**:

```powershell
powercfg /energy
```

Windows создаст HTML-отчёт и выведет путь к нему.

> [!NOTE]
> Не исправляйте все предупреждения из `powercfg /energy` автоматически. Часть из них нормальна для современного ПК, USB-устройств и энергосберегающих режимов. Смотрите на конкретную проблему.

---

## 6. Автозагрузка

```powershell
Get-CimInstance Win32_StartupCommand |
Select-Object Name, Command, Location, User
```

Команда выводит приложения, зарегистрированные для запуска при входе в Windows.

### Как отключать правильно

Не удаляйте запись через PowerShell или реестр, если можно сделать это проще:

```text
Диспетчер задач → Автозагрузка приложений → Отключить
```

Подробнее: [Фоновые приложения и автозагрузка](07-background-apps.md).

---

## 7. Процессы

### Процессы с наибольшим накопленным временем CPU

```powershell
Get-Process |
Sort-Object CPU -Descending |
Select-Object -First 20 Name, Id, CPU, WorkingSet64
```

### Процессы с наибольшим использованием RAM

```powershell
Get-Process |
Sort-Object WorkingSet64 -Descending |
Select-Object -First 20 Name, Id,
@{Name='RAM_MB';Expression={[math]::Round($_.WorkingSet64 / 1MB, 0)}}
```

### Найти процесс по имени

Пример: поиск процессов Discord.

```powershell
Get-Process *discord* -ErrorAction SilentlyContinue |
Select-Object Name, Id, CPU, WorkingSet64
```

> [!WARNING]
> Не используйте `Stop-Process` для системных или неизвестных процессов. Сначала выясните, какой программе принадлежит процесс, через Диспетчер задач и расположение файла.

---

## 8. Службы

### Запущенные службы

```powershell
Get-Service |
Where-Object Status -eq 'Running' |
Sort-Object DisplayName |
Select-Object Status, Name, DisplayName
```

### Службы с автоматическим запуском

```powershell
Get-CimInstance Win32_Service |
Where-Object StartMode -eq 'Auto' |
Sort-Object DisplayName |
Select-Object Name, DisplayName, State, StartMode
```

### Поиск службы

Пример: поиск служб, где есть слово `audio`.

```powershell
Get-Service *audio* |
Select-Object Status, Name, DisplayName
```

Команды показывают информацию, но не меняют конфигурацию служб.

Подробнее: [Службы и безопасность](08-services-and-security.md).

---

## 9. Сеть

### Информация об IP-адресах и адаптерах

```powershell
Get-NetIPConfiguration
```

### Состояние сетевых адаптеров

```powershell
Get-NetAdapter |
Select-Object Name, InterfaceDescription, Status, LinkSpeed, MacAddress
```

### Проверка связи с DNS

```powershell
Resolve-DnsName microsoft.com
```

### Проверка задержки до адреса

```powershell
Test-Connection 1.1.1.1 -Count 10
```

### Что смотреть

- потерю пакетов;
- нестабильные значения задержки;
- отключённый адаптер;
- LinkSpeed, который явно ниже возможностей подключения.

> [!NOTE]
> Тест до `1.1.1.1` не измеряет ping до игрового сервера. Он лишь показывает базовую связь с указанным адресом. Для сетевых проблем также проверяйте Wi-Fi, Ethernet-кабель, роутер, загрузки и провайдера.

---

## 10. Проверка системных файлов

Используйте только при подозрении на повреждение Windows: ошибки обновления, системные вылеты, проблемы с компонентами ОС или повреждённые файлы.

Откройте PowerShell **от имени администратора**.

### Проверка системных файлов

```powershell
sfc /scannow
```

### Восстановление образа Windows

```powershell
DISM /Online /Cleanup-Image /RestoreHealth
```

### Правильный порядок

1. Запустите `sfc /scannow`.
2. Если проблема не исчезла или SFC не смог исправить файлы — запустите DISM.
3. После завершения DISM перезагрузите ПК.
4. При необходимости снова запустите SFC.

> [!IMPORTANT]
> SFC и DISM — средства восстановления Windows, а не «ежедневный FPS-твик». Не запускайте их постоянно без причины.

---

## 11. Журнал событий и WHEA

WHEA-ошибки могут говорить о нестабильности CPU, RAM, PBO, Curve Optimizer, undervolt, PCIe или другого оборудования.

### Найти WHEA-события через PowerShell

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'System'
    ProviderName = 'Microsoft-Windows-WHEA-Logger'
} -MaxEvents 20 |
Select-Object TimeCreated, Id, LevelDisplayName, Message
```

### Что делать при WHEA

1. Не игнорируйте повторяющиеся ошибки.
2. Верните разгон, undervolt, XMP/EXPO или PBO/Curve Optimizer к стабильным значениям.
3. Проверьте температуры и питание.
4. Повторите тест.
5. Если ошибки остаются на заводских настройках — нужна более глубокая диагностика железа.

> [!WARNING]
> Даже если игра не вылетела, WHEA после разгона или Curve Optimizer — повод ослабить настройку.

---

## 12. Отчёт о системе

### Сохранить общую информацию в текстовый файл

```powershell
systeminfo | Out-File -Encoding utf8 "$env:USERPROFILE\Desktop\systeminfo.txt"
```

Команда создаст файл `systeminfo.txt` на рабочем столе текущего пользователя.

### Экспорт информации о драйверах

```powershell
driverquery /v /fo csv | Out-File -Encoding utf8 "$env:USERPROFILE\Desktop\drivers.csv"
```

> [!NOTE]
> Файл с драйверами может содержать много строк. Он нужен для диагностики, а не как список драйверов, которые обязательно нужно обновить.

---

## 13. Команды, которых здесь нет специально

Этот гайд не предлагает команды для:

- массового отключения служб;
- отключения Defender, Firewall, UAC и Windows Update;
- удаления системных компонентов;
- агрессивного изменения BCD/HPET/таймеров;
- отключения файла подкачки;
- «очистки RAM»;
- изменения скрытых параметров реестра без конкретной диагностики.

Причина простая: такие действия могут навредить стабильности, безопасности и совместимости, а универсального подтверждённого прироста FPS не дают.

---

## Чек-лист

- [ ] Команды запускаются только с понятной целью.
- [ ] Для обычной диагностики не используются права администратора без необходимости.
- [ ] Результаты команд интерпретируются вместе с мониторингом и тестами в игре.
- [ ] Не завершаются неизвестные процессы.
- [ ] Не отключаются службы через команды «по списку».
- [ ] SFC/DISM запускаются только при подозрении на повреждение Windows.
- [ ] WHEA проверяются после разгона или нестабильности.

---

## Итог

PowerShell полезен для диагностики и контроля, а не для «однокнопочной оптимизации». С его помощью можно быстро проверить компоненты, драйверы, память, диски, питание, автозагрузку, процессы, сеть и системные ошибки — после чего исправлять конкретную найденную проблему.

Следующий шаг: [Температуры, стабильность и обслуживание](10-temperatures-stability.md).
