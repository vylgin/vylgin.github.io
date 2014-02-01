---
layout: post
title: "Создание блога на github с помощью octopress"
date: 2014-01-09 15:13:50 +0400
comments: true
categories: [octopress, github]
---

Octopress — это фреймворк, позволяющий генерировать статические HTML сайты с помощью Jekyll. Каждый, кто начинает вести свой блог на нем, пишет подобную статью, это уже стало традицией, поэтому я опущу то, как установить ruby, rvm, завести акканут на гитхабе, настроить ssh-ключи и т.д., а акцентирую внимание на более интересных деталях.

<!-- more -->

### Цель
Моя цель - создать собственный блог с минимальными временными и материальными вложениями на своем домене (http://blog.vylgin.pro). Можно завести блог бесплатно, но он будет доступен по адресу http://username.github.io, где *username* - это имя пользователя на гитхабе.

### Установка Octopress+GitHub Pages
Первым делом необходимо установить сам octopress, клонировав репозиторий в каталог, где будут находиться исходные файлы:
	git clone git://github.com/imathis/octopress.git octopress
	cd octopress
	bundle install
В этот момент возникла ошибка о том, что не установлен пакет fast-stemmer версии 1.0.1, в описании ошибки было указано, как установить необходимую версию.
	sudo gem install fast-stemmer -v '1.0.1'
	bundle install
Если ошибки не было, то выполняем следующие команды:
	rake install
	rake setup_github_pages
В этом моменте надо будет указать адрес репозитория на github, например: git@github.com:*username*/*username*.github.io.git

Блог создался, теперь можно сгененировать его из исходников, запустить локально (по-умолчанию он будет доступен по адресу http://localhost:4000) и задеплоить на гитхаб:
	rake generate
	rake preview	
	rake deploy
	
### Привязка собственного домена
Гитхаб можно считать полноценным хостингом для статических сайтов, привязав собственный домен согласно [документации](https://help.github.com/articles/setting-up-a-custom-domain-with-pages). Для этого нужно:

1. Создать файл *CNAME* в каталоге *source* и записать в него имя домена, по которому будет доступен блог.
2. Настроить DNS:
    * Добавить запись **A** с именем хоста **github.map.fastly.net** и IP-адресом **199.27.76.133**
    * Добавить запись **CNAME** с именем ***username*.github.io.** и значением **github.map.fastly.net.** (точки в конце обязательны)
    * Добавить запись **CNAME** с именем **blog** и значением ***username*.github.io.** (точки в конце обязательны, blog - имя моего поддомена)
    * Пример моих настроек:

{% img center https://cloclo1.datacloudmail.ru/weblink/view/7c38a7b367de/blog/images/post_1/img1.png %}

Теперь мой блог доступен по адресу http://blog.vylgin.pro.
	
### Настройка Octopress
#### Изменение названия и описание блога
Для изменения названия и описания можно отредактировать следующие строки в файле _config.yml:
	url: http://username.github.io
	title: Название блога
	subtitle: Подзаголовок блога
	author: Имя автора
	simple_search: http://google.com/search
	description: Описание блога

#### Русификация
Подробно о русификации octopress описано в [этой статье](http://zenbro.github.io/blog/2013/09/21/octopress-russification/).

### Добавление статей
Новая статья добавляется командой:
	rake new_post["Название новой статьи"]

#### Превью статьи на главной странице
Если статья длинная, то можно добавить кнопку "Читать дальше" после абзаца, для этого нужно изменить строку в файле _config.yml, указав текст кнопки:
	excerpt_link: "Читать дальше &rarr;" 
В файле статьи, после абзаца нужно добавить строку:
	<!-- more -->
	
### Сохранение исходных файлов на github
Удобно хранить на гитхабе не только сгенерированный статический сайт, но и исходные фалы статей и настроек, чтобы иметь возможность редактировать блог на других компьютерах, да и просто иметь бэкап на всякий случай.
Нужно закоммитить изменения, с добавлением новых файлов (ключ -a) и отправить изменения в центральный репозиторий:
    git commit -a
    git push origin source
    
### Продолжение редактирования блога с другого компьютера
После того, как все исходные файлы блога стали храниться на гитхабе, можно клонировать репозиторий на другой компьютер, для этого достаточно выполнить:
    git clone git@github.com:*username*/*username*.github.io.git
Если нет доступа к гитхабу по ssh: 
    git clone https://github.com/*username*/*username*.github.io.git
Мы находимся в ветке *master*, в которой содержатся сгенерированные файлы блога, чтобы иметь возможность редактировать, необходимо переключиться в ветку *source*:
    git checkout source
Нужно привязать локальный репозиторий к центральному на гитхабе, для этого нужно выполнить команду, указав адрес репозитория (git@github.com:*username*/*username*.github.io.git):
    rake setup_github_pages
Теперь можно можно обновлять блог командой:
    rake deploy
В этот момент может возникнуть ошибка:
    Pushing generated _deploy website
    To git@github.com:vylgin/vylgin.github.io.git
     ! [rejected]        master -> master (non-fast-forward)
    error: failed to push some refs to 'git@github.com:vylgin/vylgin.github.io.git'
    hint: Updates were rejected because the tip of your current branch is behind
    hint: its remote counterpart. Merge the remote changes (e.g. 'git pull')
    hint: before pushing again.
    hint: See the 'Note about fast-forwards' in 'git push --help' for details.
Она возникает из-за того, что отличаются файлы в ветке центральной ветке *master* и локальном каталоге *_deploy*. Для исправления нужно перейти в каталог *_deploy*:
    cd _deploy
Забрать и слить удаленную ветку *master* с локальной:
    git pull origin master
Теперь нужно разрешить конфликты, в интернете полно информации, как это сделать, лично я использовал программу git-cola и meld. После того, как конфликты разрешены, можно сделать коммит и отправить изменения в центральный репозиторий:
    git commit -m "Исправлены конфликты"
    git push origin master
Конфликты разрешены, можно перейти в каталог с блогом:
    cd ..
И воспользоваться командой:
    rake deploy
После внесения изменений нужно не забыть сделать коммит и залить изменения на гитхаб:
    git commit -a
    git push origin source
    
### Возможные ошибки
У меня возникли проблемы с rake, при вызове команд появлялась ошибка:
	/usr/local/bin/rake:23:in `load': cannot load such file -- /usr/share/rubygems-integration/1.9.1/gems/rake-10.0.4/bin/rake (LoadError)
		from /usr/local/bin/rake:23:in `<main>'
Быстрым решением является вызов команды:
	bundle exec rake имя_команды

