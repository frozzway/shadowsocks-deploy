
## Shadowsocks proxy server

#### Предварительная подготовка

Для автоматического развертывания контейнера с shadowsocks сервером на удаленной машине в первую очередь необходимо установить pipx и ansible согласно [документации](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-and-upgrading-ansible-with-pipx):

[Инструкция по установке pipx для разных систем](https://github.com/pypa/pipx?tab=readme-ov-file#install-pipx)

Установка ansible через pipx:

```
pipx install --include-deps ansible
```

#### Настройка переменных окружения и параметров hosts

Самый простой и, по совместительству, наименее безопасный способ авторизации на удаленной машине - по имени пользователя и паролю SSH. Предполагается, что такая попытка уже была предпринята ранее во время тестирования подключения к VM. Для предоставления Ansible доступа к удаленной машине: 

- Ip адрес удаленной машины указывается в `inventories/hosts.ini` в значении `ansible_host`
- Имя пользователя, от имени которого произведется настройка машины в значении `ansible_user`
- Пароль для подключения в одном из трех полей ниже по списку

Задаем необходимые переменные окружения в `variables.yml`.

- `user`: то же имя пользователя
- `proxy_password`: пароль для подключения к shadowsocks серверу
- `local_port`: порт, на котором будет запущен сервер

Остальное менять не рекомендую.


#### Запуск плейбука Ansible

```
ansible-playbook -i inventories/hosts.ini --extra-vars "@variables.yml"  deploy/setup.yml 
ansible-playbook -i inventories/hosts.ini --extra-vars "@variables.yml"  deploy/server.yml
```