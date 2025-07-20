# Домашнее задание к занятию 5 «Тестирование roles»

## Подготовка к выполнению

1. Установите molecule и его драйвера: `pip3 install "molecule molecule_docker molecule_podman`.
2. Выполните `docker pull aragast/netology:latest` —  это образ с podman, tox и несколькими пайтонами (3.7 и 3.9) внутри.

## Основная часть

Ваша цель — настроить тестирование ваших ролей. 

Задача — сделать сценарии тестирования для vector. 

Ожидаемый результат — все сценарии успешно проходят тестирование ролей.

### Molecule

1. Запустите  `molecule test -s ubuntu_xenial` (или с любым другим сценарием, не имеет значения) внутри корневой директории clickhouse-role, посмотрите на вывод команды. Данная команда может отработать с ошибками или не отработать вовсе, это нормально. Наша цель - посмотреть как другие в реальном мире используют молекулу И из чего может состоять сценарий тестирования.
2. Перейдите в каталог с ролью vector-role и создайте сценарий тестирования по умолчанию при помощи `molecule init scenario --driver-name docker`.
3. Добавьте несколько разных дистрибутивов (oraclelinux:8, ubuntu:latest) для инстансов и протестируйте роль, исправьте найденные ошибки, если они есть.
4. Добавьте несколько assert в verify.yml-файл для  проверки работоспособности vector-role (проверка, что конфиг валидный, проверка успешности запуска и др.). 
5. Запустите тестирование роли повторно и проверьте, что оно прошло успешно.
5. Добавьте новый тег на коммит с рабочим сценарием в соответствии с семантическим версионированием.

## Выполнение
В общем, образы pycontrib уже давно не обновлялись и в них живёт python 3.6, что уже не подходит.
У меня возникала ошибка.

![IMG](/IMG/img1.png)

Поэтому я собрал образ из актуальной версии Centos 10 и Ubuntu 24.04 (Latest)
Установил на них необходимые вещи, взял за основу устанавливаемые пакеты слоя из образов pytcontrib
Затем, видимо в новой версии при `ansible.builtin.import_role` нужно указывать полный пусть namespace.role_name

---

Если возникает ошибка в Ubuntu 24.04:
`Failed to find required executable \"systemctl\" in paths: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin`

![IMG](/IMG/img2.png)

Добавить в `apt install systemctl`

---

Ошибка:
```
RUNNING HANDLER [chigridovko.vector : Restart vector service] ******************
fatal: [centos10_last]: FAILED! => {"changed": false, "msg": "Service is in unknown state", "status": {}}
```
По итогу нужно запустить контейнер в privileged mode
https://ansible.readthedocs.io/projects/molecule/guides/systemd-container/#systemd-container

Тогда systemctl в контейнере стал запускаться иначе ошибка:
```
System has not been booted with systemd as init system (PID 1). Can't operate.
Failed to connect to system scope bus via local transport: Host is down
```

---

По итогу имеем корректный тест:

