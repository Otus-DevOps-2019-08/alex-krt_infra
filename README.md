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
	