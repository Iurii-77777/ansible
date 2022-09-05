## **Задача 1.**
#### Приготовьте свой собственный inventory файл prod.yml. 
```
iurii-devops@Host-SPB:~/ansible$ cat 8_2_playbook/inventory/prod.yml; echo ""
---
elasticsearch:
  hosts:
    ARM1:
      ansible_connection: docker
kibana:
  hosts:
    kibana:
      ansible_connection: docker
```
## **Задача 2.** 
#### Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает kibana.
```
- name: Install Kibana
  hosts: kibana
  tasks:
    - name: Upload tar.gz Kibana from remote URL
      get_url:
        url: "https://artifacts.elastic.co/downloads/kibana/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
        dest: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
        mode: 0755
        timeout: 60
        force: true
        validate_certs: false
      register: get_kibana
      until: get_kibana is succeeded
      tags: kibana
    - name: Create directrory for Kibana ({{ kibana_home }})
      file:
        path: "{{ kibana_home }}"
        state: directory
      tags: kibana
    - name: Extract Kibana in the installation directory
      #become: yes
      unarchive:
        copy: false
        src: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
        dest: "{{ kibana_home }}"
        extra_opts: [--strip-components=1]
        creates: "{{ kibana_home }}/bin/kibana"
      tags:
        - skip_ansible_lint
        - kibana
    - name: Set environment Kibana
      #become: yes
      template:
        src: templates/kib.sh.j2
        dest: /etc/profile.d/kib.sh
      tags: kibana

```
## **Задача 3.**
#### Tasks должны: скачать нужной версии дистрибутив, выполнить распаковку в выбранную директорию, сгенерировать конфигурацию с параметрами.
```
Добавил переменную в group_vars и файл kib.sh.j2 в template
```
## **Задача 4.**
#### Запустите ansible-lint site.yml и исправьте ошибки, если они есть.
```
iurii-devops@Host-SPB:~/ansible/8_2_playbook$ sudo ansible-playbook -i inventory/prod.yml site.yml
[WARNING]: Found both group and host with same name: kibana

PLAY [Install Java] **************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************
ok: [ARM1]
ok: [kibana]

TASK [Set facts for Java 11 vars] ************************************************************************************************************************
ok: [ARM1]
ok: [kibana]

TASK [Upload .tar.gz file containing binaries from local storage] ****************************************************************************************
ok: [kibana]
ok: [ARM1]

TASK [Ensure installation dir exists] ********************************************************************************************************************
ok: [kibana]
ok: [ARM1]

TASK [Extract java in the installation directory] ********************************************************************************************************
skipping: [ARM1]
skipping: [kibana]

TASK [Export environment variables] **********************************************************************************************************************
ok: [ARM1]
ok: [kibana]

PLAY [Install Elasticsearch] *****************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************
ok: [ARM1]

TASK [Upload tar.gz Elasticsearch from remote URL] *******************************************************************************************************
ok: [ARM1]

TASK [Create directrory for Elasticsearch] ***************************************************************************************************************
ok: [ARM1]

TASK [Extract Elasticsearch in the installation directory] ***********************************************************************************************
skipping: [ARM1]

TASK [Set environment Elastic] ***************************************************************************************************************************
ok: [ARM1]

PLAY [Install Kibana] ************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************
ok: [kibana]

TASK [Upload tar.gz Kibana from remote URL] **************************************************************************************************************
changed: [kibana]

TASK [Create directrory for Kibana (/opt/kibana/7.12.0)] *************************************************************************************************
changed: [kibana]

TASK [Extract Kibana in the installation directory] ******************************************************************************************************
changed: [kibana]

TASK [Set environment Kibana] ****************************************************************************************************************************
changed: [kibana]

PLAY RECAP ***********************************************************************************************************************************************
ARM1                       : ok=9    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
kibana                     : ok=10   changed=4    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   

iurii-devops@Host-SPB:~/ansible/8_2_playbook$ 

```
## **Задача 5.**
#### Попробуйте запустить playbook на этом окружении с флагом --check.
```
iurii-devops@Host-SPB:~/ansible/8_2_playbook$ sudo ansible-playbook -i inventory/prod.yml site.yml --check
[sudo] пароль для iurii-devops: 
[WARNING]: Found both group and host with same name: kibana

PLAY [Install Java] *************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************
ok: [ARM1]
ok: [kibana]

TASK [Set facts for Java 11 vars] ***********************************************************************************************************************
ok: [ARM1]
ok: [kibana]

TASK [Upload .tar.gz file containing binaries from local storage] ***************************************************************************************
ok: [ARM1]
ok: [kibana]

TASK [Ensure installation dir exists] *******************************************************************************************************************
ok: [kibana]
ok: [ARM1]

TASK [Extract java in the installation directory] *******************************************************************************************************
skipping: [ARM1]
skipping: [kibana]

TASK [Export environment variables] *********************************************************************************************************************
ok: [ARM1]
ok: [kibana]

PLAY [Install Elasticsearch] ****************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************
ok: [ARM1]

TASK [Upload tar.gz Elasticsearch from remote URL] ******************************************************************************************************
changed: [ARM1]

TASK [Create directrory for Elasticsearch] **************************************************************************************************************
ok: [ARM1]

TASK [Extract Elasticsearch in the installation directory] **********************************************************************************************
skipping: [ARM1]

TASK [Set environment Elastic] **************************************************************************************************************************
ok: [ARM1]

PLAY [Install Kibana] ***********************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************
ok: [kibana]

TASK [Upload tar.gz Kibana from remote URL] *************************************************************************************************************
changed: [kibana]

TASK [Create directrory for Kibana (/opt/kibana/7.12.0)] ************************************************************************************************
ok: [kibana]

TASK [Extract Kibana in the installation directory] *****************************************************************************************************
skipping: [kibana]

TASK [Set environment Kibana] ***************************************************************************************************************************
ok: [kibana]

PLAY RECAP **********************************************************************************************************************************************
ARM1                       : ok=9    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
kibana                     : ok=9    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   

iurii-devops@Host-SPB:~/ansible/8_2_playbook$ 
```
## **Задача 6.**
#### Запустите playbook на prod.yml окружении с флагом --diff. Убедитесь, что изменения на системе произведены.
```
iurii-devops@Host-SPB:~/ansible/8_2_playbook$ sudo ansible-playbook -i inventory/prod.yml site.yml --diff
[WARNING]: Found both group and host with same name: kibana

PLAY [Install Java] ************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************
ok: [ARM1]
ok: [kibana]

TASK [Set facts for Java 11 vars] **********************************************************************************************************************
ok: [ARM1]
ok: [kibana]

TASK [Upload .tar.gz file containing binaries from local storage] **************************************************************************************
ok: [kibana]
ok: [ARM1]

TASK [Ensure installation dir exists] ******************************************************************************************************************
ok: [ARM1]
ok: [kibana]

TASK [Extract java in the installation directory] ******************************************************************************************************
skipping: [ARM1]
skipping: [kibana]

TASK [Export environment variables] ********************************************************************************************************************
ok: [kibana]
ok: [ARM1]

PLAY [Install Elasticsearch] ***************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************
ok: [ARM1]

TASK [Upload tar.gz Elasticsearch from remote URL] *****************************************************************************************************
ok: [ARM1]

TASK [Create directrory for Elasticsearch] *************************************************************************************************************
ok: [ARM1]

TASK [Extract Elasticsearch in the installation directory] *********************************************************************************************
skipping: [ARM1]

TASK [Set environment Elastic] *************************************************************************************************************************
ok: [ARM1]

PLAY [Install Kibana] **********************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************
ok: [kibana]

TASK [Upload tar.gz Kibana from remote URL] ************************************************************************************************************
ok: [kibana]

TASK [Create directrory for Kibana (/opt/kibana/7.12.0)] ***********************************************************************************************
ok: [kibana]

TASK [Extract Kibana in the installation directory] ****************************************************************************************************
skipping: [kibana]

TASK [Set environment Kibana] **************************************************************************************************************************
ok: [kibana]

PLAY RECAP *********************************************************************************************************************************************
ARM1                       : ok=9    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
kibana                     : ok=9    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   

```
## **Задача 7.**
#### Повторно запустите playbook с флагом --diff и убедитесь, что playbook идемпотентен.
```
Вывод аналогичен прошлой команды.
```
## **Задача 8.**
#### Подготовьте README.md файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.
```
https://github.com/Iurii-77777/ansible/blob/master/8_2_playbook/info/readme.md
```
## **Задача 9.**
#### Готовый playbook выложите в свой репозиторий, в ответ предоставьте ссылку на него.
```
https://github.com/Iurii-77777/ansible/tree/master/8_2_playbook
```
