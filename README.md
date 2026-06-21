# 🔍 Windows Driver Disassembly — IDA Free

<div align="center">

![IDA Pro](https://img.shields.io/badge/IDA-Free%207.0-blue?style=for-the-badge&logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI+PHBhdGggZmlsbD0id2hpdGUiIGQ9Ik0xMiAyQzYuNDggMiAyIDYuNDggMiAxMnM0LjQ4IDEwIDEwIDEwIDEwLTQuNDggMTAtMTBTMTcuNTIgMiAxMiAyem0xIDE1aC0ydi02aDJ2NnptMC04aC0yVjdoMnYyeiIvPjwvc3ZnPg==)
![Windows Driver](https://img.shields.io/badge/Windows-Kernel%20Driver-0078D6?style=for-the-badge&logo=windows)
![Assembly](https://img.shields.io/badge/Language-x86%20Assembly-green?style=for-the-badge)
![University](https://img.shields.io/badge/ДВФУ-Лаб%20%2313-red?style=for-the-badge)

**Дизассемблирование и анализ Windows kernel-mode драйвера фильтра мыши с помощью IDA Free**

**Disassembly and analysis of a Windows kernel-mode mouse filter driver using IDA Free**

</div>

---

## 🇷🇺 Русский

### 📋 О проекте

Лабораторная работа №13 по дисциплине **«Вычислительные системы, сети и низкоуровневое программирование»**, ДВФУ-FEFU, Институт математики и компьютерных технологий.

**Объект исследования:** `mafmouse.sys` — Windows kernel-mode драйвер-фильтр мыши (MAF-Soft filtered Mouse Device), который инвертирует ось Y мыши.

### 🎯 Цель работы

- Изучить программное обеспечение IDA Free
- Изучить основную структуру драйверов Windows (kernel-mode)
- Дизассемблировать драйвер `mafmouse.sys`
- Привести код в читабельный вид: присвоить смысловые имена автоматически сгенерированным переменным, функциям и смещениям структур

### 🛠️ Инструменты

| Инструмент | Версия | Назначение |
|---|---|---|
| IDA Free | 7.0 | Дизассемблер и отладчик |
| mafmouse.sys | — | Исследуемый драйвер |
| mafmouse.idc | — | IDC-скрипт для загрузки структур |

### 📁 Структура репозитория

```
windows-driver-disassembly/
├── driver/                  # Бинарные файлы драйверов
│   ├── mafmouse.sys         # Основной драйвер-фильтр мыши
│   ├── drvmouse.sys         # Вспомогательный драйвер мыши
│   ├── mafmouse_ps2.inf     # INF-файл для PS/2 мыши
│   └── mafmouse_usb.inf     # INF-файл для USB мыши
├── scripts/                 # IDC-скрипты для IDA
│   ├── mafmouse.idc         # Скрипт для mafmouse.sys
│   └── drvmouse.idc         # Скрипт для drvmouse.sys
├── references/              # Справочные материалы
│   └── Irp Major Functions Code.png  # Таблица IRP major functions
├── docs/                    # Документация
│   └── mafmouse.txt         # Описание драйвера от разработчика
└── README.md
```

### 🔬 Что было сделано

#### 1. Загрузка драйвера в IDA
- Открыт `mafmouse.sys` как **Portable Executable for 80386 (PE)**
- Загружены IDC-скрипты (`mafmouse.idc`, `drvmouse.idc`) для импорта стандартных структур WDM/NDDK

#### 2. Анализ точки входа
- Найдена функция `start` (точка входа компилятора)
- Через `jmp sub_15006` обнаружен переход на `DriverEntry`
- Функция переименована в `DriverEntry_main`
- Аргумент `arg_0` переименован в `DriverObject`

#### 3. Переименование Dispatch-процедур

В `DriverEntry_main` были найдены и переименованы все обработчики IRP:

| Адрес | Старое имя | Новое имя | Смещение в _DRIVER_OBJECT |
|---|---|---|---|
| `sub_11026` | `sub_11026` | `DispatchPassThrough` | — |
| `sub_14202` | `sub_14202` | `DispatchInternalDeviceControl` | `[edx+IRP_MJ_INTERNAL_DEVICE_CONTROL]` |
| `sub_14098` | `sub_14098` | `DispatchPnp` | `[edx+0A4h]` |
| `sub_141B0` | `sub_141B0` | `DispatchPower` | `[edx+90h]` |
| `sub_1425C` | `sub_1425C` | `DispatchCreateClose` | `[edx+74h]` |
| `sub_14006` | `sub_14006` | `AddDevice` | `[eax+4]` |
| `nullsub_1` | `nullsub_1` | `DispatchUnload` | `[edx+34h]` |

#### 4. Переименование аргументов
В каждой Dispatch-процедуре переименованы аргументы:
- `arg_0` → `pDeviceObject`
- `arg_4` → `Irp`

#### 5. Анализ DispatchPassThrough
Процедура `DispatchPassThrough` просто передаёт IRP вниз по стеку устройств через вызов `IofCallDriver` — стандартная реализация pass-through драйвера-фильтра.

### 📖 Структура Windows Kernel Driver

```
DriverEntry()
├── Инициализация DriverObject
├── Регистрация Dispatch-обработчиков:
│   ├── IRP_MJ_CREATE / IRP_MJ_CLOSE  → DispatchCreateClose
│   ├── IRP_MJ_INTERNAL_DEVICE_CONTROL → DispatchInternalDeviceControl
│   ├── IRP_MJ_PNP                    → DispatchPnp
│   ├── IRP_MJ_POWER                  → DispatchPower
│   └── (все остальные)               → DispatchPassThrough
└── AddDevice() → IoCreateDevice()
```

---

## 🇬🇧 English

### 📋 About

Lab work #13 for the course **"Computer Systems, Networks and Low-Level Programming"** at Far Eastern Federal University (FEFU), Institute of Mathematics and Computer Technologies.

**Target:** `mafmouse.sys` — a Windows kernel-mode mouse filter driver (MAF-Soft filtered Mouse Device) that inverts the Y-axis of the mouse.

### 🎯 Goals

- Study IDA Free disassembler
- Understand the structure of Windows kernel-mode drivers
- Disassemble `mafmouse.sys`
- Reverse engineer the code: rename auto-generated variables, functions, and structure offsets to meaningful names

### 🔬 What Was Done

#### Key findings:
- Identified the `DriverEntry` entry point (via `jmp` from compiler stub `start`)
- Mapped all IRP dispatch handlers registered in `DriverObject`
- Renamed all major dispatch routines and their arguments (`pDeviceObject`, `Irp`)
- Identified `DispatchPassThrough` as a simple IRP forwarding function using `IofCallDriver`
- Identified `AddDevice` calling `IoCreateDevice` to create the filter device object

### 📚 References

- [IRP Major Function Codes — Microsoft Docs](https://learn.microsoft.com/en-us/windows-hardware/drivers/kernel/irp-major-function-codes)
- [Writing a DriverEntry Routine](https://learn.microsoft.com/en-us/windows-hardware/drivers/kernel/writing-a-driverentry-routine)
- [Filter Drivers — Windows Driver Kit](https://learn.microsoft.com/en-us/windows-hardware/drivers/kernel/filter-drivers)
- IDA Free 7.0 — [hex-rays.com](https://hex-rays.com/ida-free/)

---

<div align="center">

**ДВФУ / FEFU · 2026**

*Дальневосточный федеральный университет*  
*Институт математики и компьютерных технологий*  
*Департамент программной инженерии и искусственного интеллекта*

</div>
