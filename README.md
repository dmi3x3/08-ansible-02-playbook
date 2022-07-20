# Домашнее задание к занятию "08.03 Использование Yandex Cloud"

## Подготовка к выполнению

1. Подготовьте в Yandex Cloud три хоста: для `clickhouse`, для `vector` и для `lighthouse`.

Ссылка на репозиторий LightHouse: https://github.com/VKCOM/lighthouse

## Основная часть

1. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает lighthouse.
2. При создании tasks рекомендую использовать модули: `get_url`, `template`, `yum`, `apt`.
3. Tasks должны: скачать статику lighthouse, установить nginx или любой другой webserver, настроить его конфиг для открытия lighthouse, запустить webserver.
4. Приготовьте свой собственный inventory файл `prod.yml`.

```yaml
---
clickhouse:
  hosts:
    clickhouse-01:
      ansible_host: root@centos7-n
vector:
  hosts:
    vector-01:
      ansible_host: root@centos7-n
lighthouse:
  hosts:
    lighthouse-01:
      ansible_host: root@centos7-n
```

5. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.

```shell
08-ansible-03-yandex$ ansible-lint site.yml
WARNING  Overriding detected file kind 'yaml' with 'playbook' for given positional argument: site.yml
```

6. Попробуйте запустить playbook на этом окружении с флагом `--check`.
7. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.

В play для lighthouse назначал права на каталог после задачи git clone (в процессе чего каталог и появлялся) и не понимал, почему в этом play всегда одно изменение, оказалось, что git clone запускается с от root, и каждый запуск меняет права на клонированную директорию,
для решения этой проблемы перенес назначение прав на каталог уровнем выше (в ту папку, куда происходит клонирование) и саму задачу поставил до клонирования. А его запустил от пользователя nginx, с помощью опции become_user: nginx. 
В play для clickhouse для задачи "Create table {{ clickhouse_table4logs }}.logs" добавил параметр changed_when: create_table.rc != 0 т.о. при успешном завершении задачи он считается не измененной, считаю это нормальным, т.к. в sql запросе на создание таблицы из этого файла используется конструкция CREATE TABLE IF NOT EXISTS, которая не допустит ошибки если таблица уже существует.

8. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.
9. Подготовьте README.md файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.

т.к. занный playbook является доработкой предыдущего ДЗ, расскажу только про новые сущности. Появился тег ligthouse, которым маркированы две новые задачи. Первая устанавливает nginx, вторая lighthouse.
Весь плейбук претерпел некоторые изменения, связанные с возможностью запуска его для нескольких разных хостов. Для этого пришлось добавить возможность получать ip адреса хостов и прописывать их в конфиги перекрестно. Например в задаче для vector получить ip хоста с clickhouse, это обеспечивает конструкция:

```yaml
    - name: Set IP addresd clickhouse
      set_fact:
        clickhouse_node_ip: "{{ item }}"
      with_items:
        - "{{ hostvars['clickhouse-01']['ansible_facts']['default_ipv4']['address'] }}"
      tags: vector
```

Постарался свести к минимуму ручные действия при использовании данного playbook, требуется только прописать ip адреса 3 хостов с centos7 в inventory и он (playbook) настроит и запустит хосты с clickhouse, vector и lighthouse.
C хоста lighthouse-01 сервис nginx отправляет подготовленные в формате json access логи по протоколу syslog udp на машину с vector. Vector-01 обрабатывает логи выставляет формат полей, некоторые переименовавает, удаляет и отправляет в БД clickhouse, на хост clickhouse-01, записи в котором можно посмотреть из lighthouse.


10. Готовый playbook выложите в свой репозиторий, поставьте тег `08-ansible-03-yandex` на фиксирующий коммит, в ответ предоставьте ссылку на него.

### [Готовый playbook 08-ansible-03-yandex](https://github.com/dmi3x3/08-ansible-02-playbook/tree/08-ansible-03-yandex) 



[Предыдущее ДЗ](https://github.com/dmi3x3/08-ansible-02-playbook/tree/08-ansible-02-playbook)

---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
