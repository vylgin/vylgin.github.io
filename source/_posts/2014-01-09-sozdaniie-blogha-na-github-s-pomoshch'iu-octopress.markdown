---
layout: post
title: "Создание блога на github с помощью octopress"
date: 2014-01-09 15:13:50 +0400
comments: true
categories: [octopress, github]
---

Octopress — это фреймворк, позволяющий генерировать статические HTML сайты с помощью Jekyll. 

<!-- more -->

### Установика RVM и Ruby
#### Установка RVM
Выполните в командной строке:
	curl -L https://get.rvm.io | bash -s stable --ruby

#### Установка Ruby
Выполните в командной строке:
	rvm install 1.9.3
	rvm use 1.9.3
	rvm rubygems latest

### Установка Octopress
Я выполнял последовательно следующие команды:
	git clone git://github.com/imathis/octopress.git octopress
	cd octopress
	bundle install
В этот момент возникла ошибка о том, что не установлен пакет fast-stemmer версии 1.0.1, в описании ошибки было указано, как установить необходимую версию.
	sudo gem install fast-stemmer -v '1.0.1'
	bundle install
	rake install
	rake setup_github_pages
В этом моменте надо будет указать адрес репозитория на github, например: git@github.com:username/username.github.io.git
	rake generate
После генерации страниц можно отправить исходный код блога на гитхаб в ветку source, если ее нет, то ветвь создастся.
	git push origin source
	rake preview
	rake deploy

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

### Добавление статей
Новая статья добавляется командой:
	rake new_post["Название новой статьи"]

#### Превью статьи на главной странице
Если статья длинная, то можно добавить кнопку "Читать дальше" после абзаца, для этого нужно изменить строку в файле _config.yml, указав в кавычках текст кнопки:
	excerpt_link: "Читать дальше &rarr;" 
В файле статьи, после абзаца нужно добавить строку:
	<!-- more -->

### Возможные ошибки
У меня возникли проблемы с rake, при вызове команд появлялась ошибка:
	/usr/local/bin/rake:23:in `load': cannot load such file -- /usr/share/rubygems-integration/1.9.1/gems/rake-10.0.4/bin/rake (LoadError)
		from /usr/local/bin/rake:23:in `<main>'
Быстрым решением может быть вызов следущей команды:
	bundle exec rake generate

