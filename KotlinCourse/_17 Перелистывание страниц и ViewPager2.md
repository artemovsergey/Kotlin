# Перелистывание страниц и ViewPager2

# ViewPager2 и разделение приложения на страницы

Нередко можно встретить приложения, которые реализуют систему перелистывания, а само приложение предстает в виде набора страниц, которые можно пролистывать влево и вправо. В приложении Android для создания подобного эффекта можно использовать элемент ViewPager2 из комплекта JetPack. Для создания эффекта страниц ViewPager2 использует фрагменты.

Итак, создадим новый проект. Добавим в папку res/layout файл разметки для фрагмента, который будет представлять страницу. Назовем его fragment_page.xml и определим в нем следующий код:

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <TextView
        android:id="@+id/displayText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="22sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />
 
</androidx.constraintlayout.widget.ConstraintLayout>
```
Фрагмент будет отображать текстовое поле с номером страницы.

Теперь добавим в проект сам класс фрагмента. Назовем его PageFragment:

```java
package com.example.viewpagerapp;
 
import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;
 
import androidx.fragment.app.Fragment;
 
public class PageFragment extends Fragment {
 
    private int pageNumber;
 
    public static PageFragment newInstance(int page) {
        PageFragment fragment = new PageFragment();
        Bundle args=new Bundle();
        args.putInt("num", page);
        fragment.setArguments(args);
        return fragment;
    }
 
    public PageFragment() {
    }
 
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        pageNumber = getArguments() != null ? getArguments().getInt("num") : 1;
    }
 
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        View result=inflater.inflate(R.layout.fragment_page, container, false);
        TextView pageHeader = result.findViewById(R.id.displayText);
        String header = "Фрагмент " + (pageNumber+1);
        pageHeader.setText(header);
        return result;
    }
}
```
Переменная pageNumber указывает на номер текущей страницы. Номер страницы будет передаваться извне через фабричный метод newInstance(). Передача номера происходит путем добавления значения в аргумент "num"

Затем при создании фрагмента в методе onCreate() этот номер будет извлекаться из аргумента "num" (если аргументы определены):

```java
pageNumber = getArguments() != null ? getArguments().getInt("num") : 1;
```
В методе onCreateView() полученный номер страницы будет отображаться в текстовом поле.

Сам по себе фрагмент еще не создает функциональность постраничной навигации. Для этого нам нужен один из классов PagerAdapter. Android SDK содержит ряд встроенных реализаций PagerAdapter, в частности, класс FragmentStateAdapter. Этот класс являются абстрактным, поэтому напрямую мы его использовать не можем, и нам нужно создать класс-наследник. Для этого добавим в проект новый класс, который назовем MyAdapter:

```java
package com.example.viewpagerapp;
 
import androidx.annotation.NonNull;
import androidx.fragment.app.Fragment;
import androidx.fragment.app.FragmentActivity;
import androidx.viewpager2.adapter.FragmentStateAdapter;
 
public class MyAdapter extends FragmentStateAdapter {
    public MyAdapter(FragmentActivity fragmentActivity) {
        super(fragmentActivity);
    }
 
    @NonNull
    @Override
    public Fragment createFragment(int position) {
        return(PageFragment.newInstance(position));
    }
 
    @Override
    public int getItemCount() {
        return 10;
    }
}
```
Класс FragmentStateAdapter определяет два метода:

- int getItemCount(): возвращает количество страниц, которые будут в ViewPager2 (в нашем случае 10)

- Fragment createFragment(int position): по номеру страницы, передаваемому в качестве параметра position, возвращает объект фрагмента

Стоит отметить, что в качестве параметра конструктор FragmentStateAdapter принимает контекст выполнения - обычно это объект FragmentActivity, но также это может быть объект Fragment

В завершении установим в файле activity_main.xml элемент ViewPager2:

```xml
<androidx.viewpager2.widget.ViewPager2
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/pager"
    android:layout_width="match_parent"
    android:layout_height="match_parent"  />
```
И также изменим код MainActivity:

```xml
package com.example.viewpagerapp;
 
