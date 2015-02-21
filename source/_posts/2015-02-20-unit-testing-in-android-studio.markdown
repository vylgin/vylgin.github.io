---
layout: post
title: "Unit-тестирование в Android Studio становится проще"
date: 2015-02-20 12:19:32 -0500
comments: true
categories: [android, gradle, testing]
---

{% img left /images/post_2015-02-20/android-testing.png 150 %}
*«Как бы хорошо было запускать unit-тесты без симулятора или физического android устройства, — скорее всего, именно так думает каждый android-разработчик. — Да еще и без Robolectric, на локальной JVM, отделив их от функциональных».*

Возможность разделения исходного кода функциональных и unit-тестов появилась в Android Studio версии 1.1 и плагина Gradle версии 1.1.0, с выполнением последних на локальной JVM.

<!-- more -->

### Включение поддержки unit-тестов в Android Studio

1. Для начала необходимо обновить Android Studio до версии 1.1.

2. Обновить версию gradle плагина в build.gradle на 1.1.0 или более новую (это можно сделать из среды разработки File > Project Structure) 

    {% img /images/post_2015-02-20/2.png %}
    
3. Добавить JUnit зависимость в app/build.gradle

    {% img /images/post_2015-02-20/3.png %}

        dependencies {
            testCompile 'junit:junit:4.12'
        }

4. Включить unit-тестирование в Settings > Gradle > Experimental

    {% img /images/post_2015-02-20/4.png %}
 
5. Выполнить синхронизацию проекта с помощью gradle

    {% img /images/post_2015-02-20/5.png 300 %}

6. Открыть панель "Build variants" и изменить Test Artifact на "Unit tests"

    {% img /images/post_2015-02-20/6.png %}

7. Добавить каталоги `src/test/java` в модуль проекта

    {% img /images/post_2015-02-20/7.png 200 %}

### Запуск тестов с помощью Gradle

Для запуска unit-тестов нужно выполнить из каталога проекта, в консоли, команду для linux (OS X):

    ./gradlew test
    
Для windows:
    
    gradlew.bat test

Можно указать тип запускаемого теста командами:

    ./gradlew testDebug
    ./gradlew testRelease
    
Для выполнения одного теста, или группы тестов, можно задать маску у флага `--tests` команды `gradlew`:

    ./gradlew testDebug --tests='*.MyTestClass'
    
Чтобы запускать отдельно unit-тесты от функциональных, я задаю имена файлов с `*UnitTest.java` на конце и запускаю их командой:

    ./gradlew testDebug --tests='*UnitTest'


