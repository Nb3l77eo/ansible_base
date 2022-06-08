# Описание playbook.

## Применение роли "Install Clickhouse role"
```yaml
- name: Install Clickhouse role
  hosts: clickhouse
  roles:
    - clickhouse
```

## Создание таблицы my_table для записи логов
```yaml
- name: Create my_table
  hosts: clickhouse
  tasks:
  - ansible.builtin.command: "clickhouse-client -q 'CREATE TABLE logs.my_table
      (
          `appname` String,
          `facility` String,
          `hostname` String,
          `message` String,
          `msgid` String,
          `procid` UInt32,
          `severity` String,
          `timestamp` String,
          `version` UInt32
      )
      ENGINE = MergeTree
      ORDER BY timestamp;'"
```

## Применение роли "Install vector role"
```yaml
- name: Install vector role
  hosts: vector
  roles:
    - vector
```

## Применение роли "Install vector role"
```yaml
- name: Install lighthouse role
  hosts: lighthouse
  roles:
    - lighthouse
```

## Задачи по установке web сервера "Install nginx"
```yaml
- name: Install nginx
  hosts: lighthouse
  tasks:
    - name: Install epel-release
      become: true
      ansible.builtin.yum:
        name:
        - epel-release
        state: present

    - name: Install nginx
      become: true
      ansible.builtin.yum:
        name:
        - nginx
        state: installed

    - name: Add conf lighthouse to nginx
      become: true
      ansible.builtin.template:
        src: nginx-lighthouse.conf.j2
        dest: /etc/nginx/conf.d/lighthouse.conf
        owner: nginx
        group: wheel
        mode: '0644'

    - name: Start nginx
      become: true
      ansible.builtin.service:
        name: nginx
        state: restarted
```