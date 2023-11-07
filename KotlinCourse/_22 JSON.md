# Работа с json

Для работы с форматом json нет встроенных средств, но есть куча библиотек и пакетов, которые можно использовать для данной цели. Одним из наиболее популярных из них является пакет com.google.code.gson.

Для его использования в проекте Android в файл guild.gradle, который относится к модулю app, в секцию dependencies необходимо добавить соответствующую зависимость:

```java
implementation 'com.google.code.gson:gson:2.8.8'
```
То есть после добавления секция зависимостей dependencies в файле build.gradle может выглядеть следующим образом:

```java
dependencies {
 
    implementation 'com.google.code.gson:gson:2.8.6'
 
    implementation 'androidx.appcompat:appcompat:1.2.0'
    implementation 'com.google.android.material:material:1.2.1'
    implementation 'androidx.constraintlayout:constraintlayout:2.0.4'
    testImplementation 'junit:junit:4.+'
    androidTestImplementation 'androidx.test.ext:junit:1.1.2'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.3.0'
}
```
После добавления пакета в проект добавим новый класс User, который будет представлять данные:

```java
package com.example.filesapp;
 
public class User {
 
    private String name;
    private int age;
 
    User(String name, int age){
        this.name = name;
        this.age = age;
    }
 
    public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
 
    public int getAge() {
        return age;
    }
 
    public void setAge(int age) {
        this.age = age;
    }
 
    @Override
    public  String toString(){
        return "Имя: " + name + " Возраст: " + age;
    }
}
```
Объекты этого класса мы будем сериализовать в формат json и наоборот десериализовать из файла.

Для работы с json добавим следующий класс JSONHelper:

```java
package com.example.filesapp;
 
import android.content.Context;
import com.google.gson.Gson;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.List;
 
class JSONHelper {
 
    private static final String FILE_NAME = "data.json";
 
    static boolean exportToJSON(Context context, List<User> dataList) {
 
        Gson gson = new Gson();
        DataItems dataItems = new DataItems();
        dataItems.setUsers(dataList);
        String jsonString = gson.toJson(dataItems);
 
        try(FileOutputStream fileOutputStream =
                    context.openFileOutput(FILE_NAME, Context.MODE_PRIVATE)) {
            fileOutputStream.write(jsonString.getBytes());
            return true;
        } catch (Exception e) {
            e.printStackTrace();
        }
 
        return false;
    }
 
    static List<User> importFromJSON(Context context) {
 
        try(FileInputStream fileInputStream = context.openFileInput(FILE_NAME);
        InputStreamReader streamReader = new InputStreamReader(fileInputStream)){
 
            Gson gson = new Gson();
            DataItems dataItems = gson.fromJson(streamReader, DataItems.class);
            return  dataItems.getUsers();
        }
        catch (IOException ex){
            ex.printStackTrace();
        }
 
        return null;
    }
 
    private static class DataItems {
        private List<User> users;
 
        List<User> getUsers() {
            return users;
        }
        void setUsers(List<User> users) {
            this.users = users;
        }
    }
}
```
package com.example.filesapp;
 
import android.content.Context;
import com.google.gson.Gson;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.List;
 
class JSONHelper {
 
    private static final String FILE_NAME = "data.json";
 
    static boolean exportToJSON(Context context, List<User> dataList) {
 
        Gson gson = new Gson();
        DataItems dataItems = new DataItems();
        dataItems.setUsers(dataList);
        String jsonString = gson.toJson(dataItems);
 
        try(FileOutputStream fileOutputStream =
                    context.openFileOutput(FILE_NAME, Context.MODE_PRIVATE)) {
            fileOutputStream.write(jsonString.getBytes());
            return true;
        } catch (Exception e) {
            e.printStackTrace();
        }
 
        return false;
    }
 
    static List<User> importFromJSON(Context context) {
 
        try(FileInputStream fileInputStream = context.openFileInput(FILE_NAME);
        InputStreamReader streamReader = new InputStreamReader(fileInputStream)){
 
            Gson gson = new Gson();
            DataItems dataItems = gson.fromJson(streamReader, DataItems.class);
            return  dataItems.getUsers();
        }
        catch (IOException ex){
            ex.printStackTrace();
        }
 
        return null;
    }
 
