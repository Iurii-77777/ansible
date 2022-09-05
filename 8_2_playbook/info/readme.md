# 8.2 описание Playbook

## папка GROUP_VARS
java_oracle_jdk_package - имя пакета установки Java
java_jdk_version - используемая версия Java

elastic_home - переменная домашнего каталога для Elasticsearch
kibana_home - переменная для домашнего каталога для Kibana

elastic_version - версия Elasticsearch
kibana_version - версия Kibana

## Описание Play 

### Java
 установлены тэги java для дальнейшего использования и отладки 
 - Имя "Set facts for Java 11 vars". Устанавливаем факты хоста
 - Имя "Upload .tar.gz file containing binaries from local storage" загрузка установосного пакета
 - Имя "Ensure installation dir exists" создние рабочего каталога
 - Имя "Extract java in the installation directory" распаковка установочника в рабочий каталог
 - Имя "Export environment variables" создание по шаблону переменных окружений (папка templates)

### Elastic
 установлены тэги *elastic* для дальнейшего использования и отладки 
 - Имя "Upload tar.gz Elasticsearch from remote URL" загрузка ПО
 - Имя "Create directrory for Elasticsearch" создание каталога
 - Имя "Create directrory for Elasticsearch" распаковка
 - Имя "Set environment Elastic" создание по шаблону переменных окружений (папка templates)

### Kibana
Аналогично прошлой задачи, только для kibana
