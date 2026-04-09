# Сравнение технологий виртуализации

> Обзор охватывает полную виртуализацию, паравиртуализацию, контейнеры, micro-VM и sandbox-рантаймы. Данные актуальны на 2025–2026 г.

---

## Сводная таблица

| Технология | Тип | CPU перф. | I/O перф. | Изоляция | Оверхед RAM | Boot time | Типичный use case |
|---|---|---|---|---|---|---|---|
| **VMware ESXi / vSphere** | Type 1 HV | ~98–99% | Высокая | Полная (VM) | ~256–512 MB base | ~30–60 с | Датацентр, продакшн |
| **Microsoft Hyper-V** | Type 1 HV | ~97–99% | Высокая | Полная (VM) | ~300 MB base | ~20–40 с | Windows Server, dev |
| **KVM + QEMU** | Type 1/2 HV | ~98–99% | Высокая | Полная (VM) | ~50–100 MB base | ~10–30 с | Linux-серверы, homelabs |
| **Xen** | Type 1 HV | ~95–99% | Высокая | Полная (VM) | ~4 MB hypervisor | ~10–20 с | Cloud provider, Qubes OS |
| **VMware Workstation / Fusion** | Type 2 HV | ~90–96% | Средняя–высокая | Полная (VM) | ~512 MB base | ~30–60 с | Dev, тестирование |
| **Oracle VirtualBox** | Type 2 HV | ~85–93% | Ниже средней | Полная (VM) | ~300 MB base | ~30–60 с | Dev, образование |
| **Parallels Desktop** | Type 2 HV | ~95–98% | Высокая | Полная (VM) | ~512 MB base | ~10–20 с | macOS / Apple Silicon |
| **WSL 2** | Lightweight VM | ~98–99% | Средняя* | Частичная | ~250–500 MB | ~1–3 с | Dev-окружение на Windows |
| **QEMU (TCG, без KVM)** | Emulation | ~5–30% | Низкая | Полная | ~100 MB base | ~30–90 с | Cross-arch, embedded |
| **Docker** | Container | ~99–100% | Почти нативная | Namespace/cgroup | ~10–50 MB/ctr | <1 с | Микросервисы, CI/CD |
| **LXC / LXD** | Container | ~99–100% | Нативная | Namespace/cgroup | ~5–30 MB/ctr | <1–2 с | Dev-окружения, homelab |
| **Podman** | Container | ~99–100% | Нативная | Namespace/cgroup | ~5–30 MB/ctr | <1 с | RHEL/Fedora, rootless |
| **containerd / CRI-O** | Container runtime | ~99–100% | Нативная | Namespace/cgroup | Минимальный | <1 с | Kubernetes node runtime |
| **Firecracker** | microVM | ~97–99% | Высокая (virtio) | VM-level (KVM) | ~5 MB base | ~125 мс | Serverless, multi-tenant |
| **gVisor (runsc)** | Sandbox | ~70–90% | Ниже средней | Syscall intercept | ~20–40 MB | <1 с | Ненадёжный код, GKE |
| **Kata Containers** | microVM | ~95–98% | Средняя–высокая | VM-level (KVM) | ~50–100 MB | ~200–500 мс | k8s, regulated workloads |
| **WASM (Wasmtime/WasmEdge)** | WASM runtime | ~90–98% | Средняя | WASM sandbox | <1 MB/instance | <1 мс | Edge, plugins, serverless |

\* WSL 2: внутри ext4 — быстро; cross-mount (Windows ↔ Linux) — медленно.

---

## Важные нюансы

### Производительность CPU

У всех современных Type 1/2 гипервизоров с аппаратной поддержкой (VT-x / AMD-V) CPU-производительность практически нативная (97–99%). Узкое место почти всегда — I/O, латентность памяти и сетевые операции, а не CPU-throughput.

### Изоляция: VM vs. контейнеры

VM-уровень изоляции (VMware, KVM, Hyper-V, Firecracker) принципиально сильнее контейнерной. Контейнеры разделяют ядро с хостом: kernel exploit на хосте автоматически компрометирует все контейнеры. Именно поэтому существуют гибридные решения (Kata Containers, gVisor, Firecracker), предлагающие компромисс «скорость контейнера + изоляция VM».

