# hostPath

Используется когда необходимо предоставить доступ 
к локальной файловой системе. Например, к логам приложений в /var/log.

Файлы на хостах доступны для записи только подам,
запущенным с правами пользователя root. Если контейнер будет запускаться
с правами другого пользователя, вы должны заранее позаботиться об установке
необходимых прав на файл или директории.

Некоторые значения параметра _type_:

- **DirectoryOrCreate**	- Если по данному пути ничего не 
существует, при необходимости будет создан пустой 
каталог с разрешением 0755, имеющим ту же группу и 
владельца, что и Kubelet.
- **Directory** - Каталог должен существовать по указанному пути.
- **FileOrCreate** - Если по указанному пути ничего не 
существует, при необходимости будет создан пустой 
файл с разрешением 0644, имеющим ту же группу и 
владельца, что и Kubelet.
- **File** - Файл должен существовать. 

## Метки на node
Поставить label на node.

`kubectl label nodes node3.k.erushnikov.ru directory=centos`

Удалит метку с node

`kubectl label nodes node3.k.erushnikov.ru directory-`

## Видео
https://youtu.be/FiAMIDYQmt0
