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
