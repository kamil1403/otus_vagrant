<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/4/4e/AlmaLinux_OS_logo.svg/800px-AlmaLinux_OS_logo.svg.png" alt="AlmaLinux Banner" width="30%">
</p>

## ![Lesson](https://img.shields.io/badge/Lesson-otus__vagrant-0A84FF?style=for-the-badge&logo=linux&logoColor=white&labelColor=111827)![Author](https://img.shields.io/badge/Author-Kamil%20Ibragimov-10B981?style=for-the-badge&logo=github&logoColor=white&labelColor=111827)![Date](https://img.shields.io/badge/Date-15.12.2025-F59E0B?style=for-the-badge&logo=calendar&logoColor=white&labelColor=111827)

### 📌 Задание
1. Обновить ядро ОС (AlmaLinux 9) до актуальной версии `mainline`.

### ✅ Результат
- [x] Ядро обновлено до версии **6.18.1**. Результат см. на скриншоте 🖼️ ["success_kernel.png"](success_kernel.png)

### 🧭 Оглавление
- [🧰 Шаг 1 - Подготовка Vagrantfile](#one)
- [🧰 Шаг 2 - Запуск и установка](#two)
- [🧰 Шаг 3 - Проверка результата](#three)

---

<a id="one"></a>
## 🧰 Шаг 1 - Подготовка Vagrantfile

Создаем файл `Vagrantfile`
```bash
# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_SERVER_URL'] = '[https://vagrant.elab.pro](https://vagrant.elab.pro)'

Vagrant.configure("2") do |config|
  config.vm.box = "almalinux/9"
  
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus = 2
  end

  config.vm.provision "shell", inline: <<-SHELL
    rpm --import [https://www.elrepo.org/RPM-GPG-KEY-elrepo.org](https://www.elrepo.org/RPM-GPG-KEY-elrepo.org)
    dnf install -y [https://www.elrepo.org/elrepo-release-9.el9.elrepo.noarch.rpm](https://www.elrepo.org/elrepo-release-9.el9.elrepo.noarch.rpm)
    
    dnf --enablerepo=elrepo-kernel install -y kernel-ml
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

```bash
vagrant up
```

<a id="three"></a>
🧰 Шаг 3 - Проверка

```bash
# Перезагрузка виртуальной машины
vagrant reload

# Подключение
vagrant ssh

# Проверка
uname -r

#Вывод:
6.18.1-1.el9.elrepo.x86_64
```
