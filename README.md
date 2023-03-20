# Домашнее задание к занятию «Операционные системы. Лекция 1»

### Цель задания

В результате выполнения задания вы:

* познакомитесь с инструментом strace, который помогает отслеживать системные вызовы процессов и является необходимым для отладки и расследований при возникновении ошибок в работе программ;
* рассмотрите различные режимы работы скриптов, настраиваемые командой set. Один и тот же код в скриптах в разных режимах работы ведёт себя по-разному.

### Чеклист готовности к домашнему заданию

1. Убедитесь, что у вас установлен инструмент `strace`, выполнив команду `strace -V` для проверки версии. В Ubuntu 20.04 strace установлен, но в других дистрибутивах его может не быть в коплекте «из коробки». Обратитесь к документации дистрибутива, чтобы понять, как установить инструмент strace.
2. Убедитесь, что у вас установлен пакет `bpfcc-tools`, информация по установке [по ссылке](https://github.com/iovisor/bcc/blob/master/INSTALL.md).

### Дополнительные материалы для выполнения задания

1. Изучите документацию lsof — `man lsof`. Та же информация есть [в сети](https://linux.die.net/man/8/lsof).
2. Документация по режимам работы bash находится в `help set` или [в сети](https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html).

------

## Задание

1. Какой системный вызов делает команда `cd`? 
    В прошлом ДЗ вы выяснили, что `cd` не является самостоятельной  программой. Это `shell builtin`, поэтому запустить `strace` непосредственно на `cd` не получится. Вы можете запустить `strace` на `/bin/bash -c 'cd /tmp'`. В этом случае увидите полный список системных вызовов, которые делает сам `bash` при старте. 

    Вам нужно найти тот единственный, который относится именно к `cd`. Обратите внимание, что `strace` выдаёт результат своей работы в поток stderr, а не в stdout.

```
chdir("/tmp") 
```

1. Попробуйте использовать команду `file` на объекты разных типов в файловой системе. Например:

    ```bash
    vagrant@netology1:~$ file /dev/tty
    /dev/tty: character special (5/0)
    vagrant@netology1:~$ file /dev/sda
    /dev/sda: block special (8/0)
    vagrant@netology1:~$ file /bin/bash
    /bin/bash: ELF 64-bit LSB shared object, x86-64
    ```
    
    Используя `strace`, выясните, где находится база данных `file`, на основании которой она делает свои догадки.
```
команда file читает из файла начальный идентификатор (магическая последовательность) и сверяет его со своей базой расположенной по пути
/usr/share/misc/magic.mgc
```

1. Предположим, приложение пишет лог в текстовый файл. Этот файл оказался удалён (deleted в lsof), но сказать сигналом приложению переоткрыть файлы или просто перезапустить приложение возможности нет. Так как приложение продолжает писать в удалённый файл, место на диске постепенно заканчивается. Основываясь на знаниях о перенаправлении потоков, предложите способ обнуления открытого удалённого файла, чтобы освободить место на файловой системе.
```
с помощью ps aux  выясняем pid процесса который пишет в лог
lsof -p pid процесса - выводим все открытые файлы находим который со статусом deleted
и echo > /proc/pid_процесса/fd/номер_файлового_дескриптора_лога
```
1. Занимают ли зомби-процессы ресурсы в ОС (CPU, RAM, IO)?
```
нет т.к процессы по факту завершены но по каким то прчинам не обработан вызов wait родительским процессом и в списке процессов остаются пока init их не подберет
опасность таких процессов существует т.к есть лимит кол-ва процессов на пользователя.
```
1. В IO Visor BCC есть утилита `opensnoop`:

    ```bash
    root@vagrant:~# dpkg -L bpfcc-tools | grep sbin/opensnoop
    /usr/sbin/opensnoop-bpfcc
    ```
    
    На какие файлы вы увидели вызовы группы `open` за первую секунду работы утилиты? Воспользуйтесь пакетом `bpfcc-tools` для Ubuntu 20.04. Дополнительные сведения по установке [по ссылке](https://github.com/iovisor/bcc/blob/master/INSTALL.md).
```
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libm.so.6", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libexpat.so.1", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libz.so.1", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache", O_RDONLY) = 3
openat(AT_FDCWD, "/usr/bin/pyvenv.cfg", O_RDONLY) = -1 ENOENT (Нет такого файла или каталога)
openat(AT_FDCWD, "/usr/pyvenv.cfg", O_RDONLY) = -1 ENOENT (Нет такого файла или каталога)
openat(AT_FDCWD, "/usr/bin/pybuilddir.txt", O_RDONLY) = -1 ENOENT (Нет такого файла или каталога)
openat(AT_FDCWD, "/etc/localtime", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/python3.10", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
openat(AT_FDCWD, "/usr/lib/python3.10/encodings/__pycache__/__init__.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/codecs.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/python3.10/encodings", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
openat(AT_FDCWD, "/usr/lib/python3.10/encodings/__pycache__/aliases.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/python3.10/encodings/__pycache__/utf_8.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/io.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/abc.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/site.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/os.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/stat.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/_collections_abc.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/posixpath.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/genericpath.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/_sitebuiltins.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/local/lib/python3.10/dist-packages", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
openat(AT_FDCWD, "/usr/lib/python3/dist-packages", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
openat(AT_FDCWD, "/usr/lib/python3.10/dist-packages", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/sitecustomize.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/python3.10/lib-dynload", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
openat(AT_FDCWD, "/usr/local/lib/python3.10/dist-packages", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
openat(AT_FDCWD, "/usr/lib/python3/dist-packages", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
openat(AT_FDCWD, "/usr/lib/python3/dist-packages/__pycache__/apport_python_hook.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/python3.10/dist-packages", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
```


1. Какой системный вызов использует `uname -a`? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в `/proc` и где можно узнать версию ядра и релиз ОС.
```
Используется системный вызов uname 
man 2 uname
Part of the utsname information is also accessible via /proc/sys/kernel/{ostype, hostname, osrelease, version, domainname}.
```
1. Чем отличается последовательность команд через `;` и через `&&` в bash? Например:

    ```bash
    root@netology1:~# test -d /tmp/some_dir; echo Hi
    Hi
    root@netology1:~# test -d /tmp/some_dir && echo Hi
    root@netology1:~#
    ```
    
    Есть ли смысл использовать в bash `&&`, если применить `set -e`?
```
разделение команд с помощью ; - безусловное выполнение всех команд разделенных ;
&& логические оператор и т.е вторая команда будет выполнена при условии что первая выполнена успешно
имеет
set -e останавливает команды при ненулевом результате и при использовании && ничего не изменится
```

1. Из каких опций состоит режим bash `set -euxo pipefail`, и почему его хорошо было бы использовать в сценариях?
-e останов скрипта при ошибке выполнения команды
-u cчитать неустановленные переменные ошибкой при подстановке
-x выводить команды перед выполенением
-o pipefail устанавливает код выхода конвейера на код выхода самой правой команды для выхода с ненулевым статусом или на ноль, если все команды конвейера завершаются успешно
Для отладки использовать хорошо.
```

1. Используя `-o stat` для `ps`, определите, какой наиболее часто встречающийся статус у процессов в системе. В `man ps` изучите (`/PROCESS STATE CODES`), что значат дополнительные к основной заглавной букве статуса процессов. Его можно не учитывать при расчёте (считать S, Ss или Ssl равнозначными).
```
-o stat пользовательский формат вывода в данном случае выводит только статусы процессов наиболее часто встречающиеся статусы начинают с S
s - лидер сессии
l - многопоточный
< - высокий приоритет
N - низкий приоритет
L - страницы закреплены в памяти
+ - процесс переднего плана ассоциирован обычно с терминалом


----

### Правила приёма домашнего задания

В личном кабинете отправлена ссылка на .md-файл в вашем репозитории.


### Критерии оценки

Зачёт:

* выполнены все задания;
* ответы даны в развёрнутой форме;
* приложены соответствующие скриншоты и файлы проекта;
* в выполненных заданиях нет противоречий и нарушения логики.

На доработку:

* задание выполнено частично или не выполнено вообще;
* в логике выполнения заданий есть противоречия и существенные недостатки.  
