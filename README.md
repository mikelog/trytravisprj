# mikelog_infra
mikelog Infra repository
Самостоятельная работа:
1) для входа на someinternalhost  через bastion  по SSH в одну команду, используем

ssh -i ~/.ssh/id_rsa userName@35.210.90.82 -A -t  ssh -t 10.132.0.3

Где 35.210.90.82 ip  бастиона, а 10.132.0.3 ip  внутреннего хоста
для проброса авторизации по ключу используется ключ -A, но чтобы оно работало  ssh-add -L и ssh-add ~/.ssh/id_rsa недостачно, по крайней мере на CentOS, необходимо перед этим еще сделать eval `ssh-agent`

2) для  входа на внутренний хост по алиасу, через бастион, то есть ssh user@someinternalhost 
правим конфиг  на локальной машине vi ~/.ssh/config и приводим к такому виду

Host someinternalhost

HostName 10.132.0.3

User userName

ProxyCommand ssh userName@35.210.90.82  -W %h:%p


bastion_IP=35.210.90.82
 
someinternalhost_IP=10.132.0.3

HomeWork4
Задание *
Создание правила фаервола, с применением на инсанс:
gcloud compute firewall-rules create puma-server-port --action allow --rules tcp:9292 --target-tags puma-server --source-ranges 0.0.0.0/0

startup_script из файла через гугл СДК:
gcloud compute instances create reddit-app   --boot-disk-size=10GB   --image-family ubuntu-1604-lts   --image-project=ubuntu-os-cloud   --machine-type=g1-small   --tags puma-server   --restart-on-failure   --metadata-from-file startup-script=./startup_script.sh

startup_url:
gcloud compute instances create reddit-app   --boot-disk-size=10GB   --image-family ubuntu-1604-lts   --image-project=ubuntu-os-cloud   --machine-type=g1-small   --tags puma-server   --restart-on-failure --metadata startup-script-url=https://mydomain.com/startup-script.sh


testapp_IP=35.189.249.202

testapp_port=9292

HW.5
1. Выполнены все инструкции для создания базового шаблона для создания образа с помощью packer на базе ubuntu16.4 и ранее созданных скриптов по установке rubby и  mongoDB, задеплоено приложение и проверена его работоспособность.
2. Сделана самостоятельная работа:
добавлены  параметры в шаблон пакера:
    "project_id": null,
    "source_image_family": null,
    "machine_type": "f1-micro",
    "disk_size": "10",
    "disk_type": "pd-ssd",
    "image_description": "",
    "network": "default",
    "ssh_username": "packer",
    "tags": ""
 "project_id" и "source_image_family" обязательные, остальные заданы или могут быть переопределены  с помощью *.json файла параметров, передаваемого через директиву -var-file
*1.Создан шаблон для установки всех зависимостей и включение самого приложения в образ, с демонизацией сервиса Puma через systemd
Для проверки необходимо:
1. клонировать проект
2. перейти в каталог packer
3. выполнить packer build -var-file=./variables.json ./immutable.json
4. после создания образа скопировать его название и через параметр передать в команду
gcloud compute instances create reddit-app  --image-project=moonlit-watch-219508  --tags=puma-server  --restart-on-failure --image=IMAGE_NAME
5. после выполнения команды запомнить/скопировать внешний IP адрес и в строке браузера набрать внешний_IP:9292
*2. Создан sh скрипт для запуска инстанса из собранного образа с включенным приложением и зависимостями
для проверки:
1. запустить скрипт config-scripts/create-reddit-vm.sh дождаться его выполнения
2. после выполнения команды запомнить/скопировать внешний IP адрес и в строке браузера набрать внешний_IP:9292

HW#6
Добавления ssh-ключей нескольких пользователей в прокт была использована следующая конструкция:
resource "google_compute_project_metadata" "ssh_keys" {
 metadata {
   ssh-keys= <<EOF
    appuser:${file(var.public_key_path)}
    appuser1:${file(var.public_key_path)}
EOF
 }
}
 при terraform apply все ранее  добавленные ключи были перезаписаны тем, что  было указано в выше приведеной конструкции.
в случае использования конструкции 
resource "google_compute_project_metadata" "ssh_keys" {
 metadata {
   ssh-keys= "appuser:${file(var.public_key_path)}"
   ssh-keys= "appuser1:${file(var.public_key_path)}"
 }
}
эффект аналогичный, за исключением одного момента, в ssh ключи проекта на gcloud пропишется последний указанный ключ
То есть, в случае использования терраформ для определения доступа по ssh,  от использовая веб интерфейсе необходимо отказаться, а все ключи для проекта регисрировать через main.tf

На уровне инстанса поведение аналогичное.


HW#6 terraform 1
При дублировании описания инстанса для запуска нескольких инстансов, есть проблема в расползании конфгурации, увеличение точек отказа изза ошибок человеческого фактора, неудобочитаемость кода.
для вывода IP адресов  бекэендов в outputs.tf  надо прописать:

output "app_external_ip" {
 value = "${google_compute_instance.app.*.network_interface.0.access_config.0.assigned_nat_ip}"
}


для вывода IP балансировщика 
output "lb_ip_addr" {
 value = "${google_compute_global_forwarding_rule.default.ip_address}"
}



