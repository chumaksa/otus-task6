# otus-task6

# Размещаем свой RPM в своем репозитории

1. Создать свой RPM пакет (можно взять свое приложение, либо собрать, например, апач с определенными опциями);
2. Создать свой репозиторий и разместить там ранее собранный RPM.

## Создать свой RPM пакет

### Решение

Для примера возьмём пакет NGINX и соберём его с поддержкой Brotli. \

Для выполнения задания будем использовать виртуальную машину с ОС AlmaLinux 9. \
ВМ будем разворачивать с помощью Vagrant. Vagrantfile следющего содержания:
```

# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "almalinux/9"

  config.vm.provider "virtualbox" do |v|
    v.memory = 2048
    v.cpus = 1
  end

end
```

Подготовку тестового стенда будем выполнять с помощью Ansible.