    private static class DataItems {
        private List<User> users;
 
        List<User> getUsers() {
            return users;
        }
        void setUsers(List<User> users) {
            this.users = users;
        }
    }
}

![](https://metanit.com/java/android/pics/json3.png)

Теперь определим основной функционал для взаимодействия с пользователем. Изменим файл activity_main.xml следующим образом:

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
 
    <EditText
        android:id="@+id/nameText"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:hint="Введите имя"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintBottom_toTopOf="@id/ageText"
        app:layout_constraintTop_toTopOf="parent" />
    <EditText
        android:id="@+id/ageText"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:hint="Введите возраст"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintBottom_toTopOf="@id/addButton"
        app:layout_constraintTop_toBottomOf="@id/nameText" />
    <Button
        android:id="@+id/addButton"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="Добавить"
        android:onClick="addUser"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintBottom_toTopOf="@id/saveButton"
        app:layout_constraintTop_toBottomOf="@id/ageText" />
    <Button
        android:id="@+id/saveButton"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="Сохранить"
        android:onClick="save"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toLeftOf="@id/openButton"
        app:layout_constraintBottom_toTopOf="@id/list"
        app:layout_constraintTop_toBottomOf="@id/addButton"/>
    <Button
        android:id="@+id/openButton"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="Открыть"
        android:onClick="open"
        app:layout_constraintLeft_toRightOf="@id/saveButton"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintBottom_toTopOf="@id/list"
        app:layout_constraintTop_toBottomOf="@id/addButton"/>
    <ListView
        android:id="@+id/list"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintTop_toBottomOf="@id/openButton" />
 
</androidx.constraintlayout.widget.ConstraintLayout>
```
Здесь определены два текстовых поля для ввода названия модели и цены объекта User и одна кнопка для добавления данных в список. Еще одна кнопка выполняет сериализацию данных из списка в файл, а третья кнопка - восстановление данных из файла.

Для вывода сами данных определен элемент ListView.

И изменим класс MainActivity:

```java
package com.example.filesapp;
 
import androidx.appcompat.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.ArrayAdapter;
import android.widget.EditText;
import android.widget.ListView;
import android.widget.Toast;
 
import java.util.ArrayList;
import java.util.List;
 
public class MainActivity extends AppCompatActivity {
 
    private ArrayAdapter<User> adapter;
    private EditText nameText, ageText;
    private List<User> users;
    ListView listView;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
 
        nameText = findViewById(R.id.nameText);
        ageText = findViewById(R.id.ageText);
        listView = findViewById(R.id.list);
        users = new ArrayList<>();
 
        adapter = new ArrayAdapter<>(this, android.R.layout.simple_list_item_1, users);
        listView.setAdapter(adapter);
    }
 
    public void addUser(View view){
        String name = nameText.getText().toString();
        int age = Integer.parseInt(ageText.getText().toString());
        User user = new User(name, age);
        users.add(user);
        adapter.notifyDataSetChanged();
    }
 
    public void save(View view){
 
        boolean result = JSONHelper.exportToJSON(this, users);
        if(result){
            Toast.makeText(this, "Данные сохранены", Toast.LENGTH_LONG).show();
        }
        else{
            Toast.makeText(this, "Не удалось сохранить данные", Toast.LENGTH_LONG).show();
        }
    }
    public void open(View view){
        users = JSONHelper.importFromJSON(this);
        if(users!=null){
            adapter = new ArrayAdapter<>(this, android.R.layout.simple_list_item_1, users);
            listView.setAdapter(adapter);
            Toast.makeText(this, "Данные восстановлены", Toast.LENGTH_LONG).show();
        }
        else{
            Toast.makeText(this, "Не удалось открыть данные", Toast.LENGTH_LONG).show();
        }
    }
}
```
Все данные находятся в списке users, который представляет объект List<User>. Через адаптер этот список связывается с ListView.

Для сохранения и восстановления данных вызываются ранее определенные методы в классе JSONHelper. Кнопка добавления добавляет данные в список user, и они сразу же отображаются в ListView. При нажатии на кнопку сохранения данные из списка users сохраняются в локальный файл. Затем с помощью кнопки открытия мы сможем открыть ранее сохраненный файл.

![](https://metanit.com/java/android/pics/json1.png)