### I/O подводные камни

- **VirtualBox** сильно проигрывает конкурентам по disk I/O — paravirt storage требует установки Guest Additions.
- **WSL 2**: bind mounts между Windows и Linux (через Plan 9 / virtio-fs) ощутимо медленнее работы внутри ext4.
- **Docker на macOS / Windows**: под капотом всегда Linux VM, поэтому bind mounts медленнее нативных даже с virtiofs.

### Оверхед памяти

Контейнеры и micro-VM выигрывают по плотности: Firecracker запускает тысячи изолированных экземпляров на одной машине с минимальным footprint. Полные VM всегда несут базовый оверхед на гостевое ядро + userspace.

---

## Глоссарий

### Типы гипервизоров

**Type 1 Hypervisor (bare-metal)**
Выполняется непосредственно на железе, без host-OS. Управляет hardware напрямую. Примеры: VMware ESXi, Hyper-V, KVM (технически гибридный — модуль ядра Linux), Xen. Максимальная производительность и изоляция.

**Type 2 Hypervisor (hosted)**
Работает поверх host-OS как обычное приложение. Host-OS обрабатывает все hardware-вызовы. Примеры: VirtualBox, VMware Workstation, Parallels. Удобнее для десктопа, но чуть больше оверхед.

**Paravirtualization (PV)**
Гостевая ОС знает, что работает в виртуальной среде, и использует специальные hypercall-интерфейсы вместо эмуляции реального железа. Примеры: Xen PV guests, virtio-драйверы в KVM. Даёт более высокую производительность I/O по сравнению с полной эмуляцией.

**Full Emulation (TCG)**
QEMU без аппаратной виртуализации: каждая инструкция гостя транслируется в инструкцию хоста программно. Медленно (5–30% нативной скорости), но позволяет эмулировать другую архитектуру (ARM → x86, RISC-V → x86 и т.д.).

---

### Конкретные продукты

**VMware ESXi / vSphere**
Корпоративный Type 1 гипервизор от Broadcom (бывш. VMware). vSphere — надстройка с управлением кластером. Возможности: vMotion (live migration), HA, DRS, vSAN. PVSCSI и VMXNET3 — обязательные паравиртуальные драйверы для приемлемой производительности. Платный, closed source.

**Microsoft Hyper-V**
Type 1 гипервизор, встроенный в Windows Server и Windows 10/11 Pro. Требует SLAT (Second Level Address Translation). Generation 2 VM — UEFI, без legacy BIOS. Основа для WSL 2 и Windows Sandbox.

**KVM (Kernel-based Virtual Machine)**
Модуль Linux-ядра, превращающий Linux в Type 1 гипервизор. Работает совместно с QEMU, который обеспечивает эмуляцию устройств. Поддерживает VFIO (PCIe/GPU passthrough), hugepages, CPU pinning. Де-факто стандарт для Linux-серверов и облаков (OpenStack, AWS Nitro частично).

**Xen**
Один из первых production-grade Type 1 гипервизоров (2003). Архитектура с привилегированным Dom0 и гостевыми DomU. Режимы: HVM (полная виртуализация) и PV (паравиртуализация). Используется в Qubes OS (безопасность через изоляцию) и исторически в AWS EC2.

**VMware Workstation / Fusion**
Type 2 гипервизоры для десктопа. Workstation — Windows/Linux, Fusion — macOS (включая Apple Silicon через Rosetta). Лучшая совместимость с ESXi-форматами (.vmdk, .vmx). Unity mode — запуск гостевых приложений в окнах хоста.

**Oracle VirtualBox**
Бесплатный cross-platform Type 2 гипервизор (Windows, macOS, Linux). Guest Additions критичны для нормальной производительности. Слабее конкурентов по storage I/O и GPU. Nested virtualization — ограниченно.

**Parallels Desktop**
Type 2 гипервизор для macOS. Лучший вариант для Apple Silicon (ARM64 Windows нативно, x86 через Rosetta). Тесная интеграция с macOS. Платный, модель подписки.

**WSL 2 (Windows Subsystem for Linux 2)**
Настоящее Linux-ядро в лёгкой Hyper-V micro-VM. Быстрый старт (~1–3 с), общая память с хостом. Ограничения: медленный cross-filesystem I/O, неполная сетевая изоляция. Поддержка systemd (Windows 11), GPU passthrough (CUDA, DirectML через WSLg).

