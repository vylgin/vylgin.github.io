---
layout: post
title: "Совместное использование Java и Javascript"
date: 2014-12-05 03:58:25 -0500
comments: true
categories: [java, javascript, nashorn]
---

{% img left /images/post_2014-12-05/1.jpg 150 %}
Javascript победил на фронтенде и завоевывает бекенд, а если добавить богатство библиотек языка Java, то можно получить любопытную смесь.

Поддержка динамических языков началась в java 6, когда был реализован стандарт JSR 223. Поддержка javascript осуществлялась с помощью [Rhino](https://developer.mozilla.org/ru/docs/Rhino "Rhino"), но эта имплементация была очень медленной и в java 8 от нее отказались в пользу Nashorn.

<!-- more -->

### Nashorn

* Javascript engine для jvm.
* Полная java имплементация.
* Полностью совместима с ecmascript 5.1
* Полностью собирается в байткод, не интерпретируется

### Примеры использования

#### В качестве системных скриптов.

Создадим скрипт, который выводит текущий год, используя класс Date языка Java.

Создать файл `now_year.jjs`:

    #!/path_to_jdk8/bin/jjs

    var Date = Java.type('java.util.Date');

    var date = new Date();
    date.year += 1900;

    // Вывод с помощью javascript
    print("javascript, print:");
    print("Now year: " + date.year);

    // Вывод с помощью java
    var System = Java.type('java.lang.System');
    System.out.println("\njava, System.out.println:");
    System.out.println("Now year: " + date.year);

Сделаем исполняемым, выполнив в консоле команду:

        chmod +x now_year.jjs

Запустим скрипт:

        /now_year.jjs

Видим результат:

        javascript, print:
        Now year: 2014

        java, System.out.println:
        Now year: 2014

#### Добавление javascript в java код

Пример из статьи на хабре [Введение в Nashorn](http://habrahabr.ru/post/195870).

	import javax.script.*;
	
	public class EvalScript {
	    public static void main(String[] args) throws Exception {
	        // create a script engine manager
	        ScriptEngineManager factory = new ScriptEngineManager();
	        // create a Nashorn script engine
	        ScriptEngine engine = factory.getEngineByName("nashorn");
	        // evaluate JavaScript statement
	        try {
	            engine.eval("print('Hello, World!');");
	        } catch (final ScriptException se) { se.printStackTrace(); }
	    }
	}


Компилируем класс: 

	./javac EvalScript.java 

И выполняем его:

	./java EvalScript

Видим вывод: 

	Hello, World!

#### Расширение функциональности приложения в рантайме

Пример из статьи на хабре [Введение в Nashorn](http://habrahabr.ru/post/195870).

**MyScript.js**

	var MyClass = Java.type("EvalScript.MyClass");
	var my = new MyClass();
	my.printMsg("Hello!");

**EvalScript.java**

	import javax.script.*;
	import java.io.*;
	
	public class EvalScript {
	
	    public static void main(String[] args) throws Exception {
	        // create a script engine manager
	        ScriptEngineManager factory = new ScriptEngineManager();
	        // create a Nashorn script engine
	        ScriptEngine engine = factory.getEngineByName("nashorn");
	        // evaluate JavaScript statement
	        try {
	            BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
	            engine.eval(br);
	        } catch (final ScriptException se) { se.printStackTrace(); }
	    }
	
	    public static class MyClass {
	        public void printMsg(String msg) {
	            System.out.println("printMsg : "+msg);
	        }
	    }
	}

Скомпилировав, запускаем:

	./java EvalScript < MyScript.js

Видим вывод: 

	printMsg : Hello!

### Зачем это нужно?

Да хотя бы просто так, занять себя в один из вечеров.

### Ссылки

* [http://www.oracle.com/technetwork/articles/java/jf14-nashorn-2126515.html](http://www.oracle.com/technetwork/articles/java/jf14-nashorn-2126515.html)
* [http://www.infoworld.com/article/2607426/application-development/nashorn--javascript-made-great-in-java-8.html](http://www.infoworld.com/article/2607426/application-development/nashorn--javascript-made-great-in-java-8.html)
* [https://github.com/gAmUssA/java-scripting-experiments](https://github.com/gAmUssA/java-scripting-experiments)
* [https://speakerdeck.com/vikgamov/nosoroghi-na-volie-prikladnyie-vozmozhnosti-ispol-zovaniia-javascript-v-razrabotkie-na-java](https://speakerdeck.com/vikgamov/nosoroghi-na-volie-prikladnyie-vozmozhnosti-ispol-zovaniia-javascript-v-razrabotkie-na-java)
* [http://habrahabr.ru/post/195870](http://habrahabr.ru/post/195870/)
