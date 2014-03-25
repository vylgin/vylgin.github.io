---
layout: post
title: "Создание Sliding Menu приложения в Android Studio"
date: 2014-03-25 09:27:00 -0400
comments: true
categories: [android, slidingmenu, actionbarsherlock]
---

Рассмотрим пример создания приложения с использованием библиотек [Sliding Menu](https://github.com/jfeinstein10/SlidingMenu "Sliding Menu GitHub"), [ActionBarSherlock](http://actionbarsherlock.com) и среды разработки [Android Studio](http://developer.android.com/sdk/installing/studio.html).

Приложение будет состоять из бокового меню и пары фрагментов:

{% img /images/post_3/Page_1.png 300 %}
{% img /images/post_3/Page_2.png 300 %}

{% img /images/post_3/Page_3.png 300 %}
{% img /images/post_3/Page_4.png 300 %}

<!-- more --> 

### Создание проекта в Android Studio

С созданием проекта не должно возникнуть сложностей, если вы хоть раз использовали IntelliJ IDEA.

{% img center /images/post_3/create_project.png %}

### Импортирование Sliding Menu в проект Android Studio.

Создадим каталог `lib`, в котором будет хранится библиотека Sliding Menu. 

{% img center /images/post_3/add_lib_directory.png %}

Заходим на страницу проекта [https://github.com/jfeinstein10/SlidingMenu]() и сохраняем каталог `library` с помощью утилиты git, или скачав архив с исходниками. Переименовываем каталог `library` в `SlidingMenu` и копируем в только что созданный каталог `lib` нашего проекта, должна получиться такая структура проекта:

{% img center /images/post_3/project_structure.png %}

Добавляем в файл `settings.gradle` строку:

	include ":lib:SlidingMenu"

Добавляем в файл `SlidingApplication/app/build.gradle` в структуру зависимостей (`dependencies`) модуля `app` строку:

	compile project(':lib:SlidingMenu')

Стоит обратить внимание на то, что в обоих файлах `build.gradle`, нашего проекта и подключенной библиотеки Sliding Menu, должны совпадать значения у `compileSdkVersion` и `buildToolsVersion`. 

#### Проверка работы приложения после подключения библиотеки Sliding Menu.

Для бокового меню нужно создадим файл разметки `sidemenu.xml` в каталоге `SlidingApplication/app/src/main/res/layout/` 

	 <?xml version="1.0" encoding="utf-8"?>
	
	<ListView android:id="@+id/sidemenu"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    android:layout_marginRight="50dp"
	    android:divider="@null"
	    android:dividerHeight="0px"
	    android:footerDividersEnabled="false"
	    android:headerDividersEnabled="false"
	    android:choiceMode="singleChoice"
	    xmlns:android="http://schemas.android.com/apk/res/android">
	</ListView>

Создадим файл разметки для элемента меню `sidemenu_item.xml` в каталоге `SlidingApplication/app/src/main/res/layout/`

	<?xml version="1.0" encoding="utf-8"?>
	
	<LinearLayout android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    android:layout_margin="0dp"
	    android:orientation="vertical"
	    android:padding="10dp"
	    xmlns:android="http://schemas.android.com/apk/res/android">
	
	    <TextView android:id="@+id/text"
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:layout_marginLeft="10dp"
	        android:text="TextView"/>
	
	</LinearLayout>

Добавим в файл `dimens.xml` значения ширины бокового меню в каталоге `SlidingApplication/app/src/main/res/values`:

	<dimen name="slidingmenu_behind_width">200dp</dimen>

Создадим класс `MainActivity`, унаследованного от `ActionBarActivity`, и переопределим метод `onCreate`:

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        SlidingMenu menu = new SlidingMenu(this);
        menu.setMode(SlidingMenu.LEFT);
        menu.setTouchModeAbove(SlidingMenu.TOUCHMODE_FULLSCREEN);
        menu.setFadeDegree(0.35f);
        menu.attachToActivity(this, SlidingMenu.SLIDING_WINDOW);
        menu.setMenu(R.layout.sidemenu);
        menu.setBehindWidthRes(R.dimen.slidingmenu_behind_width);

        String[] items = {"Первый фрагмент","Второй фрагмент"};
        ((ListView) findViewById(R.id.sidemenu)).setAdapter(
                new ArrayAdapter<Object>(
                        this,
                        R.layout.sidemenu_item,
                        R.id.text,
                        items
                )
        );
    }

Теперь можно собрать и запустить приложение. Если провести пальцем слева на право в любой месте окна приложения, то откроется боковое меню с двумя пунктами.

### Импортирование ActionBarSherlock в проект Android Studio.

Импортировать ActionBarSherlock очень просто, достаточно в файл `SlidingApplication/app/build.gradle` в структуру зависимостей (`dependencies`) модуля app добавить строки:

	compile 'com.actionbarsherlock:actionbarsherlock:4.4.0@aar'
	compile 'com.android.support:support-v4:19.0.0'

И удалить строку:

	compile 'com.android.support:appcompat-v7:+'

Должно получиться так:

	dependencies {
	    compile 'com.android.support:support-v4:19.0.0'
	    compile 'com.actionbarsherlock:actionbarsherlock:4.4.0@aar'
	    compile project(':lib:SlidingMenu')
	}

Нужно заменить тему приложения в файле `style.xml` в каталоге `SlidingApplication/app/src/main/res/values` на тему шерлока:

	<style name="AppTheme" parent="Theme.Sherlock.Light">

Удаляем каталог `menu` в `SlidingApplication/app/src/main/res/`

Изменим родительский класс нашего `MainActivity` на `SherlockFragmentActivity`

	public class MainActivity extends SherlockFragmentActivity

Собираем и запускаем приложение. Если оно работает, то мы успешно импортировали библиотеку ActionBarSherlock.

### Создание фрагментов приложения

Добавим строковый ресурс в `strings.xml` каталога `SlidingApplication/app/src/main/res/values`:  

	<string name="callmenu"><![CDATA[<- ВЫЗОВ МЕНЮ!]]></string>

Создадим два java класса фрагментов `FirstFragment` и `SecondFragment`, унаследованных от класса `SherlockFragment`.

#### `FirstFragment`

Первый фрагмент будет отображать изображение и текст.

Создадим файл разметки для фрагмента `fragment_first.xml` в каталоге `SlidingApplication/app/src/main/res/layout/`

	<?xml version="1.0" encoding="utf-8"?>
	
	<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
	    android:layout_width="fill_parent"
	    android:layout_height="fill_parent">
	
	    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	        android:orientation="vertical" android:layout_width="match_parent"
	        android:layout_height="match_parent">
	    
	        <ImageView
	            android:layout_width="wrap_content"
	            android:layout_height="wrap_content"
	            android:layout_gravity="center_horizontal"
	            android:src="@drawable/ic_launcher"/>
	    
	        <TextView
	            android:layout_width="wrap_content"
	            android:layout_height="wrap_content"
	            android:layout_gravity="center_horizontal"
	            android:text="@string/large_text"/>
	    
	    </LinearLayout>
	
	</ScrollView>

Добавим строковый ресурс в `strings.xml` каталога `SlidingApplication/app/src/main/res/values`:  

	<string name="large_text">Некоторый очень длинный текст</string>

Добавим в класс `FirstFragment` код:

	public class FirstFragment extends SherlockFragment {
	    private ActionBar actionBar;
	
	    @Override
	    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
	        View view = inflater.inflate(R.layout.fragment_first, container, false);
	
	        actionBar = getSherlockActivity().getSupportActionBar();
	        actionBar.removeAllTabs();
	        actionBar.setNavigationMode(ActionBar.NAVIGATION_MODE_STANDARD);
	        actionBar.setTitle(getString(R.string.first_fragment_name));
	
	        return view;
	    }  
	}