**Docker**
Платформа для упаковки и запуска приложений в OCI-контейнерах. Использует Linux namespaces + cgroups для изоляции процессов. Контейнеры разделяют ядро хоста — не VM. На Windows/macOS работает через встроенную Linux VM. Экосистема: Docker Hub, Docker Compose, BuildKit, Docker Desktop.

**LXC / LXD**
LXC (Linux Containers) — низкоуровневый интерфейс к namespace/cgroup изоляции. LXD — надстройка с REST API, поддержкой кластеризации, storage pools (ZFS, Btrfs, LVM). «System containers» — полноценная Linux OS с init, cron, syslog внутри контейнера. Ближе к VM по UX, чем Docker, но без изоляции ядра.

**Podman**
OCI-совместимая альтернатива Docker без демона (daemonless). Rootless by default — контейнеры без прав root. Поддерживает Pods (группы контейнеров, как в Kubernetes). Нативная интеграция с systemd. На macOS/Windows требует Podman Machine (встроенная VM).

**containerd / CRI-O**
Low-level OCI container runtimes для Kubernetes. containerd — выделен из Docker, используется по умолчанию в большинстве k8s-дистрибутивов. CRI-O — альтернатива, разработанная Red Hat специально под Kubernetes CRI. Не предназначены для прямого использования пользователем.

**Firecracker**
MicroVM на базе KVM, разработанный AWS для Lambda и Fargate. Минимальная поверхность атаки: нет BIOS, legacy устройств, только virtio. Старт ~125 мс. Только Linux guest. Написан на Rust. Тысячи изолированных VM на одном хосте — основа современных serverless-платформ.

**gVisor (runsc)**
Sandbox от Google: перехватывает системные вызовы приложения в user-space через компонент Sentry (реализует Linux-совместимое ядро на Go). Совместим с Docker и Kubernetes через runtime. Используется в GKE Sandbox. Оверхед на syscall-intensive приложениях (базы данных, компиляция).

**Kata Containers**
OCI-совместимые контейнеры, запускаемые внутри лёгких VM (QEMU, Firecracker, Cloud Hypervisor). Совмещает UX контейнеров с изоляцией уровня VM. Поддержка в Kubernetes через RuntimeClass. Компромисс: медленнее старт и чуть больше оверхед, чем runc, но полная изоляция ядра.

**WASM (Wasmtime / WasmEdge)**
WebAssembly — архитектурно-нейтральный байткод с capability-based security моделью. WASI (WebAssembly System Interface) — стандарт для syscall. Wasmtime (Bytecode Alliance), WasmEdge (CNCF) — server-side рантаймы. Ультимативно малый footprint (<1 MB), старт <1 мс. Ограничения: нет fork, ограниченные threads, развивающийся POSIX-сабсет.

---

## Не вошедшее в основную таблицу

| Технология | Описание |
|---|---|
| **VMware Workstation Player** | Бесплатная версия Workstation, без поддержки snapshots |
| **UTM** | macOS frontend над QEMU/Hypervisor.framework, популярен на Apple Silicon |
| **bhyve** | FreeBSD hypervisor, используется в TrueNAS, OmniOS |
| **Proxmox VE** | Дистрибутив на базе KVM + LXC с веб-интерфейсом управления |
| **Cloud Hypervisor** | Intel-проект на Rust, конкурент Firecracker (virtio, минимальный footprint) |
| **Amazon Nitro** | Кастомный гипервизор AWS на базе KVM + Nitro Cards (FPGA offload) |
| **Apple Hypervisor.framework** | Нативный macOS API для VM без Type 2 гипервизора поверх; основа UTM/Parallels на Apple Silicon |
| **Windows Sandbox** | Одноразовая Hyper-V VM для безопасного запуска приложений, встроена в Windows 10/11 Pro |
| **Vagrant** | Не гипервизор — инструмент для декларативного управления VM поверх VirtualBox/VMware/Hyper-V/libvirt |
| **Lima** | Lightweight Linux VM для macOS, основа Colima (rootless Docker/containerd на macOS) |

---

*Составлено с помощью Claude (Anthropic), апрель 2026*
