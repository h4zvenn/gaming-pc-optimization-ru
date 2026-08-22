# Оптимизация Windows 11 и игрового ПК

> Безопасный и практичный гайд по настройке игрового ПК:
> BIOS → драйверы → Windows → GPU → фоновые процессы → диагностика.

[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-Windows%2011-blue)](#)

## Для кого этот гайд

- Для владельцев стационарных ПК и ноутбуков на Windows 11
- Для тех, кто хочет стабильный FPS, хороший 1% low и ровный frametime
- Для тех, кто хочет понять, что делает каждая настройка
- Не для «одной кнопки +500 FPS»

## Начни отсюда

1. [Создай точку восстановления](docs/01-before-you-start.md)
2. [Определи узкое место: CPU, GPU, RAM, SSD или температуры](docs/02-find-the-bottleneck.md)
3. [Настрой BIOS и оперативную память](docs/03-bios-and-hardware.md)
4. [Установи нормальные драйверы](docs/04-drivers-and-updates.md)
5. [Настрой Windows 11](docs/05-windows-11.md)

## Полный гайд

| Раздел | Что внутри |
|---|---|
| [Подготовка](docs/01-before-you-start.md) | Бэкап, точка восстановления, измерения до твиков |
| [Поиск bottleneck](docs/02-find-the-bottleneck.md) | CPU, GPU, RAM, VRAM, SSD, перегрев |
| [BIOS и железо](docs/03-bios-and-hardware.md) | EXPO/XMP, ReBAR, память, PBO, Curve Optimizer |
| [Драйверы](docs/04-drivers-and-updates.md) | Чипсет, GPU, BIOS, чистая установка |
| [Windows 11](docs/05-windows-11.md) | Game Mode, питание, HAGS, графика |
| [Видеокарта](docs/06-gpu-settings.md) | NVIDIA, AMD, Intel, VRR, апскейлеры |
| [Фоновые процессы](docs/07-background-apps.md) | Автозагрузка, оверлеи, браузер, записи |
| [Службы и безопасность](docs/08-services-and-security.md) | Что нельзя отключать |
| [PowerShell](docs/09-powershell-diagnostics.md) | Команды для диагностики |
| [Стабильность](docs/10-temperatures-stability.md) | Температуры, WHEA, тесты |
| [Опасные твики](docs/11-what-not-to-do.md) | Что не стоит делать |
| [FAQ](docs/12-faq.md) | Частые вопросы |

## Быстрый чек-лист

- [ ] Включён EXPO/XMP, если система стабильна
- [ ] Включены Above 4G Decoding и Resizable BAR
- [ ] Установлены драйверы чипсета и GPU с официальных сайтов
- [ ] Монитор работает на нужной герцовке
- [ ] Включён Game Mode
- [ ] Нет перегрева, троттлинга и WHEA-ошибок
- [ ] Закрыты загрузки, лишние оверлеи и тяжёлые фоновые программы
- [ ] Все изменения измерены и проверены

## Важно

Этот гайд не заменяет диагностику неисправностей. Меняйте один параметр за раз, проверяйте результат и всегда имейте возможность отката.
