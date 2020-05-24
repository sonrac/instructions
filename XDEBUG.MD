# Установка и настройка XDebug

## Ubuntu
* Установить командой

```
sudo apt install php-xdebug
```

Обычно хdebug поставляется как метапакет и при установке просто пересобирается под конкретную версию php с помощью утилиты phpize.

Либо через pear

```
sudo pecl install xdebug
```

* Отредактировать настройки в  `mods-available/20-xdebug.ini`. 

Настройки чаще всего находятся по пути 

`/etc/php/` 

или 

`/etc/php/<version>/`

Внутрь вставьте следующее содержимое:
 
```
[xdebug]
; Модуль xdebug (или полный путь к нему)
zend_extension=xdebug.so
; Ключ сессии IDE, которая слушает xdebug
xdebug.idekey = PHPSTORM
;Автоматическое включение xdebug (на prod режиме создает существенный оверхед)
xdebug.remote_autostart = On
;Разрешение удаленного подключения    
xdebug.remote_enable = On
; По необходимости можно ограничить подключения с определенного хоста
; xdebug.remote_host = test.host 
xdebug.trace_output_dir = /tmp/xdebug ;Директория к трейс-логам xdebug
xdebug.var_display_max_children = 128
xdebug.var_display_max_data = 512
xdebug.var_display_max_depth = 3
```

* Настроить свою IDE для работы с XDebug (например, мануал для PHPStorm доступен в офф документации https://www.jetbrains.com/help/phpstorm/2016.3/configuring-xdebug.html )

## Установка под macOS

### Установка под brew: brew install php70-xdebug

Конфигурационные файлы лежат в директории /usr/local/etc/php или /usr/local/etc/php/<version>

Остальное все так же как и под Ubuntu.

### Если вы используете докер или vagrant
 
 * Вам необходимо настроить удаленный debug. Для этого добавьте директиву

```
xdebug.remote_host = IP_ADDRES_VM
```

После всех настроек не забудьте перезапустить сервер (если php не запущен отдельным демоном) или php-fpm.

Приятного использования, дебага и профилирования ;-)
