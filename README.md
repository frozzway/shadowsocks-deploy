
## Wireguard & shadowsocks proxy server

#### Предварительная подготовка

Для автоматического развертывания контейнера с shadowsocks и wireguard сервером на удаленной машине в первую очередь необходимо установить pipx и ansible согласно [документации](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-and-upgrading-ansible-with-pipx):

> Ansible не будет работать из под Windows natively. Для запуска Ansible в Windows рекомендуется использовать WSL или виртуальную машину Ubuntu.

[Инструкция по установке pipx для разных систем](https://github.com/pypa/pipx?tab=readme-ov-file#install-pipx)

Установка ansible через pipx:

```
pipx install --include-deps ansible
```

#### Настройка переменных окружения и параметров hosts

Самый простой и, по совместительству, наименее безопасный способ авторизации на удаленной машине - по имени пользователя и паролю SSH (необходима утилита `sshpass`). Предполагается, что такая попытка уже была предпринята ранее во время тестирования подключения к VM, а соответственно fingerprint VM добавлена в known-hosts. Для предоставления Ansible доступа к удаленной машине: 

- Ip адрес удаленной машины указывается в `inventories/hosts.ini` в значении `ansible_host`
- Имя пользователя, от имени которого произведется настройка машины в значении `ansible_user`
- Пароль для подключения в одном из трех полей ниже по списку

Задаем необходимые переменные окружения в `variables.yml`.

- `user`: то же имя пользователя
- `proxy_password`: пароль для подключения к shadowsocks серверу
- `local_port`: порт, на котором будет запущен сервер
- `wg_configs_directory`: директория на контрольной ноде, куда будут скопированы конфиги пиров (клиентов)
- `wg_peers_amount`: количество клиентов, которыми планируется использовать Wireguard

Остальное менять не рекомендую.


#### Запуск плейбука Ansible

```
ansible-playbook -i inventories/hosts.ini --extra-vars "@variables.yml"  deploy/setup.yml 
ansible-playbook -i inventories/hosts.ini --extra-vars "@variables.yml"  deploy/server.yml
```


#### Использование Shadowsocks
Рекомендуемый мульти-системный клиент [Nekoray](https://github.com/MatsuriDayo/nekoray). Принцип работы у других клиентов аналогичен:
1. Создать новую группу профилей для подключения или использовать Default
2. ПКМ - New profile
3. Выбираем тип Shadosocks, адрес нашей VM и порт, который указывали в `local_port`. Encryption такой же как и `method`. Password как `proxy_password`. Добавляем профиль в группу (жмём Ок)
4. Проставленная галочка на `System Proxy` выбирает режим перенаправления трафика через Proxy.
5. Осталось запустить ядро -> выделяем профиль из списка нажатием ЛКМ, жмём Enter и видим в логах сообщения о подключении.

#### Использование Wireguard
Использовать файлы `*.conf` из ваших директорий пиров (peer1, peer2, ... из директории `wg_configs_directory`) в официальном клиенте при создании профиля подключения по одному на каждое устройство.