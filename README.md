# Описание playbook.

## Play: "Install Clickhouse"
Данный play состоит из тасок выполняемых на группе узлов "clickhouse".<p>
Используется для установки Clickhouse. Выполняется на группе хостов "clickhouse".

### Task: Get clickhouse distrib 
```
- block:
    - name: Get clickhouse distrib
        ansible.builtin.get_url:
        url: "https://packages.clickhouse.com/rpm/stable/{{ item }}-{{ clickhouse_version }}.noarch.rpm"
        dest: "./{{ item }}-{{ clickhouse_version }}.rpm"
        with_items: "{{ clickhouse_packages }}"
    rescue:
    - name: Get clickhouse distrib
        ansible.builtin.get_url:
        url: "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-{{ clickhouse_version }}.x86_64.rpm"
        dest: "./clickhouse-common-static-{{ clickhouse_version }}.rpm"
```
Таска загрузки дистрибутивов необходимы для установки clickhouse.
Использует модуль get_url  - загрузка фалов по http,https,ftp
С атрибутами: url - что загружаем и dest - куда.
Блок rescue используется для запуска таски ошибки выполнения вышестоящей(их) таски.
{{ item }} - перечисление пакетов необходимых к установке указанных в group_vars/clickhouse/vars.yml


### Task: Install clickhouse packages 
```
- name: Install clickhouse packages
    become: true
    ansible.builtin.yum:
    name:
        - clickhouse-server-{{ clickhouse_version }}.rpm
        - clickhouse-common-static-{{ clickhouse_version }}.rpm
        - clickhouse-client-{{ clickhouse_version }}.rpm
```
Установка пакетов rpm используя модуль yum. {{ clickhouse_version }} - Переменная с версией необходимого пакета clickhouse указанной в group_vars/clickhouse/vars.yml. become - указывает на необходимость выполнения данной таски с повышением прав.



### Task: Start clickhouse service 
```
- name: Start clickhouse service
    become: true
    ansible.builtin.service:
    name: clickhouse-server
    state: restarted
```
Модуль управления службами. В данном описании мы перезапускам необходимую нам службу указав атрибут name (имя службы) и state (необходимый результат после исполнения модуля)

### Task: Create database 
```
- name: Create database
    ansible.builtin.command: "clickhouse-client -q 'create database logs;'"
    register: create_db
    failed_when: create_db.rc != 0 and create_db.rc !=82
    changed_when: create_db.rc == 0
```
ansible.builtin.command - выполнение указанной команды на хосте. В данном случае мы создаем базу. <p>
register - регистрация результата выполнения таски в переменную. <p>
Описание условий статуса выполнения таски:<p>
failed_when: create_db.rc != 0 and create_db.rc !=82 - выполнение будет считаться неуспешным в случае если код выполнения команды не равен 0 или 82 <p>
changed_when: create_db.rc == 0 - успешное выполнение таски если код возврата равен 0.


## Play: "Install vector"
Выполняется на группе хостов "vector".

### Task: Download vector 
Загрузка дистрибутивов "vector".

### Task: Install vector 
Установка "vector" из ранее загруженных дистрибутивов.

### Task: Get vector version 
Таска запроса установленной версии вектора ("vector --version") - в случае наличия установленного вектора таска не вернет положительный результат.

### Task: Create vector config
Создание конфигурационного файла vector

### Task: Validate vector config
Проверка конфигурационного файла vector

### Task: Enable vector service 
Перезапуск vector

## Play: "Install lighthouse"

### Task: Install epel-release
Подключение репозитория и установка утилит.

### Task: Install nginx
Установка web сервера.

### Task: Download lighthouse
Загрузка пакета lighthouse.

### Task: Unzip lighthouse
Распаковка архива с lighthouse. 

### Task: Add conf lighthouse to nginx
Добавление файла конфигурации для возможности обращаться к clickhouse не только по 127.0.0.1.

### Task: Start nginx
Запуск Web сервера.