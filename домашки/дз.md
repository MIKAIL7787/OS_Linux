# Скрипты на Bash

```console
vboxuser@4vboxuser:~> ./myscript
bash: ./myscript: Permission denied
vboxuser@4vboxuser:~> chmod +x ./myscript
vboxuser@4vboxuser:~> ./myscript
/home/ali
ali
```

```bash
#!/bin/bash
IFS=:
for folder in \$PATH
do
    echo "\$folder:"
    for file in \$folder/*
    do
        if [ -x "\$file" ]
        then
            echo "    \$file"
        fi
    done
done
```
