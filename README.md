<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/d/d8/Red_Hat_logo.svg/800px-Red_Hat_logo.svg.png" alt="Red Hat Banner" width="20%">
</p>

## ![Lesson](https://img.shields.io/badge/Lesson-otus__kernel-0A84FF?style=for-the-badge&logo=linux&logoColor=white&labelColor=111827)![Author](https://img.shields.io/badge/Author-Kamil%20Ibragimov-10B981?style=for-the-badge&logo=github&logoColor=white&labelColor=111827)![Date](https://img.shields.io/badge/Date-15.12.2025-F59E0B?style=for-the-badge&logo=calendar&logoColor=white&labelColor=111827)

### 📌 Задание
1. Обновить ядро ОС (AlmaLinux 9) до актуальной версии `mainline`.
2. Использовать репозиторий ELRepo.
3. Настроить автоматическую загрузку с новым ядром через Vagrant provision.

### ✅ Результат
- [x] Ядро обновлено до версии **6.18.1**. Результат см. на скриншоте 🖼️ ["success_kernel.png"](success_kernel.png)
- [x] Реализован обход блокировок Vagrant Cloud (использовано зеркало РФ).
- [x] Настроен автоматический выбор ядра через `grubby`.

### 🧭 Оглавление
- [🧰 Шаг 1 - Подготовка Vagrantfile](#one)
- [🧰 Шаг 2 - Запуск и установка](#two)
- [🧰 Шаг 3 - Проверка результата](#three)

---

<a id="one"></a>
## 🧰 Шаг 1 - Подготовка Vagrantfile

Создаем файл `Vagrantfile` с инструкциями для автоматического обновления.
**Особенности решения:**
* Используется зеркало `vagrant.elab.pro` для скачивания образа.
* Скрипт сам находит новое ядро и делает его загружаемым по умолчанию.

```bash
# -*- mode: ruby -*-
# vi: set ft=ruby :

# === 1. Обход региональных блокировок (зеркало РФ) ===
ENV['VAGRANT_SERVER_URL'] = '[https://vagrant.elab.pro](https://vagrant.elab.pro)'

Vagrant.configure("2") do |config|
  config.vm.box = "almalinux/9"
  
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus = 2
  end

  # === 2. Скрипт обновления ядра ===
  config.vm.provision "shell", inline: <<-SHELL
    # Установка ключей и репозитория ELRepo
    rpm --import [https://www.elrepo.org/RPM-GPG-KEY-elrepo.org](https://www.elrepo.org/RPM-GPG-KEY-elrepo.org)
    dnf install -y [https://www.elrepo.org/elrepo-release-9.el9.elrepo.noarch.rpm](https://www.elrepo.org/elrepo-release-9.el9.elrepo.noarch.rpm)
    
    # Установка ядра Mainline (ml)
    dnf --enablerepo=elrepo-kernel install -y kernel-ml

    # Поиск установленного ядра и настройка GRUB
    # Ищем файл vmlinuz, содержащий "elrepo" в названии
    NEW_KERNEL=$(ls /boot/vmlinuz*elrepo* | head -n 1)
    
    if [ -z "$NEW_KERNEL" ]; then
        echo "ОШИБКА: Ядро не найдено!"
        exit 1
    else
        grubby --set-default "$NEW_KERNEL"
        echo "УСПЕХ: Выбрано ядро $NEW_KERNEL"
    fi
  SHELL
end
```

<a id="two"></a>
🧰 Шаг 2 - Запуск и установка
Разворачиваем виртуальную машину. Скрипт выполнится автоматически при первом запуске.

```bash
vagrant up
```

<a id="three"></a>
🧰 Шаг 3 - Проверка результата
Перезагружаем машину для применения изменений и проверяем версию ядра.
```bash
# Перезагрузка хоста
vagrant reload

# Подключение
vagrant ssh

# Проверка версии
uname -r
Ожидаемый вывод:

Bash

6.18.1-1.el9.elrepo.x86_64
```
