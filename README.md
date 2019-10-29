# alex-krt_infra
alex-krt Infra repository

# Домашняя работа -  ChatOps
- Добавлена интеграция GitHub со Slack
- Добавлены тесты Travis CI

# Домашняя работа - Знакомство с облачной инфраструктурой и облачными сервисами

Для подключения к хосту someinternalhost в одну команду есть классное решение, (о котором я не знал и теперь буду пользоваться регулярно)).
В каталоге ~/.ssh создаем файл config и добавляем туда следующее:
	
	host someinternalhost
		ProxyCommand ssh -W 10.128.0.2:22 alex@35.204.43.137

10.128.0.2 - ip адрес someinternalhost
35.204.43.137 - внешний ip адрес bastion

После этого подключаемся вводом команды 
	
	ssh someinternalhost

## Создание VPN сервера для серверов GCP

Pritunl установлен на bastion, подключение проверено, все работает.
Файлы setupvpn.sh и cloud-bastion.ovpn добавлены в репозиторий.
Данные для подключения:

	bastion_IP = 35.204.43.137
	someinternalhost_IP = 10.128.0.2

# Домашняя работа - Основные сервисы GCP

	testapp_IP = 35.221.186.23
	testapp_port = 9292

Создание инстанса со startup-script, который лежит в бакете.
Для начала создаем бакет:

	gsutil mb -l us-east1 gs://bucket-for-things/

Загружаем в бакет startup-script.sh

	gsutil cp ~/git/alex-krt_infra/startup_script.sh gs://bucket-for-things/

После этого создаем инстанс с нашим стартап скриптом:

	gcloud compute instances create reddit-app-with-script\
  --boot-disk-size=10GB \
  --image-family ubuntu-1604-lts \
  --image-project=ubuntu-os-cloud \
  --machine-type=g1-small \
  --tags puma-server \
  --restart-on-failure \
  --scopes storage-ro \
  --metadata startup-script-url=gs://bucket-for-things/startup_script.sh \

Создание правила фаервола:

	gcloud compute firewall-rules create puma-rule --allow tcp:9292 --target-tags=puma-server

# Домашняя работа - Модели управления инфраструктурой

Создан и протестирован шаблон виртуальной машины для Packer. 
Добавлен файл variables.json для переменных и добавлены дополнительные переменные с указанием различных параметров.

# Домашняя работа - Практика Infrastructure as a Code (IaC)

Добавлен конфиг для создания виртуальной машины с помощью terraform (main.tf) в google cloud
Работа с переменными (terraform.tfvars) и провижионерами

# Домашняя работа - Принципы организации инфраструктурного кода и работа над инфраструктурой в команде на примере Terraform

Настройка фаервола с помощью google-compute-firewall

	resource "google_compute_firewall" "firewall_ssh" {
  		name = "default-allow-ssh"
  		network = "default"
  		allow {
    	protocol = "tcp"
    	ports = ["22"]
  		}
  	source_ranges = ["0.0.0.0/0"]
	}

Работа со структурированием конфигурации.
Работа с модулями, работа с модулем storage-bucket.

# Домашняя работа - Управление конфигурацией (ansible-1)

Плейбук для клонирования репозитория:
```
---
- name: Clone
  hosts: app
  tasks:
    - name: Clone repo
      git:
        repo: https://github.com/express42/reddit.git
        dest: /home/appuser/reddit
```
После выполнения нашего плейбука в выводе есть пункт 'changed=0', т.к. репозиторий мы уже клонировали, поэтому ничего не изменилось.
После удаления директории reddit и повторного выполнения плейбука вывод:

```
PLAY [Clone] **************************************************************************

TASK [Gathering Facts] ****************************************************************
ok: [appserver]

TASK [Clone repo] *********************************************************************
changed: [appserver]

PLAY RECAP ****************************************************************************
appserver                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

Changed=1 - репозиторий склонировался заново.

# Домашняя работа - Продолжение знакомства с Ansible (ansible-2)

Работа с плейбуками, несколько сценариев в одном плейбуке, потом несколько плейбуков.
Вместо баш-скриптов в пакере теперь используем ансибл-плейбуки.
Задание со *
Для dynamic inventory использовал gcp_compute inventory plugin. Как настроить:
1. В ansible.cfg добавить строчку
```
[inventory]
enable_plugins = gcp_compute
```
2. Устанавливаем плагин google-auth
```
pip install google-auth
```
3. Создаем файл inventory.gcp.yml с таким содержанием
```
plugin: gcp_compute
projects:
  - infra-***
auth_kind: serviceaccount
service_account_file: /home/alex/bin/google-cloud-sdk/Infra-***.json
```
Проверить работоспособность можно так
```
$ ansible-inventory -i inventory.gcp.yml --graph
@all:
  |--@ungrouped:
  |  |--reddit-app
  |  |--reddit-db
```
4. Для группировки инстансов добавляем
```
groups:
  db: "'db' in name"
  app: "'app' in name"
```
Проверяем, что работает:
```
$ ansible-inventory -i inventory.gcp.yml --graph
@all:
  |--@app:
  |  |--35.233.71.135
  |--@db:
  |  |--35.195.149.196
  |--@ungrouped:
```
В ansible.cfg указываем файл *.gcp.yml 
```
inventory = ./inventory.gcp.yml
```
И проверяем - все работает :)
```

# Домашняя работа - Принципы организации кода для управления конфигурацией

Создание ролей Ansible, управление настройками нескольких окружений и best practices.

