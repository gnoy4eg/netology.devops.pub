# Домашнее задание к занятию "08.06 Создание собственных modules"

## Подготовка к выполнению
1. Создайте пустой публичных репозиторий в любом своём проекте: `my_own_collection`
2. Скачайте репозиторий ansible: `git clone https://github.com/ansible/ansible.git` по любому удобному вам пути
3. Зайдите в директорию ansible: `cd ansible`
4. Создайте виртуальное окружение: `python3 -m venv venv`
5. Активируйте виртуальное окружение: `. venv/bin/activate`. Дальнейшие действия производятся только в виртуальном окружении
6. Установите зависимости `pip install -r requirements.txt`
7. Запустить настройку окружения `. hacking/env-setup`
8. Если все шаги прошли успешно - выйти из виртуального окружения `deactivate`
9. Ваше окружение настроено, для того чтобы запустить его, нужно находиться в директории `ansible` и выполнить конструкцию `. venv/bin/activate && . hacking/env-setup`  

Ответ:
<details>
    <summary>Окружение активировано</summary>

```bash
[gnoy@manjarokde-ws01 ansible]$ . venv/bin/activate && . hacking/env-setup
running egg_info
creating lib/ansible_core.egg-info
writing lib/ansible_core.egg-info/PKG-INFO
writing dependency_links to lib/ansible_core.egg-info/dependency_links.txt
writing entry points to lib/ansible_core.egg-info/entry_points.txt
writing requirements to lib/ansible_core.egg-info/requires.txt
writing top-level names to lib/ansible_core.egg-info/top_level.txt
writing manifest file 'lib/ansible_core.egg-info/SOURCES.txt'
reading manifest file 'lib/ansible_core.egg-info/SOURCES.txt'
reading manifest template 'MANIFEST.in'
warning: no files found matching 'SYMLINK_CACHE.json'
warning: no previously-included files found matching 'docs/docsite/rst_warnings'
warning: no previously-included files found matching 'docs/docsite/rst/conf.py'
warning: no previously-included files found matching 'docs/docsite/rst/index.rst'
warning: no previously-included files found matching 'docs/docsite/rst/dev_guide/index.rst'
warning: no previously-included files matching '*' found under directory 'docs/docsite/_build'
warning: no previously-included files matching '*.pyc' found under directory 'docs/docsite/_extensions'
warning: no previously-included files matching '*.pyo' found under directory 'docs/docsite/_extensions'
warning: no files found matching '*.ps1' under directory 'lib/ansible/modules/windows'
warning: no files found matching '*.yml' under directory 'lib/ansible/modules'
warning: no files found matching 'validate-modules' under directory 'test/lib/ansible_test/_util/controller/sanityvalidate-modules'
adding license file 'COPYING'
writing manifest file 'lib/ansible_core.egg-info/SOURCES.txt'
Setting up Ansible to run out of checkout...
PATH=/home/gnoy/ansible/bin:/home/gnoy/ansible/venv/bin:/home/gnoy/yandex-cloud/bin:/home/gnoy/.local/bin:/usr/local/sbin:/usrlocal/bin:/usr/bin:/usr/lib/jvm/default/bin:/usr/bin/site_perl:/usr/bin/vendor_perl:/usr/bin/core_perl:/var/lib/snapd/snapbin:/usr/local/go/bin
PYTHONPATH=/home/gnoy/ansible/test/lib:/home/gnoy/ansible/lib:/home/gnoy/ansible/test/lib:/home/gnoy/ansible/lib
MANPATH=/home/gnoy/ansible/docs/man:/usr/local/man:/usr/local/share/man:/usr/share/man:/usr/lib/jvm/default/man
Remember, you may wish to specify your host file with -i
Done!
(venv) [gnoy@manjarokde-ws01 ansible]$
```

</details>
<br>


## Основная часть

Наша цель - написать собственный module, который мы можем использовать в своей role, через playbook. Всё это должно быть собрано в виде collection и отправлено в наш репозиторий.