<details>
<summary>Molecule test</summary>
<p>
WARNING  Driver docker does not provide a schema.
INFO     default scenario test matrix: dependency, cleanup, destroy, syntax, create, prepare, converge, idempotence, side_effect, verify, cleanup, destroy
INFO     Performing prerun with role_name_check=0...
INFO     Running default > dependency
WARNING  Skipping, missing the requirements file.
WARNING  Skipping, missing the requirements file.
INFO     Running default > cleanup
WARNING  Skipping, cleanup playbook not configured.
INFO     Running default > destroy
INFO     ansible-playbook version: ansible-playbook 
  config file = None
  configured module search path = ['/Users/konstantincigridov/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.13/site-packages/ansible
  ansible collection location = /Users/konstantincigridov/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible-playbook
  python version = 3.13.5 (main, Jun 11 2025, 15:36:57) [Clang 15.0.0 (clang-1500.1.0.2.5)] (/usr/local/opt/python@3.13/bin/python3.13)
  jinja version = 3.1.6
  libyaml = True
INFO     Sanity checks: 'docker'

PLAY [Destroy] *****************************************************************

TASK [Set async_dir for HOME env] **********************************************
ok: [localhost]

TASK [Destroy molecule instance(s)] ********************************************
changed: [localhost] => (item=centos_last)
changed: [localhost] => (item=ubuntu_latest)

TASK [Wait for instance(s) deletion to complete] *******************************
ok: [localhost] => (item=centos_last)
ok: [localhost] => (item=ubuntu_latest)

TASK [Delete docker networks(s)] ***********************************************
skipping: [localhost]

PLAY RECAP *********************************************************************
localhost                  : ok=3    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

INFO     Running default > syntax
INFO     ansible-playbook version: ansible-playbook 
  config file = None
  configured module search path = ['/Users/konstantincigridov/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.13/site-packages/ansible
  ansible collection location = /Users/konstantincigridov/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible-playbook
  python version = 3.13.5 (main, Jun 11 2025, 15:36:57) [Clang 15.0.0 (clang-1500.1.0.2.5)] (/usr/local/opt/python@3.13/bin/python3.13)
  jinja version = 3.1.6
  libyaml = True

playbook: /Users/konstantincigridov/Downloads/Devops/Ansible/AnsibleHW4/roles/vector-role/molecule/default/converge.yml
INFO     Running default > create
INFO     ansible-playbook version: ansible-playbook 
  config file = None
  configured module search path = ['/Users/konstantincigridov/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.13/site-packages/ansible
  ansible collection location = /Users/konstantincigridov/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible-playbook
  python version = 3.13.5 (main, Jun 11 2025, 15:36:57) [Clang 15.0.0 (clang-1500.1.0.2.5)] (/usr/local/opt/python@3.13/bin/python3.13)
  jinja version = 3.1.6
  libyaml = True

PLAY [Create] ******************************************************************

TASK [Set async_dir for HOME env] **********************************************
ok: [localhost]

TASK [Log into a Docker registry] **********************************************
skipping: [localhost] => (item=None) 
skipping: [localhost] => (item=None) 
skipping: [localhost]

TASK [Check presence of custom Dockerfiles] ************************************
ok: [localhost] => (item={'command': '/sbin/init', 'dockerfile': 'Dockerfile.centos', 'image': 'centos_last', 'name': 'centos_last', 'pre_build_image': False, 'privileged': True})
ok: [localhost] => (item={'dockerfile': 'Dockerfile', 'image': 'ubuntu_latest', 'name': 'ubuntu_latest', 'pre_build_image': False})

TASK [Create Dockerfiles from image names] *************************************
changed: [localhost] => (item={'command': '/sbin/init', 'dockerfile': 'Dockerfile.centos', 'image': 'centos_last', 'name': 'centos_last', 'pre_build_image': False, 'privileged': True})
changed: [localhost] => (item={'dockerfile': 'Dockerfile', 'image': 'ubuntu_latest', 'name': 'ubuntu_latest', 'pre_build_image': False})

TASK [Synchronization the context] *********************************************
changed: [localhost] => (item={'command': '/sbin/init', 'dockerfile': 'Dockerfile.centos', 'image': 'centos_last', 'name': 'centos_last', 'pre_build_image': False, 'privileged': True})
ok: [localhost] => (item=None)
changed: [localhost]

TASK [Discover local Docker images] ********************************************
ok: [localhost] => (item=None)
ok: [localhost] => (item=None)
ok: [localhost]

TASK [Build an Ansible compatible image (new)] *********************************
ok: [localhost] => (item=molecule_local/centos_last)
ok: [localhost] => (item=molecule_local/ubuntu_latest)

TASK [Create docker network(s)] ************************************************
skipping: [localhost]

TASK [Determine the CMD directives] ********************************************
ok: [localhost] => (item=None)
ok: [localhost] => (item=None)
ok: [localhost]

TASK [Create molecule instance(s)] *********************************************
changed: [localhost] => (item=centos_last)
changed: [localhost] => (item=ubuntu_latest)

TASK [Wait for instance(s) creation to complete] *******************************
changed: [localhost] => (item=None)
FAILED - RETRYING: [localhost]: Wait for instance(s) creation to complete (300 retries left).
changed: [localhost] => (item=None)
changed: [localhost]

PLAY RECAP *********************************************************************
localhost                  : ok=9    changed=4    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0

INFO     Running default > prepare
WARNING  Skipping, prepare playbook not configured.
INFO     Running default > converge
INFO     ansible-playbook version: ansible-playbook 
  config file = None
  configured module search path = ['/Users/konstantincigridov/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.13/site-packages/ansible
  ansible collection location = /Users/konstantincigridov/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible-playbook
  python version = 3.13.5 (main, Jun 11 2025, 15:36:57) [Clang 15.0.0 (clang-1500.1.0.2.5)] (/usr/local/opt/python@3.13/bin/python3.13)
  jinja version = 3.1.6
  libyaml = True

PLAY [Converge] ****************************************************************

TASK [Gathering Facts] *********************************************************
ok: [centos_last]
ok: [ubuntu_latest]

TASK [chigridovko.vector : Download Vector (CentOS)] ***************************
skipping: [ubuntu_latest]
changed: [centos_last]

TASK [chigridovko.vector : Install Vector (CentOS)] ****************************
skipping: [ubuntu_latest]
changed: [centos_last]

TASK [chigridovko.vector : Download Vector (Ubuntu)] ***************************
skipping: [centos_last]
changed: [ubuntu_latest]

TASK [chigridovko.vector : Install Vector (Ubuntu)] ****************************
skipping: [centos_last]
changed: [ubuntu_latest]

TASK [chigridovko.vector : Deploy Vector configuration] ************************
changed: [centos_last]
changed: [ubuntu_latest]

RUNNING HANDLER [chigridovko.vector : Restart vector service] ******************
changed: [centos_last]
changed: [ubuntu_latest]

PLAY RECAP *********************************************************************
centos_last                : ok=5    changed=4    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
ubuntu_latest              : ok=5    changed=4    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0

INFO     Running default > idempotence
INFO     ansible-playbook version: ansible-playbook 
  config file = None
  configured module search path = ['/Users/konstantincigridov/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.13/site-packages/ansible
  ansible collection location = /Users/konstantincigridov/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible-playbook
  python version = 3.13.5 (main, Jun 11 2025, 15:36:57) [Clang 15.0.0 (clang-1500.1.0.2.5)] (/usr/local/opt/python@3.13/bin/python3.13)
  jinja version = 3.1.6
  libyaml = True

PLAY [Converge] ****************************************************************

TASK [Gathering Facts] *********************************************************
ok: [centos_last]
ok: [ubuntu_latest]

TASK [chigridovko.vector : Download Vector (CentOS)] ***************************
skipping: [ubuntu_latest]
ok: [centos_last]

TASK [chigridovko.vector : Install Vector (CentOS)] ****************************
skipping: [ubuntu_latest]
ok: [centos_last]

TASK [chigridovko.vector : Download Vector (Ubuntu)] ***************************
skipping: [centos_last]
ok: [ubuntu_latest]

TASK [chigridovko.vector : Install Vector (Ubuntu)] ****************************
skipping: [centos_last]
ok: [ubuntu_latest]

TASK [chigridovko.vector : Deploy Vector configuration] ************************
ok: [centos_last]
ok: [ubuntu_latest]

PLAY RECAP *********************************************************************
centos_last                : ok=4    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
ubuntu_latest              : ok=4    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0

INFO     Idempotence completed successfully.
INFO     Running default > side_effect
WARNING  Skipping, side effect playbook not configured.
INFO     Running default > verify
INFO     Running Ansible Verifier
INFO     ansible-playbook version: ansible-playbook 
  config file = /Users/konstantincigridov/.ansible/tmp/molecule.9bzX.default/ansible.cfg
  configured module search path = ['/Users/konstantincigridov/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.13/site-packages/ansible
  ansible collection location = /Users/konstantincigridov/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible-playbook
  python version = 3.13.5 (main, Jun 11 2025, 15:36:57) [Clang 15.0.0 (clang-1500.1.0.2.5)] (/usr/local/opt/python@3.13/bin/python3.13)
  jinja version = 3.1.6
  libyaml = True

PLAY [Verify Vector installation] **********************************************

TASK [Gathering Facts] *********************************************************
ok: [centos_last]
ok: [ubuntu_latest]

TASK [Run 'vector validate'] ***************************************************
ok: [centos_last]
ok: [ubuntu_latest]

TASK [Check Vector service status] *********************************************
ok: [centos_last]
ok: [ubuntu_latest]

PLAY RECAP *********************************************************************
centos_last                : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu_latest              : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

INFO     Verifier completed successfully.
INFO     Running default > cleanup
WARNING  Skipping, cleanup playbook not configured.
INFO     Running default > destroy
INFO     ansible-playbook version: ansible-playbook 
  config file = /Users/konstantincigridov/.ansible/tmp/molecule.9bzX.default/ansible.cfg
  configured module search path = ['/Users/konstantincigridov/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.13/site-packages/ansible
  ansible collection location = /Users/konstantincigridov/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible-playbook
  python version = 3.13.5 (main, Jun 11 2025, 15:36:57) [Clang 15.0.0 (clang-1500.1.0.2.5)] (/usr/local/opt/python@3.13/bin/python3.13)
  jinja version = 3.1.6
  libyaml = True

PLAY [Destroy] *****************************************************************

TASK [Set async_dir for HOME env] **********************************************
ok: [localhost]

TASK [Destroy molecule instance(s)] ********************************************
changed: [localhost] => (item=centos_last)
changed: [localhost] => (item=ubuntu_latest)

TASK [Wait for instance(s) deletion to complete] *******************************
changed: [localhost] => (item=centos_last)
changed: [localhost] => (item=ubuntu_latest)

TASK [Delete docker networks(s)] ***********************************************
skipping: [localhost]

PLAY RECAP *********************************************************************
localhost                  : ok=3    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

INFO     Pruning extra files from scenario ephemeral directory
</p>
</details>

---

### Tox

1. Добавьте в директорию с vector-role файлы из [директории](./example).
2. Запустите `docker run --privileged=True -v <path_to_repo>:/opt/vector-role -w /opt/vector-role -it aragast/netology:latest /bin/bash`, где path_to_repo — путь до корня репозитория с vector-role на вашей файловой системе.
3. Внутри контейнера выполните команду `tox`, посмотрите на вывод.
5. Создайте облегчённый сценарий для `molecule` с драйвером `molecule_podman`. Проверьте его на исполнимость.
6. Пропишите правильную команду в `tox.ini`, чтобы запускался облегчённый сценарий.
8. Запустите команду `tox`. Убедитесь, что всё отработало успешно.
9. Добавьте новый тег на коммит с рабочим сценарием в соответствии с семантическим версионированием.

После выполнения у вас должно получится два сценария molecule и один tox.ini файл в репозитории. Не забудьте указать в ответе теги решений Tox и Molecule заданий. В качестве решения пришлите ссылку на  ваш репозиторий и скриншоты этапов выполнения задания. 