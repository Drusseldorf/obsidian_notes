### 1. Запуск сервера на всех интерфейсах

При запуске Django-сервера в WSL используйте команду, которая заставляет сервер слушать не только localhost, но и все IP-адреса:

```bash
python manage.py runserver 0.0.0.0:8000
```

Это необходимо для того, чтобы запросы, приходящие с других устройств, могли достучаться до сервера.

### 2. Различие между WSL1 и WSL2

- **WSL1:**  
    Сеть WSL1 интегрирована с Windows, поэтому сервер, запущенный командой выше, будет доступен по IP-адресу Windows. Если ваш компьютер подключен к Wi-Fi с IP-адресом, например, `192.168.1.100`, то на других устройствах можно обращаться по адресу `http://192.168.1.100:8000`.
    
- **WSL2:**  
    В WSL2 используется виртуализированная сеть с собственным IP-адресом, который отличается от IP-адреса Windows. Это означает, что даже при запуске сервера с `0.0.0.0` он будет слушать на виртуальном интерфейсе внутри WSL2, а для внешнего доступа понадобится настроить перенаправление портов.
    

### 3. Настройка перенаправления портов для WSL2

Если вы используете WSL2, выполните следующие шаги:

1. **Определите IP-адрес WSL2:**  
    В терминале WSL выполните:
    
    ```bash
    hostname -I
    ```
    
    или
    
    ```bash
    ip addr show eth0
    ```
    
    Запишите полученный IP-адрес (например, `172.20.5.10`).
    
2. **Определите IP-адрес Wi-Fi адаптера Windows:**  
    В Windows запустите `ipconfig` и найдите IPv4-адрес для вашего Wi-Fi адаптера (например, `192.168.1.100`).
    
3. **Настройте перенаправление портов:**  
    Откройте командную строку Windows с правами администратора и выполните следующую команду, чтобы перенаправить запросы, приходящие на порт 8000 вашего Windows-хоста, на порт 8000 в WSL2:
    
    ```cmd
    netsh interface portproxy add v4tov4 listenport=8000 listenaddress=0.0.0.0 connectport=8000 connectaddress=<WSL2_IP>
    ```
    
    Замените `<WSL2_IP>` на IP-адрес, полученный в первом шаге.
    
4. **Проверьте настройки брандмауэра:**  
    Убедитесь, что Windows Firewall разрешает входящие подключения на порт 8000. При необходимости добавьте соответствующее правило.
    

### 4. Доступ к серверу

После настройки для:

- **WSL1:**  
    На других устройствах в вашей сети обращайтесь по адресу:
    
    ```
    http://<Windows_IP>:8000
    ```
    
- **WSL2:**  
    После настройки перенаправления, также используйте:
    
    ```
    http://<Windows_IP>:8000
    ```
    
    где `<Windows_IP>` – IP-адрес вашего Wi-Fi адаптера (например, `192.168.1.100`).
    

### Важные замечания

- **Изменение IP-адреса WSL2:**  
    При перезапуске WSL2 IP-адрес может измениться, поэтому в таком случае правило перенаправления портов нужно будет обновить.
- **Безопасность:**  
    Обратите внимание, что такой сервер будет доступен в локальной сети. Если сервер не предназначен для публичного доступа, убедитесь, что ваша сеть надежно защищена.

Следуя этим шагам, вы сможете запустить Django-сервер в WSL и сделать его доступным по вашей Wi-Fi сети.

Если вы запускаете Django на WSL2, используя адрес 127.0.0.1, сервер будет слушать только loopback-интерфейс внутри WSL2. Это значит, что он будет доступен **только изнутри самой WSL2-среды** и не будет виден ни для Windows-хоста, ни для других устройств в сети.

Чтобы сервер был доступен на вашей Wi-Fi сети, следует запустить его с привязкой к 0.0.0.0:

```bash
python manage.py runserver 0.0.0.0:8000
```

При этом важно помнить, что в WSL2 используется отдельная виртуальная сеть. Если вы хотите, чтобы сервер был доступен извне (например, с других устройств в вашей сети), потребуется настроить перенаправление портов на Windows с помощью команды `netsh interface portproxy`, как описывалось ранее.

Таким образом, запуск на 127.0.0.1 ограничивает сервер внутренним доступом в WSL2, а привязка к 0.0.0.0 позволяет принимать входящие подключения извне.

Далее нужно отключать firewall
![](Pasted%20image%2020250223000750.png)

Чтобы удалить правило перенаправления портов, необходимо воспользоваться командой `netsh` в командной строке с правами администратора. Вот два способа:

1. **Удаление конкретного правила:**  
    Если вы добавляли правило для порта 8000, например:
    
    ```cmd
    netsh interface portproxy add v4tov4 listenport=8000 listenaddress=0.0.0.0 connectport=8000 connectaddress=<WSL2_IP>
    ```
    
    то для его удаления выполните:
    
    ```cmd
    netsh interface portproxy delete v4tov4 listenport=8000 listenaddress=0.0.0.0
    ```
    
    Эта команда удалит правило, привязанное к указанным адресу и порту.
    
2. **Сброс всех настроек порт-прокси:**  
    Если вы хотите удалить все созданные правила, можно выполнить:
    
    ```cmd
    netsh interface portproxy reset
    ```
    

После выполнения этих команд правило(а) будет(ут) удалено(ы). Если вы не уверены в текущих настройках, можно посмотреть список активных правил с помощью:

```cmd
netsh interface portproxy show all
```

### 5. Легкий доступ через интернет

```bash
ssh -R 80:192.168.1.84:8000 serveo.net
```
указываем адрес в локальной сети, если приложение запущено в WSL2