Добавим обработчик событий на пункты меню в классе `MainActivity`:

    ((ListView) findViewById(R.id.sidemenu)).setOnItemClickListener(new AdapterView.OnItemClickListener() {
        @Override
        public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
            changeFragment(position);
        }
    });

А также метод смены фрагмента:

    private void changeFragment(int position) {
        switch (position) {
            case 0:
                showFragment(new FirstFragment());
                break;
        }
    }

И метод для отображения нового фрагмента:

    private void showFragment(Fragment currentFragment) {
        FragmentManager fragmentManager = getSupportFragmentManager();
        fragmentManager.beginTransaction()
                .replace(R.id.container, currentFragment)
                .commit();
    }

Собираем проект, запускаем приложение, открываем меню и нажимаем на первый пункт, радуемся появившемуся фрагменту. Только есть минус, надо вручную закрыть меню сдвигом появившегося фрагмента, но это мы исправим в дальнейшем.

#### `SecondFragment`

Второй фрагмент будет отображать табы, при смене которых будет меняться цвет фона.

Создадим файл разметки для фрагмента отображающего табы `fragment_second.xml` в каталоге `SlidingApplication/app/src/main/res/layout/`:

	<?xml version="1.0" encoding="utf-8"?>
	
	<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent">
	
	    <android.support.v4.view.ViewPager
	        android:id="@+id/pager_color"
	        android:layout_width="fill_parent"
	        android:layout_height="wrap_content" >
	    </android.support.v4.view.ViewPager>
	
	</RelativeLayout>

