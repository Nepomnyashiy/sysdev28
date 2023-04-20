# Домашнее задание к занятию «Операционные системы. Лекция 1»

### Чеклист готовности к домашнему заданию

1. Убедитесь, что у вас установлен инструмент `strace`, выполнив команду `strace -V` для проверки версии. В Ubuntu 20.04 strace установлен, но в других дистрибутивах его может не быть в коплекте «из коробки». Обратитесь к документации дистрибутива, чтобы понять, как установить инструмент strace.
2. Убедитесь, что у вас установлен пакет `bpfcc-tools`, информация по установке [по ссылке](https://github.com/iovisor/bcc/blob/master/INSTALL.md).

### Дополнительные материалы для выполнения задания

1. Изучите документацию lsof — `man lsof`. Та же информация есть [в сети](https://linux.die.net/man/8/lsof).
2. Документация по режимам работы bash находится в `help set` или [в сети](https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html).

------

## Задание

1. Какой системный вызов делает команда `cd`? 

***Ответ:***
	`chdir("/tmp")`

2. Попробуйте использовать команду `file` на объекты разных типов в файловой системе. Например:

    ```bash
    vagrant@netology1:~$ file /dev/tty
    /dev/tty: character special (5/0)
    vagrant@netology1:~$ file /dev/sda
    /dev/sda: block special (8/0)
    vagrant@netology1:~$ file /bin/bash
    /bin/bash: ELF 64-bit LSB shared object, x86-64
    ```
    
    Используя `strace`, выясните, где находится база данных `file`, на основании которой она делает свои догадки.

***Ответ:***
	`/etc/magic`

3. Предположим, приложение пишет лог в текстовый файл. Этот файл оказался удалён (deleted в lsof), но сказать сигналом приложению переоткрыть файлы или просто перезапустить приложение возможности нет. Так как приложение продолжает писать в удалённый файл, место на диске постепенно заканчивается. Основываясь на знаниях о перенаправлении потоков, предложите способ обнуления открытого удалённого файла, чтобы освободить место на файловой системе.

***Ответ:***
	Необходимо отправить пустую команду `echo` и в дескриптор процесса, который записывает файл.
	
4. Занимают ли зомби-процессы ресурсы в ОС (CPU, RAM, IO)?

***Ответ:***
	Нет. Эти процессы не потребляют ресурсов, в отличие от процессов-сирот.
	
5. В IO Visor BCC есть утилита `opensnoop`:

    ```bash
    root@vagrant:~# dpkg -L bpfcc-tools | grep sbin/opensnoop
    /usr/sbin/opensnoop-bpfcc
    ```
    
    На какие файлы вы увидели вызовы группы `open` за первую секунду работы утилиты? Воспользуйтесь пакетом `bpfcc-tools` для Ubuntu 20.04. Дополнительные сведения по установке [по ссылке](https://github.com/iovisor/bcc/blob/master/INSTALL.md).
    
***Ответ:***
    Посмотрим системные вызовы, с помщью комады:

     ```bash
    sudo strace opensnoop-bpfcc  2>&1 | grep openat
    ```
    Результат выдает в основном билиотеки Python и libc
    
<details>

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
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/sitecustomize.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/lib-dynload", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/__pycache__/apport_python_hook.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    newfstatat(AT_FDCWD, "/usr/sbin/opensnoop-bpfcc", {st_mode=S_IFREG|0755, st_size=11736, ...}, 0) = 0
    openat(AT_FDCWD, "/usr/sbin/opensnoop-bpfcc", O_RDONLY|O_CLOEXEC) = 3
    newfstatat(AT_FDCWD, "/usr/sbin/opensnoop-bpfcc", {st_mode=S_IFREG|0755, st_size=11736, ...}, 0) = 0
    readlink("/usr/sbin/opensnoop-bpfcc", 0x7ffc0ce84600, 4096) = -1 EINVAL (Недопустимый аргумент)
    readlink("/usr/sbin/opensnoop-bpfcc", 0x7ffc0ce83d80, 1023) = -1 EINVAL (Недопустимый аргумент)
    openat(AT_FDCWD, "/usr/sbin/opensnoop-bpfcc", O_RDONLY) = 3
    read(3, "ext_kfunc_header_openat.replace("..., 4096) = 3544
    openat(AT_FDCWD, "/usr/sbin", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/__future__.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/__init__.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/ctypes/__pycache__/__init__.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/types.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/lib-dynload/_ctypes.cpython-310-x86_64-linux-gnu.so", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libffi.so.8", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/struct.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/ctypes", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/ctypes/__pycache__/_endian.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/json/__pycache__/__init__.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/json", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/json/__pycache__/decoder.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/re.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/enum.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/sre_compile.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/sre_parse.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/sre_constants.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/functools.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/collections/__pycache__/__init__.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/keyword.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/operator.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/reprlib.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/copyreg.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/json/__pycache__/scanner.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/lib-dynload/_json.cpython-310-x86_64-linux-gnu.so", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/json/__pycache__/encoder.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/platform.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/subprocess.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/signal.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/threading.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/_weakrefset.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/warnings.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/contextlib.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/selectors.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/collections", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/collections/__pycache__/abc.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/libbcc.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libbcc.so.0", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libclang-cpp.so.11", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libLLVM-11.so.1", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libelf.so.1", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libbpf.so.0", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libstdc++.so.6", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libgcc_s.so.1", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/glibc-hwcaps/x86-64-v3/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/glibc-hwcaps/x86-64-v2/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/tls/x86_64/x86_64/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/tls/x86_64/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/tls/x86_64/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/tls/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/x86_64/x86_64/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/x86_64/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/x86_64/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libedit.so.2", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libtinfo.so.6", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libxml2.so.2", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libbsd.so.0", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libicuuc.so.70", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/liblzma.so.5", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libmd.so.0", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libicudata.so.70", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/table.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/multiprocessing/__pycache__/__init__.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/multiprocessing", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/multiprocessing/__pycache__/context.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/multiprocessing/__pycache__/process.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/multiprocessing/__pycache__/reduction.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/pickle.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/_compat_pickle.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/socket.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/perf.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/utils.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/traceback.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/linecache.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/tokenize.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/token.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/version.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/disassembler.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/librt.so.1", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/usdt.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/containers.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/argparse.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/gettext.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/datetime.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/locale.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/shutil.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/fnmatch.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/bz2.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/_compression.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/lib-dynload/_bz2.cpython-310-x86_64-linux-gnu.so", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libbz2.so.1.0", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/lzma.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/lib-dynload/_lzma.cpython-310-x86_64-linux-gnu.so", O_RDONLY|O_CLOEXEC) = 3
    write(2, "usage: opensnoop-bpfcc [-h] [-T]"..., 208usage: opensnoop-bpfcc [-h] [-T] [-U] [-x] [-p PID] [-t TID]
    write(2, "opensnoop-bpfcc: error: argument"..., 70opensnoop-bpfcc: error: argument -d/--duration: expected one argument
    nepom@QwackBook:~$ strace opensnoop-bpfcc -n 1 -d 2>&1 | grep openat
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
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/sitecustomize.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/lib-dynload", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/__pycache__/apport_python_hook.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/sbin/opensnoop-bpfcc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/sbin/opensnoop-bpfcc", O_RDONLY) = 3
    read(3, "ext_kfunc_header_openat.replace("..., 4096) = 3544
    openat(AT_FDCWD, "/usr/sbin", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/__future__.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/__init__.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/ctypes/__pycache__/__init__.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/types.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/lib-dynload/_ctypes.cpython-310-x86_64-linux-gnu.so", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libffi.so.8", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/struct.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/ctypes", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/ctypes/__pycache__/_endian.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/json/__pycache__/__init__.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/json", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/json/__pycache__/decoder.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/re.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/enum.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/sre_compile.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/sre_parse.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/sre_constants.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/functools.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/collections/__pycache__/__init__.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/keyword.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/operator.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/reprlib.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/copyreg.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/json/__pycache__/scanner.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/lib-dynload/_json.cpython-310-x86_64-linux-gnu.so", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/json/__pycache__/encoder.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/platform.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/subprocess.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/signal.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/threading.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/_weakrefset.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/warnings.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/contextlib.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/selectors.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/collections", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/collections/__pycache__/abc.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/libbcc.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libbcc.so.0", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libclang-cpp.so.11", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libLLVM-11.so.1", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libelf.so.1", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libbpf.so.0", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libstdc++.so.6", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libgcc_s.so.1", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/glibc-hwcaps/x86-64-v3/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/glibc-hwcaps/x86-64-v2/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/tls/x86_64/x86_64/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/tls/x86_64/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/tls/x86_64/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/tls/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/x86_64/x86_64/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/x86_64/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/x86_64/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libedit.so.2", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libtinfo.so.6", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libxml2.so.2", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libbsd.so.0", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libicuuc.so.70", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/liblzma.so.5", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libmd.so.0", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libicudata.so.70", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/table.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/multiprocessing/__pycache__/__init__.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/multiprocessing", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/multiprocessing/__pycache__/context.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/multiprocessing/__pycache__/process.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/multiprocessing/__pycache__/reduction.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/pickle.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/_compat_pickle.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/socket.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/perf.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/utils.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/traceback.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/linecache.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/tokenize.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/token.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/version.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/disassembler.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/librt.so.1", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/usdt.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/containers.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/argparse.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/gettext.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/datetime.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/locale.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/shutil.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/fnmatch.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/bz2.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/_compression.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/lib-dynload/_bz2.cpython-310-x86_64-linux-gnu.so", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libbz2.so.1.0", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/lzma.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/lib-dynload/_lzma.cpython-310-x86_64-linux-gnu.so", O_RDONLY|O_CLOEXEC) = 3
    nepom@QwackBook:~$ strace opensnoop-bpfcc -n 1 -d 2>&1 | grep openat
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
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/sitecustomize.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/lib-dynload", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/__pycache__/apport_python_hook.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/sbin/opensnoop-bpfcc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/sbin/opensnoop-bpfcc", O_RDONLY) = 3
    read(3, "ext_kfunc_header_openat.replace("..., 4096) = 3544
    openat(AT_FDCWD, "/usr/sbin", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/__future__.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/__init__.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/ctypes/__pycache__/__init__.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/types.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/lib-dynload/_ctypes.cpython-310-x86_64-linux-gnu.so", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libffi.so.8", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/struct.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/ctypes", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/ctypes/__pycache__/_endian.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/json/__pycache__/__init__.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/json", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/json/__pycache__/decoder.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/re.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/enum.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/sre_compile.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/sre_parse.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/sre_constants.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/functools.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/collections/__pycache__/__init__.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/keyword.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/operator.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/reprlib.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/copyreg.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/json/__pycache__/scanner.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/lib-dynload/_json.cpython-310-x86_64-linux-gnu.so", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/json/__pycache__/encoder.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/platform.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/subprocess.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/signal.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/threading.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/_weakrefset.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/warnings.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/contextlib.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/selectors.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/collections", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/collections/__pycache__/abc.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/libbcc.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libbcc.so.0", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libclang-cpp.so.11", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libLLVM-11.so.1", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libelf.so.1", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libbpf.so.0", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libstdc++.so.6", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libgcc_s.so.1", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/glibc-hwcaps/x86-64-v3/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/glibc-hwcaps/x86-64-v2/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/tls/x86_64/x86_64/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/tls/x86_64/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/tls/x86_64/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/tls/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/x86_64/x86_64/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/x86_64/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/x86_64/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/../lib/libedit.so.2", O_RDONLY|O_CLOEXEC) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libedit.so.2", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libtinfo.so.6", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libxml2.so.2", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libbsd.so.0", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libicuuc.so.70", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/liblzma.so.5", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libmd.so.0", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libicudata.so.70", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/table.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/multiprocessing/__pycache__/__init__.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/multiprocessing", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/multiprocessing/__pycache__/context.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/multiprocessing/__pycache__/process.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/multiprocessing/__pycache__/reduction.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/pickle.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/_compat_pickle.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/socket.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/perf.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/utils.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/traceback.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/linecache.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/tokenize.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/token.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/version.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/disassembler.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/librt.so.1", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/usdt.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3/dist-packages/bcc/__pycache__/containers.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/argparse.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/gettext.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/datetime.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/locale.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/shutil.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/fnmatch.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/bz2.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/_compression.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/lib-dynload/_bz2.cpython-310-x86_64-linux-gnu.so", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libbz2.so.1.0", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/__pycache__/lzma.cpython-310.pyc", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/python3.10/lib-dynload/_lzma.cpython-310-x86_64-linux-gnu.so", O_RDONLY|O_CLOEXEC) = 3
</details>


6. Какой системный вызов использует `uname -a`? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в `/proc` и где можно узнать версию ядра и релиз ОС.

***Ответ:***
    Команда `uname -a` использует системный вызов `uname`. Он возвращает информацию о текущей ОС и характеристиках компьютера. 


7. Чем отличается последовательность команд через `;` и через `&&` в bash? Например:

    ```bash
    root@netology1:~# test -d /tmp/some_dir; echo Hi
    Hi
    root@netology1:~# test -d /tmp/some_dir && echo Hi
    root@netology1:~#
    ```
    
    Есть ли смысл использовать в bash `&&`, если применить `set -e`?

***Ответ:***
    Последовательность команд через `;` выполнится вне зависимости от резултата первой команды. В случае с `&&` следующая команда выполниться только если предыдущая завершлась успешно.

    Использование `&&` в скрипте вместе с `set -e` не имеет смысла. Так как `set -e` завершит выполнение скрипта на любом этапе, при возникновении ошибок.


8. Из каких опций состоит режим bash `set -euxo pipefail`, и почему его хорошо было бы использовать в сценариях?

***Ответ:***
    `-e` - Завершить немедленно, при получении ненулевого статуса выполнения любой команды.
    `-u` - Несуществующие переменные будут рассматриваться как ошибки.
    `-x` - Вывод в терминал команд и их аргуиентов перед выполнением.
    `-o` - Заершить выполнение скрипта, даже если одна из частей пайпа завершилась ошибкой. 

    Данная комбинация опций команды `set`, поможет избежать выполнения скрипта с вышеуказанных ошибок. И проследить за процессом выполнения команд.

9. Используя `-o stat` для `ps`, определите, какой наиболее часто встречающийся статус у процессов в системе. В `man ps` изучите (`/PROCESS STATE CODES`), что значат дополнительные к основной заглавной букве статуса процессов. Его можно не учитывать при расчёте (считать S, Ss или Ssl равнозначными).

***Ответ:***
    Чаще всего встречающийсч статусы у процессов: `R` и `S`
    `R` - работающий или готовый к работе (в очереди на выполнение);
    `S` - прерываемый сон (ожидание завершения какого-либо события);

----
