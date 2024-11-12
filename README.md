# Домашнее задание к занятию 2 «Работа с Playbook»

## Подготовка к выполнению

1. * Необязательно. Изучите, что такое [ClickHouse](https://www.youtube.com/watch?v=fjTNS2zkeBs) и [Vector](https://www.youtube.com/watch?v=CgEhyffisLY).
2. Создайте свой публичный репозиторий на GitHub с произвольным именем или используйте старый.
3. Скачайте [Playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.
4. Подготовьте хосты в соответствии с группами из предподготовленного playbook.

## Основная часть

1. Подготовьте свой inventory-файл `prod.yml`.


### Ответ: 
Подготавливаем inventory-файл [`prod.yml`](./playbook/inventory/prod.yml)

2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает [vector](https://vector.dev). Конфигурация vector должна деплоиться через template файл jinja2. От вас не требуется использовать все возможности шаблонизатора, просто вставьте стандартный конфиг в template файл. Информация по шаблонам по [ссылке](https://www.dmosk.ru/instruktions.php?object=ansible-nginx-install).

### Ответ: 
Дописал: [vars.yml](./playbook/group_vars/clickhouse/vars.yml) и [site.yml](./playbook/site.yml)

3. При создании tasks рекомендую использовать модули: `get_url`, `template`, `unarchive`, `file`.
4. Tasks должны: скачать дистрибутив нужной версии, выполнить распаковку в выбранную директорию, установить vector.

### Ответ: 
При выполнении Tasks скачался vector 0.31.0 и установился на удаленной машине.
```yaml
[user@compute-vm-2-2-10-hdd-1731409086781 ~]$ vector --version
vector 0.31.0 (x86_64-unknown-linux-gnu 0f13b22 2023-07-06 13:52:34.591204470)
```
5. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.

### Ответ: 
Ошибки:
```yaml
user@user-virtual-machine:~/Home work/pz-Ansible/mnt-homeworks/08-ansible-02-playbook/playbook$ ansible-lint site.yml
WARNING  Listing 6 violation(s) that are fatal
fqcn[action-core]: Use FQCN for builtin module actions (ansible.builtin.yum).
site.yml:25 Use `ansible.builtin.dnf` or `ansible.legacy.dnf` instead.

fqcn[action-core]: Use FQCN for builtin module actions (ansible.builtin.yum).
site.yml:58 Use `ansible.builtin.dnf` or `ansible.legacy.dnf` instead.

yaml[new-line-at-end-of-file]: No new line character at the end of file
site.yml:75

Read documentation for instructions on how to ignore specific rule violations.

                       Rule Violation Summary                        
 count tag                           profile    rule associated tags 
     1 yaml[new-line-at-end-of-file] basic      formatting, yaml     
     2 fqcn[action-core]             production formatting           

Failed: 3 failure(s), 0 warning(s) on 1 files. Last profile that met the validation criteria was 'min'.
```
Исправил ошибки:
```yaml
WARNING  Another version of 'vyos.vyos' 4.1.0 was found installed in /usr/local/lib/python3.10/dist-packages/ansible_collections, only the first one will be used, 4.1.0 (/home/user/.local/lib/python3.10/site-packages/ansible_collections).
WARNING  Another version of 'wti.remote' 1.0.5 was found installed in /usr/local/lib/python3.10/dist-packages/ansible_collections, only the first one will be used, 1.0.10 (/home/user/.local/lib/python3.10/site-packages/ansible_collections).

Passed: 0 failure(s), 0 warning(s) on 1 files. Last profile that met the validation criteria was 'production'.
```

6. Попробуйте запустить playbook на этом окружении с флагом `--check`.

### Ответ: 
Запускаем playbook с флагом --check:

```yaml
user@user-virtual-machine:~/Home work/pz-Ansible/mnt-homeworks/08-ansible-02-playbook/playbook$ ansible-playbook -i inventory/prod.yml site.yml --check -K 
BECOME password: 

PLAY [Install Clickhouse] ******************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
[WARNING]: Platform linux on host clickhouse-01 is using the discovered Python interpreter at /usr/bin/python3.9, but future installation of another Python interpreter could change the
meaning of that path. See https://docs.ansible.com/ansible-core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [clickhouse-01]

TASK [Get clickhouse distrib] **************************************************************************************************************************************************************
ok: [clickhouse-01] => (item=clickhouse-client)
ok: [clickhouse-01] => (item=clickhouse-server)
ok: [clickhouse-01] => (item=clickhouse-common-static)

TASK [Install clickhouse packages] *********************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Flush handlers to restart clickhouse] ************************************************************************************************************************************************

TASK [Create database] *********************************************************************************************************************************************************************
skipping: [clickhouse-01]

PLAY [Install vector] **********************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
[WARNING]: Platform linux on host vector-01 is using the discovered Python interpreter at /usr/bin/python3.9, but future installation of another Python interpreter could change the
meaning of that path. See https://docs.ansible.com/ansible-core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [vector-01]

TASK [Get vector distrib] ******************************************************************************************************************************************************************
ok: [vector-01]

TASK [Install vector packages] *************************************************************************************************************************************************************
ok: [vector-01]

TASK [Flush handlers to restart vector] ****************************************************************************************************************************************************

TASK [Configure Vector | ensure what directory exists] *************************************************************************************************************************************
ok: [vector-01]

TASK [Configure Vector | Template config] **************************************************************************************************************************************************
ok: [vector-01]

PLAY RECAP *********************************************************************************************************************************************************************************
clickhouse-01              : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
vector-01                  : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

7. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.

### Ответ: 
Запускаем playbook с флагом --diff:
```yaml
user@user-virtual-machine:~/Home work/pz-Ansible/mnt-homeworks/08-ansible-02-playbook/playbook$ ansible-playbook -i inventory/prod.yml site.yml --diff -K 
BECOME password: 

PLAY [Install Clickhouse] ******************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
[WARNING]: Platform linux on host clickhouse-01 is using the discovered Python interpreter at /usr/bin/python3.9, but future installation of another Python interpreter could change the
meaning of that path. See https://docs.ansible.com/ansible-core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [clickhouse-01]

TASK [Get clickhouse distrib] **************************************************************************************************************************************************************
ok: [clickhouse-01] => (item=clickhouse-client)
ok: [clickhouse-01] => (item=clickhouse-server)
ok: [clickhouse-01] => (item=clickhouse-common-static)

TASK [Install clickhouse packages] *********************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Flush handlers to restart clickhouse] ************************************************************************************************************************************************

TASK [Create database] *********************************************************************************************************************************************************************
ok: [clickhouse-01]

PLAY [Install vector] **********************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
[WARNING]: Platform linux on host vector-01 is using the discovered Python interpreter at /usr/bin/python3.9, but future installation of another Python interpreter could change the
meaning of that path. See https://docs.ansible.com/ansible-core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [vector-01]

TASK [Get vector distrib] ******************************************************************************************************************************************************************
ok: [vector-01]

TASK [Install vector packages] *************************************************************************************************************************************************************
ok: [vector-01]

TASK [Flush handlers to restart vector] ****************************************************************************************************************************************************

TASK [Configure Vector | ensure what directory exists] *************************************************************************************************************************************
ok: [vector-01]

TASK [Configure Vector | Template config] **************************************************************************************************************************************************
ok: [vector-01]

PLAY RECAP *********************************************************************************************************************************************************************************
clickhouse-01              : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
vector-01                  : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

8. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.

### Ответ: 
Еще раз запускаем playbook с флагом --diff:
Playbook идемпотентен.
```yaml
user@user-virtual-machine:~/Home work/pz-Ansible/mnt-homeworks/08-ansible-02-playbook/playbook$ ansible-playbook -i inventory/prod.yml site.yml --check -K 
BECOME password: 

PLAY [Install Clickhouse] ******************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
[WARNING]: Platform linux on host clickhouse-01 is using the discovered Python interpreter at /usr/bin/python3.9, but future installation of another Python interpreter could change the
meaning of that path. See https://docs.ansible.com/ansible-core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [clickhouse-01]

TASK [Get clickhouse distrib] **************************************************************************************************************************************************************
ok: [clickhouse-01] => (item=clickhouse-client)
ok: [clickhouse-01] => (item=clickhouse-server)
ok: [clickhouse-01] => (item=clickhouse-common-static)

TASK [Install clickhouse packages] *********************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Flush handlers to restart clickhouse] ************************************************************************************************************************************************

TASK [Create database] *********************************************************************************************************************************************************************
skipping: [clickhouse-01]

PLAY [Install vector] **********************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
[WARNING]: Platform linux on host vector-01 is using the discovered Python interpreter at /usr/bin/python3.9, but future installation of another Python interpreter could change the
meaning of that path. See https://docs.ansible.com/ansible-core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [vector-01]

TASK [Get vector distrib] ******************************************************************************************************************************************************************
ok: [vector-01]

TASK [Install vector packages] *************************************************************************************************************************************************************
ok: [vector-01]

TASK [Flush handlers to restart vector] ****************************************************************************************************************************************************

TASK [Configure Vector | ensure what directory exists] *************************************************************************************************************************************
ok: [vector-01]

TASK [Configure Vector | Template config] **************************************************************************************************************************************************
ok: [vector-01]

PLAY RECAP *********************************************************************************************************************************************************************************
clickhouse-01              : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
vector-01                  : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

user@user-virtual-machine:~/Home work/pz-Ansible/mnt-homeworks/08-ansible-02-playbook/playbook$ ansible-playbook -i inventory/prod.yml site.yml --diff -K 
BECOME password: 

PLAY [Install Clickhouse] ******************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
[WARNING]: Platform linux on host clickhouse-01 is using the discovered Python interpreter at /usr/bin/python3.9, but future installation of another Python interpreter could change the
meaning of that path. See https://docs.ansible.com/ansible-core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [clickhouse-01]

TASK [Get clickhouse distrib] **************************************************************************************************************************************************************
ok: [clickhouse-01] => (item=clickhouse-client)
ok: [clickhouse-01] => (item=clickhouse-server)
ok: [clickhouse-01] => (item=clickhouse-common-static)

TASK [Install clickhouse packages] *********************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Flush handlers to restart clickhouse] ************************************************************************************************************************************************

TASK [Create database] *********************************************************************************************************************************************************************
ok: [clickhouse-01]

PLAY [Install vector] **********************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
[WARNING]: Platform linux on host vector-01 is using the discovered Python interpreter at /usr/bin/python3.9, but future installation of another Python interpreter could change the
meaning of that path. See https://docs.ansible.com/ansible-core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [vector-01]

TASK [Get vector distrib] ******************************************************************************************************************************************************************
ok: [vector-01]

TASK [Install vector packages] *************************************************************************************************************************************************************
ok: [vector-01]

TASK [Flush handlers to restart vector] ****************************************************************************************************************************************************

TASK [Configure Vector | ensure what directory exists] *************************************************************************************************************************************
ok: [vector-01]

TASK [Configure Vector | Template config] **************************************************************************************************************************************************
ok: [vector-01]

PLAY RECAP *********************************************************************************************************************************************************************************
clickhouse-01              : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
vector-01                  : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

user@user-virtual-machine:~/Home work/pz-Ansible/mnt-homeworks/08-ansible-02-playbook/playbook$ ansible-playbook -i inventory/prod.yml site.yml --diff -K 
BECOME password: 

PLAY [Install Clickhouse] ******************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
[WARNING]: Platform linux on host clickhouse-01 is using the discovered Python interpreter at /usr/bin/python3.9, but future installation of another Python interpreter could change the
meaning of that path. See https://docs.ansible.com/ansible-core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [clickhouse-01]

TASK [Get clickhouse distrib] **************************************************************************************************************************************************************
ok: [clickhouse-01] => (item=clickhouse-client)
ok: [clickhouse-01] => (item=clickhouse-server)
ok: [clickhouse-01] => (item=clickhouse-common-static)

TASK [Install clickhouse packages] *********************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Flush handlers to restart clickhouse] ************************************************************************************************************************************************

TASK [Create database] *********************************************************************************************************************************************************************
ok: [clickhouse-01]

PLAY [Install vector] **********************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
[WARNING]: Platform linux on host vector-01 is using the discovered Python interpreter at /usr/bin/python3.9, but future installation of another Python interpreter could change the
meaning of that path. See https://docs.ansible.com/ansible-core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [vector-01]

TASK [Get vector distrib] ******************************************************************************************************************************************************************
ok: [vector-01]

TASK [Install vector packages] *************************************************************************************************************************************************************
ok: [vector-01]

TASK [Flush handlers to restart vector] ****************************************************************************************************************************************************

TASK [Configure Vector | ensure what directory exists] *************************************************************************************************************************************
ok: [vector-01]

TASK [Configure Vector | Template config] **************************************************************************************************************************************************
ok: [vector-01]

PLAY RECAP *********************************************************************************************************************************************************************************
clickhouse-01              : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
vector-01                  : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

9. Подготовьте README.md-файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги. Пример качественной документации ansible playbook по [ссылке](https://github.com/opensearch-project/ansible-playbook).
10. Готовый playbook выложите в свой репозиторий, поставьте тег `08-ansible-02-playbook` на фиксирующий коммит, в ответ предоставьте ссылку на него.

---

### Как оформить решение задания

Выполненное домашнее задание пришлите в виде ссылки на .md-файл в вашем репозитории.

---