import androidx.appcompat.app.AppCompatActivity;
import androidx.viewpager2.adapter.FragmentStateAdapter;
import androidx.viewpager2.widget.ViewPager2;
 
import android.os.Bundle;
 
public class MainActivity extends AppCompatActivity {
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
 
        ViewPager2 pager = findViewById(R.id.pager);
        FragmentStateAdapter pageAdapter = new MyAdapter(this);
        pager.setAdapter(pageAdapter);
    }
}
```
Класс MainActivity наследуется от AppCompatActivity - класса, который в свою очередь наследуется от FragmentActivity, и поэтому ее текущий объект мы можем передать в качестве параметра в конструктор MyAdapter (а через него - в конструктор FragmentStateAdapter). Чтобы перелистывание заработало, для ViewPager2 устанавливается адаптер MyAdapter.

И запустив проект, мы сможем с помощью перелистывания перемещаться по страницам:

![](https://metanit.com/java/android/pics/pageviewer1.png)

# Заголовки страниц и TabLayout

В прошлой теме мы рассмотели, как создать функциональность перелистывания страниц. Теперь пойдем дальше и добавим к страницам заголовки, посредством которых мы можем дополнительно перемещаться по станицам.

Для добавления заголовков мы можем использовать встроенный виджет TabLayout, который создает некоторое подобие вкладки над страницей.

Используем TabLayout. Для этого возьмем проект из прошлой темы. И прежде нам надо добавить в проект поддержку для этого виджета. Для этого в файле build.gradle добавим зависимость:

```java
implementation "com.google.android.material:material:1.4.0"
```
Далее изменим файл activity_main.xml:

```xml
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <com.google.android.material.tabs.TabLayout
        android:id="@+id/tab_layout"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        app:layout_constraintBottom_toTopOf="@id/pager"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"/>
         
    <androidx.viewpager2.widget.ViewPager2
        android:id="@+id/pager"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toBottomOf="@id/tab_layout"/>
         
</androidx.constraintlayout.widget.ConstraintLayout>
```
В данном случае над элементом ViewPager2 располагается элемент TabLayout, который определяет заголовок для определенной страницы во ViewPager2.

Затем необходимо связать ViewPager2 и TabLayout. Для этого применяется класс TabLayoutMediator. Итак, изменим код MainActivity, чтобы связать ViewPager2 и TabLayout:

```java
package com.example.viewpagerapp;
 
import androidx.appcompat.app.AppCompatActivity;
import androidx.viewpager2.adapter.FragmentStateAdapter;
import androidx.viewpager2.widget.ViewPager2;
 
import android.os.Bundle;
 
import com.google.android.material.tabs.TabLayout;
import com.google.android.material.tabs.TabLayoutMediator;
 
public class MainActivity extends AppCompatActivity {
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ViewPager2 pager = findViewById(R.id.pager);
        FragmentStateAdapter pageAdapter = new MyAdapter(this);
        pager.setAdapter(pageAdapter);
 
        TabLayout tabLayout = findViewById(R.id.tab_layout);
        TabLayoutMediator tabLayoutMediator= new TabLayoutMediator(tabLayout, pager, new TabLayoutMediator.TabConfigurationStrategy(){
 
            @Override
            public void onConfigureTab(TabLayout.Tab tab, int position) {
                tab.setText("Страница " + (position + 1));
            }
        });
        tabLayoutMediator.attach();
    }
}
```
Конструктор TabLayoutMediator принимает три параметра: объекты ViewPager2 и TabLayout и реализацию интерфейса TabConfigurationStrategy, которая с помощью метода onConfigureTab() получает отдельную вкладку в виде объекта Tab и номер страницы и позволяет настроить вид вкладки, например, установить заголовок вкладки.

После создания объекта TabLayoutMediator необходимо вызывать у него метод attach(). Все остальное остается без изменений. Запустим проект на выполнение и увидим интерактивные вкладки-заголовки поверх страниц.

Вид приложения при трех вкладках:

![](https://metanit.com/java/android/pics/tabpage.png)


