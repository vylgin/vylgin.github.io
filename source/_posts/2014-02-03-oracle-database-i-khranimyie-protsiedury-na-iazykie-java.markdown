---
layout: post
title: "Oracle Database и хранимые процедуры на языке Java"
date: 2014-02-03 23:19:47 +0400
comments: true
categories: [oracle, java]
---

{% img left https://cloclo1.cloud.mail.ru/thumb/xw1/blog/images/post_2/java_oracle_logo.jpg %}
В БД Oracle можно использовать хранимые процедуры и хранимые функции на языке Java благодаря встроенной JVM. С помощью них можно выполнять те задачи, с которыми не справляется PL/SQL, например, если необходимо обеспечить взаимодействие с операционной системой.

Для создания хранимой процедуры нужно выполнить три действия:

1. Создать исходный код с хранимой процедурой на языке Java. 
2. Загрузить скомпилированный код в Oracle. 
3. Создать псевдоним хранимой процедуры на PL/SQL.

Рассмотрим каждое из них.

<!-- more -->

#### 1. Создание исходного кода хранимой процедуры на языке Java

Для создания хранимой процедуры или функции на языке Java, нужно создать класс, в нем определить публичный статический метод, от возвращаемого значения которого зависит будет ли это метод хранимой процедурой или хранимой функцией. 

`public static void procedure() { }` - хранимая процедура.

`public static int function() { return -1; }` - хранимая функция.

Для использования IN параметра, нужно в статический метод передать ссылку на объект.

	public static void procedure(String str) { }

Для использования OUT или IN OUT параметра, нужно передать ссылку на массив.

	public static void procedure(String[] str) {}

#### 2. Загрузка скомпилированного кода в Oracle
Загрузить созданный класс можно с помощью командной строки:

	javac Swapper.java
    $ loadjava -user username/password Swapper.class

Я использовал Oracle SQL Developer и с помощью него загружал файл с исходным кодом:

{% img center https://cloclo9.datacloudmail.ru/view//blog/images/post_2/img_2.png %}

#### 3. Создание псевдонима хранимой процедуры на PL/SQL 

Псевдоним на PL/SQL для Java кода можно создать по шаблону

	CREATE OR REPLACE PROCEDURE имя_процедуры (x IN NUMBER, y IN OUT NUMBER)
	AS LANGUAGE JAVA
	NAME 'ИмяКласса.название_метода(int[], int[])';


### Пример создания хранимой процедуры.

Рассмотрим пример хранимой процедуры для обмена значений двух целочисленных переменных.

1. Создадим класс и статический метод для хранимой процедуры:

		public class Swapper {
		  public static void swap (int[] x, int[] y) {
		    int tmp = x[0];
		    x[0] = y[0];
		    y[0] = tmp;
		  }
		}

2. Скомпилируем и загрузим класс в Oracle Database:

		javac Swapper.java
    	$ loadjava -user username/password Swapper.class

3. Создадим псевдоним хранимой процедуры на PS/SQL:

		CREATE OR REPLACE PROCEDURE swap (x IN OUT NUMBER, y IN OUT NUMBER)
		AS LANGUAGE JAVA
		NAME 'Swapper.swap(int[], int[])';

4. Проверим работу созданной процедуры:

		SET SERVEROUTPUT ON
		declare
		   a number := 5;
		   b number := 3;
		begin
		   swap(a, b);
		   dbms_output.put_line(a || ' ' || b);
		end;
		/

	Убедимся, что на выводе отобразилось `3 5`.

### Вывод отладочной информации в dbms

Вывод отладочной информации с использованием `System.out.println("Отладочное сообщение")` не работает в хранимых процедурах, нужно подключиться к dbms текущего коннекта, для этого можно написать собственный метод: 

    private static void log(String s) {
        try {
            PreparedStatement ps = new OracleDriver()
                        .defaultConnection()
                        .prepareStatement("begin dbms_output.put_line(:x); end;");
            ps.setString(1, s);
            ps.execute();
            ps.close();
        } catch (SQLException e) {
            // Обработка исключения
        }
    }

#### Пример хранимой процедуры с выводом отладочной информации

1. Создание Java-кода:

		import java.sql.*;
		import oracle.jdbc.*;
	
		public class Example {
	
			public static void example (int x) {
				int z = x + 1;
				log("z = " + z);
			}
	
		    private static void log(String s) {
		        try {
		            PreparedStatement ps = new OracleDriver()
		                        .defaultConnection()
		                        .prepareStatement("begin dbms_output.put_line(:x); end;");
		            ps.setString(1, s);
		            ps.execute();
		            ps.close();
		        } catch (SQLException e) {
		            // Обработка исключения
		        }
		    }
		}

2. Компилирование и загрузка Java-кода в Oracle Database:

		javac Example.java
    	$ loadjava -user username/password Example.class

3. Создание псевдонима хранимой процедуры на PL/SQL:

		CREATE OR REPLACE PROCEDURE example (x IN NUMBER)
		AS LANGUAGE JAVA
		NAME 'Example.example(int)';

4. Проверка выполнения хранимой процедуры:
	
		SET SERVEROUTPUT ON
		declare
		   a number := 5;
		begin
		   example(a);
		end;
		/

	Вывод:
		
		z = 6
