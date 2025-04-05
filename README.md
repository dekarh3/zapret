# Быстрая настройка Linux/OpenWrt

> [!CAUTION]  
> Инструкция установки [проекта](https://github.com/bol-van/zapret) на 
> роутеры Xiaomi AX3000T и AX3200 (Redmi R6S) под OpenWRT
>  


## Вступление
Это переработанная версия [инструкции](https://github.com/bol-van/zapret/blob/master/docs/quick_start.md)
Если не получилось, то для настройки придётся читать оригинал и погружаться в в простыню [readme.md](https://github.com/bol-van/zapret/readme.md).

Обход DPI является хакерской методикой. Под этим словом понимается метод,
которому сопротивляется окружающая среда, которому автоматически не
гарантирована работоспособность в любых условиях и на любых ресурсах, требуется
настройка под специфические условия у вашего провайдера. Условия могут меняться
со временем, и методика может начинать или переставать работать, может
потребоваться повторный анализ ситуации. Могут обнаруживаться отдельные
ресурсы, которые заблокированы иначе, и которые не работают или перестали
работать. Могут и сломаться отдельные не заблокированные ресурсы. Поэтому очень
желательно иметь знания в области сетей, чтобы иметь возможность
проанализировать техническую ситуацию. Не будет лишним иметь обходные каналы
проксирования трафика на случай, если обход DPI не помогает.

Вариант, когда вы нашли стратегию где-то в интернете и пытаетесь ее приспособить к своему случаю - заведомо проблемный.
Нет универсальной таблетки. Везде ситуация разная. В сети гуляют написанные кем-то откровенные глупости, которые тиражируются массово ничего не понимающей публикой.
Такие варианты чаще всего работают нестабильно, только на части ресурсов, только на части провайдеров, не работают вообще или ломают другие ресурсы. В худших случаях еще и устраивают флуд в сети.
Если даже вариант когда-то и работал неплохо, завтра он может перестать, а в сети останется устаревшая информация.

Инструкция раасчитана на то, что на роутере установлена система на базе **openwrt**. Задача - обойти блокировки для подключенных устройств.

## Настройка

1. Скачайте последний [tar.gz релиз, вариант openwrt-embedded](https://github.com/bol-van/zapret/releases) в /tmp, распакуйте его, затем удалите архив.
   Для экономия места в /tmp можно качать через curl в stdout и сразу распаковывать. Я качал на комп linux и пользовался mc -> панель -> shell соединение

2. Убедитесь, что у вас отключены все средства обхода блокировок, в том числе и
   сам zapret. Гарантированно уберет zapret скрипт `uninstall_easy.sh`.

3. Выполните однократные действия по установке требуемых пакетов в ОС и
   настройке исполняемых файлов правильной архитектуры:
   ```sh
   $ install_bin.sh
   $ install_prereq.sh
   ```

   > Вас могут спросить о типе фаервола (iptables/nftables) и использовании
   > ipv6. Это нужно для установки правильных пакетов в ОС, чтобы не
   > устанавливать лишнее. *Я выбирал ответы, которые предлагаются по-умолчанию*

4. Скопируйте из данного репозитория или создайте файлы
- `/opt/zapret/ipset/youtube.txt`
```
play.google.ru
play.google.com
account.youtube.com
youtube.com
www.youtube.com
i.ytimg.com
studio.youtube.com
googlevideo.com
yt3.ggpht.com
yt4.ggpht.com
youtu.be
yt.be
ytimg.com
ggpht.com
gvt1.com
youtube.googleapis.com
youtubeembeddedplayer.googleapis.com
ytimg.l.google.com
nhacmp3youtube.com
jnn-pa.googleapis.com
youtube-nocookie.com
youtube-ui.l.google.com
yt-video-upload.l.google.com
wide-youtube.l.google.com
```
- `/opt/zapret/ipset/zapret-hosts-user.txt`
```
chat.signal.org
cdn2.signal.org
storage.signal.org
sfu.voip.signal.org
updates2.signal.org
uptime.signal.org

pornhub.com
rt.pornhub.com
ei.phncdn.com
phncdn.com
ew.phncdn.com
ss.phncdn.com
cdn1d-static-shared.phncdn.com
```
5. Добавьте в `/etc/hosts`
```
104.244.42.13 twitter.com www.twitter.com x.com
157.240.245.174 instagram.com www.instagram.com
```

6. Запустите скрипт облегченной установки - `install_easy.sh` 
   - Выберите `перенести в директорию /opt/zapret`
   - Выберите `nfqws`, 
   - Cогласитесь на редактирование параметров. Откроется
   редактор vi, перейдите в режим вставки (i) впишите (скопипастьте):
   ```
   NFQWS_ENABLE=1
   NFQWS_OPT="
   --filter-tcp=80 --dpi-desync=fake --dpi-desync-ttl=7 --dpi-desync-fake-http=0x00000000 <HOSTLIST> --new
   --filter-tcp=443 --hostlist=/opt/zapret/ipset/youtube.txt --dpi-desync=fake,split --dpi-desync-ttl=4 --dpi-desync-fooling=badseq --dpi-desync-fake-tls=/opt/zapret/files/fake/tls_clienthello_www_google_com.bin --dpi-desync-repeats=3 --dpi-desync-split-pos=1 --new
   --filter-tcp=443 --dpi-desync=fake,disorder2 --dpi-desync-ttl=4 --dpi-desync-fooling=badsum <HOSTLIST> --new
   --filter-udp=443 --dpi-desync=fake --dpi-desync-repeats=6 <HOSTLIST_NOAUTO>
   "
   MODE_FILTER=hostlist
   ```
   сохраните изменения (ESC : wq)

7. На все остальные вопросы `install_easy.sh` отвечайте согласно выводимой
   аннотации. *Я выбирал ответы, которые предлагаются по-умолчанию*

8. Выполнитe 
```
/opt/zapret/ipset/get_reestr_hostlist.sh 
/opt/zapret/ipset/get_antizapret_domains.sh.
```
9. Удалите директорию из /tmp, откуда производилась установка.

## Полное удаление

1. Прогоните `/opt/zapret/uninstall_easy.sh`.
2. Cогласитесь на удаление зависимостей в openwrt.
3. Удалите каталог `/opt/zapret`.

## Итог
Это минимальная инструкция, чтобы быстро сориентироваться с чего начать.
Однако, это не гарантированное решение и в некоторых случаях вы не обойдетесь
без знаний и основного "талмуда". Подробности и полное техническое описание
расписаны в [readme](https://github.com/bol-van/zapret/readme.md).

Если ломаются отдельные **не заблокированные** ресурсы, следует вносить их в
исключения, либо пользоваться ограничивающим `ipset` или хост листом. Лучше
подбирать такие стратегии, которые вызывают минимальные поломки. Есть стратегии
довольно безобидные, а есть сильно ломающие, которые подходят только для
точечного пробития отдельных ресурсов, когда ничего лучше нет. Хорошая
стратегия может сильно ломать из-за плохо подобранных ограничителей для фейков
\- ttl, fooling.
