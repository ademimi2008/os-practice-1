# Практическая работа № 1. Установка Linux и знакомство с терминалом (вариант B)

Студент: _ФИО_
Дата: 29.10.2026
Система: Ubuntu 26.04.1 LTS (WSL2)

## 0. Окружение

```bash
lsb_release -a
uname -r
whoami
```

```
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 26.04.1 LTS
Release:        26.04
Codename:       resolute
6.18.33.2-microsoft-standard-WSL2
ademi
```

Ubuntu запущена в WSL2 на Windows; основной пользователь — `ademi`.

## 1. Пользователь student2 и группа developers

Пользователь и группа созданы командами:

```bash
sudo groupadd developers
sudo adduser student2
sudo usermod -aG developers student2
```

Проверка:

```bash
id student2
getent group developers
```

```
uid=1002(student2) gid=1002(student2) groups=1002(student2),100(users),1001(developers)
developers:x:1001:student2
```

Пользователь `student2` существует и входит в группу `developers`.

## 2. Каталог /srv/shared

Каталог создан командами:

```bash
sudo mkdir -p /srv/shared
sudo chown root:developers /srv/shared
sudo chmod 775 /srv/shared
ls -ld /srv/shared
```

```
drwxrwxr-x 2 root developers 4096 Sep 30 15:18 /srv/shared
```

Права `775`: владелец (root) и группа `developers` могут читать, создавать и удалять файлы (`rwx`), остальные пользователи только читают и заходят в каталог (`r-x`).

## 3. Создание и попытка удаления файла

От имени student2 создан файл:

```bash
sudo -u student2 touch /srv/shared/from_student2.txt
ls -l /srv/shared
```

```
total 4
-rw-r--r-- 1 root     root     2396 Sep 22 16:14 README.md
-rw-r--r-- 1 student2 student2    0 Oct 29 18:53 from_student2.txt
-rw-rw-r-- 1 student2 student2    0 Sep 22 13:05 student2_file.text
-rw-rw-r-- 1 student2 student2    0 Sep 30 15:18 student2_file.txt
```

(Остальные файлы в каталоге остались от предыдущих попыток.)

От имени основного пользователя `ademi` (без sudo) попытка удалить файл:

```bash
rm /srv/shared/from_student2.txt
```

```
rm: remove write-protected regular empty file '/srv/shared/from_student2.txt'? y
rm: cannot remove '/srv/shared/from_student2.txt': Permission denied
```

**Объяснение.** Удалить файл — значит изменить каталог, в котором он лежит, поэтому нужное право — `w` на каталог `/srv/shared`, а не на сам файл. Пользователь `ademi` не владелец каталога (это root) и не входит в группу `developers`, поэтому для него действуют права «остальных» — `r-x`, без записи. Итог: `Permission denied`. Вопрос `write-protected ... ?` появился потому, что у файла нет права записи для `ademi` (`-rw-r--r--`, владелец student2); ответ `y` ничего не меняет, права на каталог всё равно не позволяют удалить файл.

Замечания:
- Если добавить `ademi` в группу `developers`, он сможет удалять чужие файлы в этом каталоге, так как у группы есть `w` на каталог.
- Чтобы участники группы не могли удалять файлы друг друга, на каталог ставят sticky-бит (`chmod 1775`).
- Файл `from_student2.txt` получил группу `student2`, а не `developers`, потому что у каталога не выставлен бит setgid. С `chmod 2775` новые файлы наследовали бы группу `developers`.

## 4. Ответы на вопросы

**Чем `su` отличается от `sudo`?**

- `su` (switch user) переключает сеанс на другого пользователя (по умолчанию root) и требует пароль **целевого** пользователя. После этого все команды выполняются от его имени, пока не выполнен `exit`.
- `sudo` выполняет **одну команду** от имени другого пользователя (по умолчанию root), запрашивая пароль **текущего** пользователя. Права задаются в `/etc/sudoers`, действия попадают в журнал, доступ можно ограничить конкретными командами.

**Почему в Ubuntu по умолчанию заблокирован вход под root?**

У учётной записи root в Ubuntu нет пароля (в `/etc/shadow` стоит `!`), поэтому войти под ней напрямую нельзя. Причины:
- Нет пароля, который можно подобрать, а имя `root` известно всем (например, при подборе пароля по SSH).
- Привилегии выдаются осознанно и на время через `sudo`; постоянная работа под root чревата случайным разрушительным действием.
- Каждое административное действие связано с конкретным пользователем и попадает в журнал.
- Администрирование делегируется группе `sudo`, общий пароль root не нужен.

При необходимости root-оболочку можно получить командой `sudo -i`.

## 5. Настройка ~/.bashrc

В конец файла добавлены два алиаса и переменная окружения:

```bash
cat >> ~/.bashrc << 'EOF'
# ---Практическая работа 1 ---
alias ll='ls -alF'
alias update='sudo apt update && sudo apt list --upgradable'
export COURSE="os-practice-1"
EOF
```

```bash
tail -5 ~/.bashrc
```

```
export MY_NAME="student"
# ---Практическая работа 1 ---
alias ll='ls -alF'
alias update='sudo apt update && sudo apt list --upgradable'
export COURSE="os-practice-1"
```

Проверка в новой сессии (интерактивная оболочка заново читает `~/.bashrc`):

```bash
bash -i -c 'alias ll; alias update; echo "COURSE=$COURSE"'
```

```
alias ll='ls -alF'
alias update='sudo apt update && sudo apt list --upgradable'
COURSE=os-practice-1
```

Алиасы `ll` и `update` и переменная `COURSE` работают в новой сессии.

## Вывод

Развернута Ubuntu (WSL2), создан пользователь `student2` и группа `developers`, настроен общий каталог `/srv/shared` с правами `775`. Проверено, что пользователь вне группы не может удалять файлы в этом каталоге. Настроены два алиаса и переменная окружения в `~/.bashrc`.
