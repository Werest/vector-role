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


# Ссылка на роль
[vector-role](https://github.com/Werest/vector-role)

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

```bash
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
```
</details>

---

### Tox

1. Добавьте в директорию с vector-role файлы из [директории](https://github.com/netology-code/mnt-homeworks/tree/MNT-video/08-ansible-05-testing/example).
2. Запустите `docker run --privileged=True -v <path_to_repo>:/opt/vector-role -w /opt/vector-role -it aragast/netology:latest /bin/bash`, где path_to_repo — путь до корня репозитория с vector-role на вашей файловой системе.
3. Внутри контейнера выполните команду `tox`, посмотрите на вывод.
4. Создайте облегчённый сценарий для `molecule` с драйвером `molecule_podman`. Проверьте его на исполнимость.
5. Пропишите правильную команду в `tox.ini`, чтобы запускался облегчённый сценарий.
6. Запустите команду `tox`. Убедитесь, что всё отработало успешно.
7. Добавьте новый тег на коммит с рабочим сценарием в соответствии с семантическим версионированием.

После выполнения у вас должно получится два сценария molecule и один tox.ini файл в репозитории. Не забудьте указать в ответе теги решений Tox и Molecule заданий. В качестве решения пришлите ссылку на  ваш репозиторий и скриншоты этапов выполнения задания.

## Выполнение
docker run --privileged=True -v /Users/konstantincigridov/Downloads/Devops/Ansible/AnsibleHW4/roles/vector-role:/opt/vector-role -w /opt/vector-role -it aragast/netology:latest /bin/bash

![img](/IMG/tox1.png)

```
molecule init scenario podman --driver-name podman
INFO     Initializing new scenario podman...

PLAY [Create a new molecule scenario] ******************************************

TASK [Check if destination folder exists] **************************************
changed: [localhost]

TASK [Check if destination folder is empty] ************************************
ok: [localhost]

TASK [Fail if destination folder is not empty] *********************************
skipping: [localhost]

TASK [Expand templates] ********************************************************
skipping: [localhost] => (item=molecule/podman/destroy.yml) 
changed: [localhost] => (item=molecule/podman/molecule.yml)
changed: [localhost] => (item=molecule/podman/converge.yml)
skipping: [localhost] => (item=molecule/podman/create.yml) 

PLAY RECAP *********************************************************************
localhost                  : ok=3    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

INFO     Initialized scenario in /Users/konstantincigridov/Downloads/Devops/Ansible/AnsibleHW4/roles/vector-role/molecule/podman successfully.
```

---

```
molecule test -s podman
WARNING  Driver podman does not provide a schema.
INFO     podman scenario test matrix: dependency, cleanup, destroy, syntax, create, prepare, converge, idempotence, side_effect, verify, cleanup, destroy
INFO     Performing prerun with role_name_check=0...
INFO     Running podman > dependency
WARNING  Skipping, missing the requirements file.
WARNING  Skipping, missing the requirements file.
INFO     Running podman > cleanup
WARNING  Skipping, cleanup playbook not configured.
INFO     Running podman > destroy
INFO     ansible-playbook version: ansible-playbook 
  config file = None
  configured module search path = ['/Users/konstantincigridov/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.13/site-packages/ansible
  ansible collection location = /Users/konstantincigridov/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible-playbook
  python version = 3.13.5 (main, Jun 11 2025, 15:36:57) [Clang 15.0.0 (clang-1500.1.0.2.5)] (/usr/local/opt/python@3.13/bin/python3.13)
  jinja version = 3.1.6
  libyaml = True
INFO     Sanity checks: 'podman'

PLAY [Destroy] *****************************************************************

TASK [Set async_dir for HOME env] **********************************************
ok: [localhost]

TASK [Destroy molecule instance(s)] ********************************************
changed: [localhost] => (item={'image': 'quay.io/centos/centos:stream8', 'name': 'instance', 'pre_build_image': True})

TASK [Wait for instance(s) deletion to complete] *******************************
changed: [localhost] => (item={'failed': 0, 'started': 1, 'finished': 0, 'ansible_job_id': 'j257307823207.2232', 'results_file': '/Users/konstantincigridov/.ansible_async/j257307823207.2232', 'changed': True, 'item': {'image': 'quay.io/centos/centos:stream8', 'name': 'instance', 'pre_build_image': True}, 'ansible_loop_var': 'item'})

PLAY RECAP *********************************************************************
localhost                  : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

INFO     Running podman > syntax
INFO     ansible-playbook version: ansible-playbook 
  config file = None
  configured module search path = ['/Users/konstantincigridov/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.13/site-packages/ansible
  ansible collection location = /Users/konstantincigridov/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible-playbook
  python version = 3.13.5 (main, Jun 11 2025, 15:36:57) [Clang 15.0.0 (clang-1500.1.0.2.5)] (/usr/local/opt/python@3.13/bin/python3.13)
  jinja version = 3.1.6
  libyaml = True

playbook: /Users/konstantincigridov/Downloads/Devops/Ansible/AnsibleHW4/roles/vector-role/molecule/podman/converge.yml
INFO     Running podman > create
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

TASK [get podman executable path] **********************************************
fatal: [localhost]: FAILED! => {"changed": false, "cmd": ["which", "podman"], "delta": "0:00:00.014125", "end": "2025-07-21 22:31:39.065358", "msg": "non-zero return code", "rc": 1, "start": "2025-07-21 22:31:39.051233", "stderr": "", "stderr_lines": [], "stdout": "", "stdout_lines": []}

PLAY RECAP *********************************************************************
localhost                  : ok=0    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0

CRITICAL Ansible return code was 2, command was: ansible-playbook --inventory /Users/konstantincigridov/.ansible/tmp/molecule.9bzX.podman/inventory --skip-tags molecule-notest,notest /usr/local/lib/python3.13/site-packages/molecule_podman/playbooks/create.yml
WARNING  An error occurred during the test sequence action: 'create'. Cleaning up.
INFO     Running podman > cleanup
WARNING  Skipping, cleanup playbook not configured.
INFO     Running podman > destroy
INFO     ansible-playbook version: ansible-playbook 
  config file = None
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
changed: [localhost] => (item={'image': 'quay.io/centos/centos:stream8', 'name': 'instance', 'pre_build_image': True})

TASK [Wait for instance(s) deletion to complete] *******************************
changed: [localhost] => (item={'failed': 0, 'started': 1, 'finished': 0, 'ansible_job_id': 'j9597430453.2303', 'results_file': '/Users/konstantincigridov/.ansible_async/j9597430453.2303', 'changed': True, 'item': {'image': 'quay.io/centos/centos:stream8', 'name': 'instance', 'pre_build_image': True}, 'ansible_loop_var': 'item'})

PLAY RECAP *********************************************************************
localhost                  : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

INFO     Pruning extra files from scenario ephemeral directory
```

---

tox
```
[root@016d3024faa0 vector-role]# tox
py37-ansible210 installed: ansible==2.10.7,ansible-base==2.10.17,ansible-compat==1.0.0,arrow==1.2.3,bcrypt==4.2.1,binaryornot==0.4.4,cached-property==1.5.2,Cerberus==1.3.7,certifi==2025.7.14,cffi==1.15.1,chardet==5.2.0,charset-normalizer==3.4.2,click==8.1.8,click-help-colors==0.9.4,cookiecutter==2.6.0,cryptography==45.0.5,distro==1.9.0,enrich==1.2.7,idna==3.10,importlib-metadata==6.7.0,Jinja2==3.1.6,jmespath==1.0.1,lxml==5.4.0,markdown-it-py==2.2.0,MarkupSafe==2.1.5,mdurl==0.1.2,molecule==3.6.1,molecule-podman==1.1.0,packaging==24.0,paramiko==2.12.0,pluggy==1.2.0,pycparser==2.21,Pygments==2.17.2,PyNaCl==1.5.0,python-dateutil==2.9.0.post0,python-slugify==8.0.4,PyYAML==6.0.1,requests==2.31.0,rich==13.8.1,selinux==0.2.1,six==1.17.0,subprocess-tee==0.3.5,text-unidecode==1.3,typing_extensions==4.7.1,urllib3==2.0.7,zipp==3.15.0
py37-ansible210 run-test-pre: PYTHONHASHSEED='3924348855'
py37-ansible210 run-test: commands[0] | molecule test -s podman --destroy always
/opt/vector-role/.tox/py37-ansible210/lib/python3.7/site-packages/ansible/parsing/vault/__init__.py:44: CryptographyDeprecationWarning: Python 3.7 is no longer supported by the Python core team and support for it is deprecated in cryptography. The next release of cryptography will remove support for Python 3.7.
  from cryptography.exceptions import InvalidSignature
INFO     podman scenario test matrix: destroy, create, converge, verify, destroy
INFO     Performing prerun...
INFO     Set ANSIBLE_LIBRARY=/root/.cache/ansible-compat/b984a4/modules:/root/.ansible/plugins/modules:/usr/share/ansible/plugins/modules
INFO     Set ANSIBLE_COLLECTIONS_PATH=/root/.cache/ansible-compat/b984a4/collections:/root/.ansible/collections:/usr/share/ansible/collections
INFO     Set ANSIBLE_ROLES_PATH=/root/.cache/ansible-compat/b984a4/roles:/root/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles
INFO     Using /root/.ansible/roles/chigridovko.vector symlink to current repository in order to enable Ansible to find the role using its expected full name.
INFO     Running podman > destroy
INFO     Sanity checks: 'podman'
/opt/vector-role/.tox/py37-ansible210/lib/python3.7/site-packages/ansible/parsing/vault/__init__.py:44: CryptographyDeprecationWarning: Python 3.7 is no longer supported by the Python core team and support for it is deprecated in cryptography. The next release of cryptography will remove support for Python 3.7.
  from cryptography.exceptions import InvalidSignature

PLAY [Destroy] *****************************************************************

TASK [Destroy molecule instance(s)] ********************************************
/opt/vector-role/.tox/py37-ansible210/lib/python3.7/site-packages/ansible/parsing/vault/__init__.py:44: CryptographyDeprecationWarning: Python 3.7 is no longer supported by the Python core team and support for it is deprecated in cryptography. The next release of cryptography will remove support for Python 3.7.
  from cryptography.exceptions import InvalidSignature
/opt/vector-role/.tox/py37-ansible210/lib/python3.7/site-packages/paramiko/pkey.py:82: CryptographyDeprecationWarning: TripleDES has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.TripleDES and will be removed from cryptography.hazmat.primitives.ciphers.algorithms in 48.0.0.
  "cipher": algorithms.TripleDES,
/opt/vector-role/.tox/py37-ansible210/lib/python3.7/site-packages/paramiko/transport.py:253: CryptographyDeprecationWarning: TripleDES has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.TripleDES and will be removed from cryptography.hazmat.primitives.ciphers.algorithms in 48.0.0.
  "class": algorithms.TripleDES,
changed: [localhost] => (item={'image': 'docker.io/aragast/netology:latest', 'name': 'netologytest', 'pre_build_image': True, 'privileged': True, 'workdir': '/opt/vector_role'})

TASK [Wait for instance(s) deletion to complete] *******************************
FAILED - RETRYING: Wait for instance(s) deletion to complete (300 retries left).
changed: [localhost] => (item={'started': 1, 'finished': 0, 'ansible_job_id': '352061922313.69', 'results_file': '/root/.ansible_async/352061922313.69', 'changed': True, 'failed': False, 'item': {'image': 'docker.io/aragast/netology:latest', 'name': 'netologytest', 'pre_build_image': True, 'privileged': True, 'workdir': '/opt/vector_role'}, 'ansible_loop_var': 'item'})

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

INFO     Running podman > create

PLAY [Create] ******************************************************************

TASK [get podman executable path] **********************************************
/opt/vector-role/.tox/py37-ansible210/lib/python3.7/site-packages/ansible/parsing/vault/__init__.py:44: CryptographyDeprecationWarning: Python 3.7 is no longer supported by the Python core team and support for it is deprecated in cryptography. The next release of cryptography will remove support for Python 3.7.
  from cryptography.exceptions import InvalidSignature
/opt/vector-role/.tox/py37-ansible210/lib/python3.7/site-packages/paramiko/pkey.py:82: CryptographyDeprecationWarning: TripleDES has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.TripleDES and will be removed from cryptography.hazmat.primitives.ciphers.algorithms in 48.0.0.
  "cipher": algorithms.TripleDES,
/opt/vector-role/.tox/py37-ansible210/lib/python3.7/site-packages/paramiko/transport.py:253: CryptographyDeprecationWarning: TripleDES has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.TripleDES and will be removed from cryptography.hazmat.primitives.ciphers.algorithms in 48.0.0.
  "class": algorithms.TripleDES,
ok: [localhost]

TASK [save path to executable as fact] *****************************************
ok: [localhost]

TASK [Log into a container registry] *******************************************
skipping: [localhost] => (item="netologytest registry username: None specified") 

TASK [Check presence of custom Dockerfiles] ************************************
ok: [localhost] => (item=Dockerfile: None specified)

TASK [Create Dockerfiles from image names] *************************************
skipping: [localhost] => (item="Dockerfile: None specified; Image: docker.io/aragast/netology:latest") 

TASK [Discover local Podman images] ********************************************
ok: [localhost] => (item=netologytest)

TASK [Build an Ansible compatible image] ***************************************
skipping: [localhost] => (item=docker.io/aragast/netology:latest) 

TASK [Determine the CMD directives] ********************************************
ok: [localhost] => (item="netologytest command: None specified")

TASK [Remove possible pre-existing containers] *********************************
changed: [localhost]

TASK [Discover local podman networks] ******************************************
skipping: [localhost] => (item=netologytest: None specified) 

TASK [Create podman network dedicated to this scenario] ************************
skipping: [localhost]

TASK [Create molecule instance(s)] *********************************************
changed: [localhost] => (item=netologytest)

TASK [Wait for instance(s) creation to complete] *******************************
FAILED - RETRYING: Wait for instance(s) creation to complete (300 retries left).
FAILED - RETRYING: Wait for instance(s) creation to complete (299 retries left).
FAILED - RETRYING: Wait for instance(s) creation to complete (298 retries left).
FAILED - RETRYING: Wait for instance(s) creation to complete (297 retries left).
FAILED - RETRYING: Wait for instance(s) creation to complete (296 retries left).
FAILED - RETRYING: Wait for instance(s) creation to complete (295 retries left).
FAILED - RETRYING: Wait for instance(s) creation to complete (294 retries left).
FAILED - RETRYING: Wait for instance(s) creation to complete (293 retries left).
FAILED - RETRYING: Wait for instance(s) creation to complete (292 retries left).
FAILED - RETRYING: Wait for instance(s) creation to complete (291 retries left).
FAILED - RETRYING: Wait for instance(s) creation to complete (290 retries left).
FAILED - RETRYING: Wait for instance(s) creation to complete (289 retries left).
FAILED - RETRYING: Wait for instance(s) creation to complete (288 retries left).
FAILED - RETRYING: Wait for instance(s) creation to complete (287 retries left).
changed: [localhost] => (item=netologytest)

PLAY RECAP *********************************************************************
localhost                  : ok=8    changed=3    unreachable=0    failed=0    skipped=5    rescued=0    ignored=0

INFO     Running podman > converge
/opt/vector-role/.tox/py37-ansible210/lib/python3.7/site-packages/ansible/parsing/vault/__init__.py:44: CryptographyDeprecationWarning: Python 3.7 is no longer supported by the Python core team and support for it is deprecated in cryptography. The next release of cryptography will remove support for Python 3.7.
  from cryptography.exceptions import InvalidSignature
/opt/vector-role/.tox/py37-ansible210/lib/python3.7/site-packages/paramiko/pkey.py:82: CryptographyDeprecationWarning: TripleDES has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.TripleDES and will be removed from cryptography.hazmat.primitives.ciphers.algorithms in 48.0.0.
  "cipher": algorithms.TripleDES,
/opt/vector-role/.tox/py37-ansible210/lib/python3.7/site-packages/paramiko/transport.py:253: CryptographyDeprecationWarning: TripleDES has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.TripleDES and will be removed from cryptography.hazmat.primitives.ciphers.algorithms in 48.0.0.
  "class": algorithms.TripleDES,
ERROR! couldn't resolve module/action 'ansible.builtin.systemd_service'. This often indicates a misspelling, missing collection, or incorrect module path.

The error appears to be in '/opt/vector-role/handlers/main.yml': line 2, column 3, but may
be elsewhere in the file depending on the exact syntax problem.

The offending line appears to be:

---
- name: Restart vector service
  ^ here
WARNING  Retrying execution failure 4 of: ansible-playbook --inventory /root/.cache/molecule/vector-role/podman/inventory --skip-tags molecule-notest,notest /opt/vector-role/molecule/podman/converge.yml
CRITICAL Ansible return code was 4, command was: ['ansible-playbook', '--inventory', '/root/.cache/molecule/vector-role/podman/inventory', '--skip-tags', 'molecule-notest,notest', '/opt/vector-role/molecule/podman/converge.yml']
WARNING  An error occurred during the test sequence action: 'converge'. Cleaning up.
INFO     Running podman > cleanup
WARNING  Skipping, cleanup playbook not configured.
INFO     Running podman > destroy

PLAY [Destroy] *****************************************************************

TASK [Destroy molecule instance(s)] ********************************************
/opt/vector-role/.tox/py37-ansible210/lib/python3.7/site-packages/ansible/parsing/vault/__init__.py:44: CryptographyDeprecationWarning: Python 3.7 is no longer supported by the Python core team and support for it is deprecated in cryptography. The next release of cryptography will remove support for Python 3.7.
  from cryptography.exceptions import InvalidSignature
/opt/vector-role/.tox/py37-ansible210/lib/python3.7/site-packages/paramiko/pkey.py:82: CryptographyDeprecationWarning: TripleDES has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.TripleDES and will be removed from cryptography.hazmat.primitives.ciphers.algorithms in 48.0.0.
  "cipher": algorithms.TripleDES,
/opt/vector-role/.tox/py37-ansible210/lib/python3.7/site-packages/paramiko/transport.py:253: CryptographyDeprecationWarning: TripleDES has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.TripleDES and will be removed from cryptography.hazmat.primitives.ciphers.algorithms in 48.0.0.
  "class": algorithms.TripleDES,
changed: [localhost] => (item={'image': 'docker.io/aragast/netology:latest', 'name': 'netologytest', 'pre_build_image': True, 'privileged': True, 'workdir': '/opt/vector_role'})

TASK [Wait for instance(s) deletion to complete] *******************************
changed: [localhost] => (item={'started': 1, 'finished': 0, 'ansible_job_id': '254099088494.697', 'results_file': '/root/.ansible_async/254099088494.697', 'changed': True, 'failed': False, 'item': {'image': 'docker.io/aragast/netology:latest', 'name': 'netologytest', 'pre_build_image': True, 'privileged': True, 'workdir': '/opt/vector_role'}, 'ansible_loop_var': 'item'})

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

INFO     Pruning extra files from scenario ephemeral directory
ERROR: InvocationError for command /opt/vector-role/.tox/py37-ansible210/bin/molecule test -s podman --destroy always (exited with code 1)
py37-ansible30 create: /opt/vector-role/.tox/py37-ansible30
ERROR: cowardly refusing to delete `envdir` (it does not look like a virtualenv): /opt/vector-role/.tox/py37-ansible30
__________________________________________________________________________ summary __________________________________________________________________________
ERROR:   py37-ansible210: commands failed
ERROR:   py37-ansible30: error
ERROR:   py39-ansible210: undefined
ERROR:   py39-ansible30: undefined
```