1. В виртуальном окружении создать новый `my_own_module.py` файл
2. Наполнить его содержимым (под спойлером) или возьмите данное наполнение из [статьи](https://docs.ansible.com/ansible/latest/dev_guide/developing_modules_general.html#creating-a-module).
    <details>
    <summary>Содержимое файла my_own_module.py</summary>

    ```python
    #!/usr/bin/python

    # Copyright: (c) 2018, Terry Jones <terry.jones@example.org>
    # GNU General Public License v3.0+ (see COPYING or https://www.gnu.org/licenses/gpl-3.0.txt)
    from __future__ import (absolute_import, division, print_function)
    __metaclass__ = type

    DOCUMENTATION = r'''
    ---
    module: my_test

    short_description: This is my test module

    # If this is part of a collection, you need to use semantic versioning,
    # i.e. the version is of the form "2.5.0" and not "2.4".
    version_added: "1.0.0"

    description: This is my longer description explaining my test module.

    options:
        name:
            description: This is the message to send to the test module.
            required: true
            type: str
        new:
            description:
                - Control to demo if the result of this module is changed or not.
                - Parameter description can be a list as well.
            required: false
            type: bool
    # Specify this value according to your collection
    # in format of namespace.collection.doc_fragment_name
    extends_documentation_fragment:
        - my_namespace.my_collection.my_doc_fragment_name

    author:
        - Your Name (@yourGitHubHandle)
    '''

    EXAMPLES = r'''
    # Pass in a message
    - name: Test with a message
      my_namespace.my_collection.my_test:
        name: hello world

    # pass in a message and have changed true
    - name: Test with a message and changed output
      my_namespace.my_collection.my_test:
        name: hello world
        new: true

    # fail the module
    - name: Test failure of the module
      my_namespace.my_collection.my_test:
        name: fail me
    '''

    RETURN = r'''
    # These are examples of possible return values, and in general should use other names for return values.
    original_message:
        description: The original name param that was passed in.
        type: str
        returned: always
        sample: 'hello world'
    message:
        description: The output message that the test module generates.
        type: str
        returned: always
        sample: 'goodbye'
    '''

    from ansible.module_utils.basic import AnsibleModule


    def run_module():
        # define available arguments/parameters a user can pass to the module
        module_args = dict(
            name=dict(type='str', required=True),
            new=dict(type='bool', required=False, default=False)
        )

        # seed the result dict in the object
        # we primarily care about changed and state
        # changed is if this module effectively modified the target
        # state will include any data that you want your module to pass back
        # for consumption, for example, in a subsequent task
        result = dict(
            changed=False,
            original_message='',
            message=''
        )

        # the AnsibleModule object will be our abstraction working with Ansible
        # this includes instantiation, a couple of common attr would be the
        # args/params passed to the execution, as well as if the module
        # supports check mode
        module = AnsibleModule(
            argument_spec=module_args,
            supports_check_mode=True
        )

        # if the user is working with this module in only check mode we do not
        # want to make any changes to the environment, just return the current
        # state with no modifications
        if module.check_mode:
            module.exit_json(**result)

        # manipulate or modify the state as needed (this is going to be the
        # part where your module will do what it needs to do)
        result['original_message'] = module.params['name']
        result['message'] = 'goodbye'

        # use whatever logic you need to determine whether or not this module
        # made any modifications to your target
        if module.params['new']:
            result['changed'] = True

        # during the execution of the module, if there is an exception or a
        # conditional state that effectively causes a failure, run
        # AnsibleModule.fail_json() to pass in the message and the result
        if module.params['name'] == 'fail me':
            module.fail_json(msg='You requested this to fail', **result)

        # in the event of a successful module execution, you will want to
        # simple AnsibleModule.exit_json(), passing the key/value results
        module.exit_json(**result)


    def main():
        run_module()


    if __name__ == '__main__':
        main()
    ```
    </details><br>

3. Заполните файл в соответствии с требованиями ansible так, чтобы он выполнял основную задачу: module должен создавать текстовый файл на удалённом хосте по пути, определённом в параметре `path`, с содержимым, определённым в параметре `content`.  
Ответ:  
Заполнил. [my_own_module.py](./src/my_own_module.py)
4. Проверьте module на исполняемость локально.  
Ответ:  
Копирую my_own_module.py в папку `/home/gnoy/ansible/lib/ansible/modules/`. Создаю тестовый [.json](./src/for-tests.json) для проверки. Тестирую:  
```bash
(venv) [gnoy@manjarokde-ws01 ansible]$ python -m ansible.modules.my_own_module ~/netology.devops.pub/homeworks/08-ansible-06-module/src/for-tests.json 

{"changed": false, "invocation": {"module_args": {"path": "/tmp/test.txt", "content": "TsSt MeSsAgE"}}}
(venv) [gnoy@manjarokde-ws01 ansible]$ cat /tmp/test.txt 
TsSt MeSsAgE
(venv) [gnoy@manjarokde-ws01 ansible]$ 

```
5. Напишите single task playbook и используйте module в нём.  
Ответ:  
[single-task-palybook.yml](./src/single-task-playbook.yml)
6. Проверьте через playbook на идемпотентность.  
Ответ:  
    <details>
    <summary>Подготовка</summary>

    ```bash
    (venv) [gnoy@manjarokde-ws01 ansible]$ pip3 install -e .
    Obtaining file:///home/gnoy/ansible
      Installing build dependencies ... done
      Checking if build backend supports build_editable ... done
      Getting requirements to build wheel ... done
      Preparing metadata (pyproject.toml) ... done
    Collecting packaging
      Downloading packaging-21.3-py3-none-any.whl (40 kB)
         ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 40.8/40.8 KB 1.0 MB/s eta 0:00:00
    Collecting PyYAML>=5.1
      Downloading PyYAML-6.0-cp310-cp310-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_12_x86_64.manylinux2010_x86_64.whl (682 kB)
         ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 682.2/682.2 KB 2.9 MB/s eta 0:00:00
    Collecting jinja2>=3.0.0
      Downloading Jinja2-3.1.2-py3-none-any.whl (133 kB)
         ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 133.1/133.1 KB 19.0 MB/s eta 0:00:00
    Collecting cryptography
      Downloading cryptography-37.0.2-cp36-abi3-manylinux_2_24_x86_64.whl (4.0 MB)
         ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 4.0/4.0 MB 15.8 MB/s eta 0:00:00
    Collecting resolvelib<0.9.0,>=0.5.3
      Downloading resolvelib-0.8.1-py2.py3-none-any.whl (16 kB)
    Collecting MarkupSafe>=2.0
      Downloading MarkupSafe-2.1.1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (25 kB)
    Collecting cffi>=1.12
      Downloading cffi-1.15.0-cp310-cp310-manylinux_2_12_x86_64.manylinux2010_x86_64.whl (446 kB)
         ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 446.3/446.3 KB 25.9 MB/s eta 0:00:00
    Collecting pyparsing!=3.0.5,>=2.0.2
      Downloading pyparsing-3.0.9-py3-none-any.whl (98 kB)
         ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 98.3/98.3 KB 19.1 MB/s eta 0:00:00
    Collecting pycparser
      Downloading pycparser-2.21-py2.py3-none-any.whl (118 kB)
         ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 118.7/118.7 KB 9.1 MB/s eta 0:00:00
    Installing collected packages: resolvelib, PyYAML, pyparsing, pycparser, MarkupSafe, packaging, jinja2, cffi, cryptography,     ansible-core
      Attempting uninstall: ansible-core
        Found existing installation: ansible-core 2.14.0.dev0
        Not uninstalling ansible-core at /home/gnoy/ansible/lib, outside environment /home/gnoy/ansible/venv
        Can't uninstall 'ansible-core'. No files were found to uninstall.
      Running setup.py develop for ansible-core
    Successfully installed MarkupSafe-2.1.1 PyYAML-6.0 ansible-core-2.14.0.dev0 cffi-1.15.0 cryptography-37.0.2 jinja2-3.1.2    packaging-21.3 pycparser-2.21 pyparsing-3.0.9 resolvelib-0.8.1
    WARNING: You are using pip version 22.0.4; however, version 22.1.2 is available.
    You should consider upgrading via the '/home/gnoy/ansible/venv/bin/python3 -m pip install --upgrade pip' command.
    ```
    </details>

    <details>
    <summary>Проверка на идемпотентность</summary>

    ```bash
    (venv) [gnoy@manjarokde-ws01 ansible]$ ansible-playbook /home/gnoy/netology.devops.pub/homeworks/08-ansible-06-module/src/single-task-playbook.yml 
    [WARNING]: You are running the development version of Ansible. You should only run Ansible from "devel" if you are modifying the    Ansible engine, or trying out features under development. This is a
    rapidly changing source of code and can become unstable at any point.
    [WARNING]: No inventory was parsed, only implicit localhost is available
    [WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

    PLAY [test my_own_module]   ***********************************************************************************************************************************   ******************************************

    TASK [Gathering Facts]  ***********************************************************************************************************************************  *********************************************
    ok: [localhost]

    TASK [run module]   ***********************************************************************************************************************************   **************************************************
    changed: [localhost]

    TASK [dump test output]     *********************************************************************************************************************************** ********************************************
    ok: [localhost] => {
        "msg": {
            "changed": true,
            "failed": false
        }
    }

    PLAY RECAP  ***********************************************************************************************************************************  *********************************************************
    localhost                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

    (venv) [gnoy@manjarokde-ws01 ansible]$ 
    (venv) [gnoy@manjarokde-ws01 ansible]$ ansible-playbook /home/gnoy/netology.devops.pub/homeworks/08-ansible-06-module/src/  single-task-playbook.yml 
    [WARNING]: You are running the development version of Ansible. You should only run Ansible from "devel" if you are modifying the    Ansible engine, or trying out features under development. This is a
    rapidly changing source of code and can become unstable at any point.
    [WARNING]: No inventory was parsed, only implicit localhost is available
    [WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

    PLAY [test my_own_module]   ***********************************************************************************************************************************   ******************************************

    TASK [Gathering Facts]  ***********************************************************************************************************************************  *********************************************
    ok: [localhost]

    TASK [run module]   ***********************************************************************************************************************************   **************************************************
    ok: [localhost]

    TASK [dump test output]     *********************************************************************************************************************************** ********************************************
    ok: [localhost] => {
        "msg": {
            "changed": false,
            "failed": false
        }
    }

    PLAY RECAP  ***********************************************************************************************************************************  *********************************************************
    localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

    (venv) [gnoy@manjarokde-ws01 ansible]$ 

    ```
    </details>
    <br>
7. Выйдите из виртуального окружения.  
Ответ:  
```bash
(venv) [gnoy@manjarokde-ws01 ansible]$ deactivate
[gnoy@manjarokde-ws01 ansible]$ 
```
8. Инициализируйте новую collection: `ansible-galaxy collection init my_own_namespace.yandex_cloud_elk`  
Ответ:  
```bash
[gnoy@manjarokde-ws01 ansible]$ ansible-galaxy collection init my_own_namespace.my_own_collection
[WARNING]: You are running the development version of Ansible. You should only run Ansible from "devel" if you are modifying the Ansible engine, or trying out features under development. This is a
rapidly changing source of code and can become unstable at any point.
- Collection my_own_namespace.my_own_collection was created successfully
[gnoy@manjarokde-ws01 ansible]$ 
```
9. В данную collection перенесите свой module в соответствующую директорию.  
Ответ:  
```bash
[gnoy@manjarokde-ws01 ansible]$ ls -lh /home/gnoy/ansible/my_own_namespace/my_own_collection/plugins/modules/
итого 4,0K
-rwxr-xr-x 1 gnoy gnoy 2,6K июн 22 23:16 my_own_module.py
```
10. Single task playbook преобразуйте в single task role и перенесите в collection. У role должны быть default всех параметров module  
Ответ:  
```yml
---
- name: run module
  my_own_module:
    path: "{{ path }}"
    content: "{{ content }}"
  register: testout
- name: dump test output
  debug:
    msg: '{{ testout }}'
```
11. Создайте playbook для использования этой role.
```yml
---
- name: Run my_own_module
  hosts: localhost
  roles:
    - single_task_role
```
12. Заполните всю документацию по collection, выложите в свой репозиторий, поставьте тег `1.0.0` на этот коммит.  
Ответ:  
[Collection 1.0.0](https://github.com/gnoy4eg/my_own_collection)
13. Создайте .tar.gz этой collection: `ansible-galaxy collection build` в корневой директории collection.  
Ответ:  
```bash
[gnoy@manjarokde-ws01 my_own_collection]$ ansible-galaxy collection build
[WARNING]: You are running the development version of Ansible. You should only run Ansible from "devel" if you are modifying the Ansible engine, or trying out features under development. This is a
rapidly changing source of code and can become unstable at any point.
Created collection for my_own_namespace.my_own_collection at /home/gnoy/ansible/my_own_namespace/my_own_collection/my_own_namespace-my_own_collection-1.0.0.tar.gz
```
14. Создайте ещё одну директорию любого наименования, перенесите туда single task playbook и архив c collection.  
Ответ:  
```bash
[gnoy@manjarokde-ws01 testDIR]$ ls -lh
итого 12K
-rw-r--r-- 1 gnoy gnoy 5,2K июн 25 18:14 my_own_namespace-my_own_collection-1.0.0.tar.gz
-rw-r--r-- 1 gnoy gnoy  261 июн 25 18:16 single-task-playbook.yml
[gnoy@manjarokde-ws01 testDIR]$ 
```
15. Установите collection из локального архива: `ansible-galaxy collection install <archivename>.tar.gz`  
Ответ:  
```bash
[gnoy@manjarokde-ws01 testDIR]$ ansible-galaxy collection install my_own_namespace-my_own_collection-1.0.0.tar.gz 
[WARNING]: You are running the development version of Ansible. You should only run Ansible from "devel" if you are modifying the Ansible engine, or trying out features under development. This is a
rapidly changing source of code and can become unstable at any point.
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Installing 'my_own_namespace.my_own_collection:1.0.0' to '/home/gnoy/.ansible/collections/ansible_collections/my_own_namespace/my_own_collection'
my_own_namespace.my_own_collection:1.0.0 was installed successfully
```
16. Запустите playbook, убедитесь, что он работает.  
Ответ:  
    <details>
    <summary>Проверка</summary>

    ```bash
    [gnoy@manjarokde-ws01 testDIR]$ ansible-playbook single-task-playbook.yml 
    [WARNING]: You are running the development version of Ansible. You should only run Ansible from "devel" if you are modifying the    Ansible engine, or trying out features under development. This is a
    rapidly changing source of code and can become unstable at any point.
    [WARNING]: No inventory was parsed, only implicit localhost is available
    [WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

    PLAY [test my_own_module]   ***********************************************************************************************************************************   ******************************************

    TASK [Gathering Facts]  ***********************************************************************************************************************************  *********************************************
    ok: [localhost]

    TASK [run module]   ***********************************************************************************************************************************   **************************************************
    ok: [localhost]

    TASK [dump test output]     *********************************************************************************************************************************** ********************************************
    ok: [localhost] => {
        "msg": {
            "changed": false,
            "failed": false
        }
    }

    PLAY RECAP  ***********************************************************************************************************************************  *********************************************************
    localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
    ```
    </details>
    <br>

17. В ответ необходимо прислать ссылку на репозиторий с collection    
Ответ:  
[Collection 1.0.0](https://github.com/gnoy4eg/my_own_collection)

## Необязательная часть

1. Реализуйте свой собственный модуль для создания хостов в Yandex Cloud.
2. Модуль может (и должен) иметь зависимость от `yc`, основной функционал: создание ВМ с нужным сайзингом на основе нужной ОС. Дополнительные модули по созданию кластеров Clickhouse, MySQL и прочего реализовывать не надо, достаточно простейшего создания ВМ.
3. Модуль может формировать динамическое inventory, но данная часть не является обязательной, достаточно, чтобы он делал хосты с указанной спецификацией в YAML.
4. Протестируйте модуль на идемпотентность, исполнимость. При успехе - добавьте данный модуль в свою коллекцию.
5. Измените playbook так, чтобы он умел создавать инфраструктуру под inventory, а после устанавливал весь ваш стек Observability на нужные хосты и настраивал его.
6. В итоге, ваша коллекция обязательно должна содержать: clickhouse-role(если есть своя), lighthouse-role, vector-role, два модуля: my_own_module и модуль управления Yandex Cloud хостами и playbook, который демонстрирует создание Observability стека.  

Ответ:  
Пока не брался :(