Создадим файл разметки для фрагмента выбранного таба `fragment_second_color` в каталоге `SlidingApplication/app/src/main/res/layout/`:

	<?xml version="1.0" encoding="utf-8"?>
	
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:orientation="vertical" android:layout_width="match_parent"
	    android:layout_height="match_parent">
	
	    <ImageView
	        android:id="@+id/imageView"
	        android:layout_width="fill_parent"
	        android:layout_height="fill_parent"
	        android:layout_gravity="center_horizontal"
	        android:layout_marginLeft="50dp"
	        android:layout_marginRight="50dp"
	        android:layout_marginTop="50dp"
	        android:layout_marginBottom="50dp"/>
	</LinearLayout>

Создадим класс для второго фрагмента, отображающего табы:

	public class SecondFragment extends SherlockFragment {
	    private ActionBar actionBar;
	    private ViewPager pager;
	    private ActionBar.Tab tab;
	
	    @Override
	    public void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	    }
	
	    @Override
	    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
	
	        View view = inflater.inflate(R.layout.fragment_second, container, false);
	
	        actionBar = getSherlockActivity().getSupportActionBar();
	        actionBar.removeAllTabs();
	        actionBar.setNavigationMode(ActionBar.NAVIGATION_MODE_TABS);
	        actionBar.setTitle(getString(R.string.second_fragment_name));
	
	        ViewPager.SimpleOnPageChangeListener ViewPagerListener = new ViewPager.SimpleOnPageChangeListener() {
	            @Override
	            public void onPageSelected(int position) {
	                super.onPageSelected(position);
	                actionBar.setSelectedNavigationItem(position);
	            }
	        };
	
	        pager = (ViewPager) view.findViewById(R.id.pager_color);
	        pager.setOnPageChangeListener(ViewPagerListener);
	
	        FragmentManager fm = getChildFragmentManager();
	        ViewPagerColorAdapter viewPagerScheduleAdapter = new ViewPagerColorAdapter(fm);
	        pager.setAdapter(viewPagerScheduleAdapter);
	
	        ActionBar.TabListener tabListener = new ActionBar.TabListener() {
	
	            @Override
	            public void onTabSelected(ActionBar.Tab tab, FragmentTransaction ft) {
	                pager.setCurrentItem(tab.getPosition());
	            }
	
	            @Override
	            public void onTabUnselected(ActionBar.Tab tab, FragmentTransaction ft) { }
	
	            @Override
	            public void onTabReselected(ActionBar.Tab tab, FragmentTransaction ft) { }
	        };
	
	        tab = actionBar.newTab().setText("КРАСНЫЙ").setTabListener(tabListener);
	        actionBar.addTab(tab);
	
	        tab = actionBar.newTab().setText("ЗЕЛЕНЫЙ").setTabListener(tabListener);
	        actionBar.addTab(tab);
	
	        tab = actionBar.newTab().setText("СИНИЙ").setTabListener(tabListener);
	        actionBar.addTab(tab);
	
	        return view;
	    }
	}

Создадим класс адаптера `ViewPagerColorAdapter`, унаследованного от класса `FragmentPagerAdapter`:

	public class ViewPagerColorAdapter extends FragmentPagerAdapter {
	    final int PAGE_COUNT = 3;
	
	    public ViewPagerColorAdapter(FragmentManager fm) {
	        super(fm);
	    }
	
	    @Override
	    public Fragment getItem(int arg0) {
	        switch (arg0) {
	            case 0:
	                return new FragmentTabColor(Color.RED);
	            case 1:
	                return new FragmentTabColor(Color.GREEN);
	            case 2:
	                return new FragmentTabColor(Color.BLUE);
	        }
	
	        return null;
	    }
	
	    @Override
	    public int getCount() {
	        return PAGE_COUNT;
	    }
	}

Создадим класс фрагмента для отображения цветного прямоугольника `FragmentTabColor`:

	public class FragmentTabColor extends SherlockFragment {
	
	    private int color;
	
	    public FragmentTabColor(int color) {
	        this.color = color;
	    }
	
	    @Override
	    public View onCreateView(LayoutInflater inflater,
	                             ViewGroup container,
	                             Bundle savedInstanceState) {
	        View view = inflater.inflate(R.layout.fragment_second_color, container, false);
	        ImageView imageView = (ImageView) view.findViewById(R.id.imageView);
	        imageView.setBackgroundColor(color);
	
	        return view;
	    }
	
	    @Override
	    public void onSaveInstanceState(Bundle outState) {
	        super.onSaveInstanceState(outState);
	        setUserVisibleHint(true);
	    }
	}

В метод `changeFragment(int position)` класса `MainActivity`, в конструкцию `switch`, добавим `case` для отображения второго фрагмента:

    private void changeFragment(int position) {
        switch (position) {
            case 0:
                showFragment(new FirstFragment());
                break;
            case 1:
                showFragment(new SecondFragment());
                break;

        }
    }

На этом этапе можно собрать и запустить приложение. При перемещении пальцем на следующий таб, стоящий справа от текущего, приложение ведет себя корректно, но если переместиться на таб, стоящий левее - то откроется меню приложения. Чтобы добиться правильного поведения приложения на жесты пользователя, выполним действия.

Добавим в класс `MainActivity` приватное поле:

	private SlidingMenu menu;

В конце метода `onCreate` проинициализируем его:

	this.menu = menu;

Также добавим геттер:

    public SlidingMenu getMenu() {
        return menu;
    }

Добавим в обработчик `onTabSelected` фрагмента `SecondFragment` следующий код, который позволяет комфортно переключаться между табами:

	SherlockFragmentActivity sherlockFragmentActivity = getSherlockActivity();
		if (sherlockFragmentActivity != null) {
			SlidingMenu slidingMenu = ((MainActivity) sherlockFragmentActivity).getMenu();
			if (tab.getPosition() == 0) {
				slidingMenu.setTouchModeAbove(SlidingMenu.TOUCHMODE_FULLSCREEN);
			} else {
				slidingMenu.setTouchModeAbove(SlidingMenu.TOUCHMODE_MARGIN);
			}
	}

Собираем приложение и проверяем работу созданного фрагмента.

### Доработка меню приложения

#### Изменение поведения меню

Добавим возможность открытия меню, нажатием на иконку приложения, для этого в метод `onCreate` класса `MainActivity` добавим строки:

	getSupportActionBar().setHomeButtonEnabled(true);
	getSupportActionBar().setDisplayShowCustomEnabled(true);
	getSupportActionBar().setDisplayHomeAsUpEnabled(true);

И в этот же класс добавим два метода:

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case android.R.id.home:
                menuToggle();
                return true;
        }
        return super.onOptionsItemSelected(item);
    }

    public void menuToggle(){
        if(menu.isMenuShowing())
            menu.showContent();
        else
            menu.showMenu();
    }

Для того, чтобы после выбора фрагмента закрывалось меню, добавим в обработчик `onItemClick`, находящегося в методе `onCreate` класса `MainActivity`, вызов метода `menuToggle()`:

	((ListView) findViewById(R.id.sidemenu)).setOnItemClickListener(new AdapterView.OnItemClickListener() {
		@Override
		public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
			menuToggle();
			changeFragment(position);
		}
	});

Приложение можно собрать и убедиться, что при нажатии на логотип, открывается меню, а при выборе фрагмента - меню закрывается.

Для того, чтобы меню закрывалось при нажатии на аппаратную кнопку "назад", можно добавить следующие строки в класс `MainActivity`:

    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        if (keyCode == KeyEvent.KEYCODE_BACK) {
            if(menu.isMenuShowing()){
                menu.toggle(true);
                return false;
            }
        }
        return super.onKeyDown(keyCode, event);
    }

#### Изменение внешнего вида меню и выделение выбранного пункта.

Первым делом - добавим фон меню и стиль пунктов в метод `onCreate` класса `MainActivity`:

	menu.setBackgroundColor(0xFF333333);
	menu.setSelectorDrawable(R.drawable.sidemenu_items_background);

Создадим файл `sidemenu_items_background`, описывающий поведение пункта меню на действия пользователя, в каталоге `SlidingApplication/app/src/main/res/drawable/`:

	<?xml version="1.0" encoding="utf-8"?>
	
	<selector xmlns:android="http://schemas.android.com/apk/res/android">
	    <item android:state_pressed="true" android:drawable="@drawable/sidemenu_item_background_pressed"/>
	    <item android:state_focused="true" android:drawable="@drawable/sidemenu_item_background_focuded"/>
	    <item android:state_activated="true" android:drawable="@drawable/sidemenu_item_background_focuded"/>
	    <item android:drawable="@drawable/sidemenu_item_background"/>
	</selector>

Следует внести изменения в файл `sidemenu_item.xml`, чтобы пункты меню стали показываться в выбранной разметке: 

- В LinearLayout добавить

		android:background="@drawable/sidemenu_items_background"
 
- В TextView добавить

		android:textColor="#fff"

Создадим файл `sidemenu_item_background`, описывающий внешний вид пункта меню, в каталоге `res/drawable`:

	<?xml version="1.0" encoding="utf-8"?>
	
	<layer-list xmlns:android="http://schemas.android.com/apk/res/android" >
	
	    <item android:left="0dp" android:top="0dp" android:right="0dp" android:bottom="1dp" >
	        <shape android:shape="rectangle">
	            <solid android:color="#404040"/>
	        </shape>
	    </item>
	
	    <item android:left="0dp" android:top="1dp" android:right="0dp" android:bottom="0dp" >
	        <shape android:shape="rectangle">
	            <solid android:color="#1A1A1A"/>
	        </shape>
	    </item>
	
	    <item android:left="0dp" android:top="1dp" android:right="0dp" android:bottom="1dp" >
	        <shape android:shape="rectangle">
	            <solid  android:color="#333"/>
	        </shape>
	    </item>
	
	</layer-list>

Создадим файл `sidemenu_item_background_pressed`, описывающий внешний вид пункта меню в момент нажатия, в каталоге `res/drawable`:

	<?xml version="1.0" encoding="utf-8"?>
	
	<layer-list xmlns:android="http://schemas.android.com/apk/res/android" >
	    <item>
	        <shape xmlns:android="http://schemas.android.com/apk/res/android" android:shape="rectangle">
	            <solid android:color="@color/menu_focused_pressed_gray_color"/>
	        </shape>
	    </item>
	</layer-list>

Создадим файл `sidemenu_item_background_focuded`, описывающий внешний вид пункта меню в момент нахождения в фокусе, в каталоге `res/drawable`:

	<?xml version="1.0" encoding="utf-8"?>
	
	<layer-list xmlns:android="http://schemas.android.com/apk/res/android" >
	
	    <item>
	        <shape xmlns:android="http://schemas.android.com/apk/res/android" android:shape="rectangle">
	            <solid android:color="@color/blue_dark_color"/>
	        </shape>
	    </item>
	
	    <item android:left="10dp" android:top="0dp" android:right="0dp" android:bottom="0dp" >
	        <shape android:shape="rectangle">
	            <solid android:color="@color/menu_focused_pressed_gray_color"/>
	        </shape>
	    </item>
	
	</layer-list>

Осталось создать файл для хранения цветов `colors.xml` в каталоге `res/values`:

	<?xml version="1.0" encoding="utf-8"?>
	<resources>
	    <color name="menu_focused_pressed_gray_color">#404040</color>
	    <color name="blue_dark_color">#3280b8</color>
	</resources>

Привычным способом собираем и проверяем работу приложения. Должно получиться так:

{% img /images/post_3/scr_1.png 250 %}
{% img /images/post_3/scr_2.png 250 %}
{% img /images/post_3/scr_3.png 250 %}


### Страница проекта на github

С полным исходным кодом приложения можно ознакомиться на странице проекта [https://github.com/vylgin/SlidingApplication](https://github.com/vylgin/SlidingApplication).
