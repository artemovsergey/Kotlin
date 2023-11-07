# Работа с XML

## Ресурсы XML и их парсинг

Одним из распространенных форматов хранения и передачи данных является xml. Рассмотрим, как с ним работать в приложении на Android.

Приложение может получать данные в формате xml различными способами - из ресурсов, из сети и т.д. В данном случае рассмотрим ситуацию, когда файл xml хранится в ресурсах.

Возьмем стандартный проект Android по умолчанию и в папке res создадим каталог xml. Для этого нажмем на каталог res правой кнопкой мыши и в контекстном меню выберем New -> Android Resource Directory:

![](https://metanit.com/java/android/pics/xml4.png)

В появившемся окне в качестве типа ресурсов укажем xml:

![](https://metanit.com/java/android/pics/xml5.png)

В этот каталог добавим новый файл, который назовем users.xml и который будет иметь следующее содержимое:

```xml
<?xml version="1.0" encoding="utf-8"?>
<users>
    <user>
        <name>Tom</name>
        <age>36</age>
    </user>
    <user>
        <name>Alice</name>
        <age>32</age>
    </user>
    <user>
        <name>Bob</name>
        <age>28</age>
    </user>
</users>
```
Это обычный файл xml, который хранит набор элементов user. Каждый элемент характеризуется наличием двух подэлементов - name и age. Условно говоря, каждый элемент описывает пользователя, у которого есть имя и возраст.

В папку, где находится основной класс MainActivity, добавим новый класс, который назовем User:

```java
package com.example.xmlapp;
 
public class User {
    private String name;
    private String age;
 
    public String getName(){
        return name;
    }
    public String getAge(){
        return age;
    }
    public void setName(String name){
        this.name = name;
    }
    public void setAge(String age){
        this.age = age;
    }
    public String toString(){
        return  "User: " + name + " - " + age;
    }
}
```
Этот класс описывает товар, информация о котором будет извлекаться из xml-файла.

И в ту же папку добавим новый класс UserResourceParser:

![](https://metanit.com/java/android/pics/xml1.png)

Определим для класса UserResourceParser следующий код:

```java
package com.example.xmlapp;
 
import org.xmlpull.v1.XmlPullParser;
import java.util.ArrayList;
 
 
public class UserResourceParser {
    private ArrayList<User> users;
 
    public UserResourceParser(){
        users = new ArrayList<>();
    }
 
    public ArrayList<User> getUsers(){
        return  users;
    }
 
    public boolean parse(XmlPullParser xpp){
        boolean status = true;
        User currentUser = null;
        boolean inEntry = false;
        String textValue = "";
 
        try{
            int eventType = xpp.getEventType();
            while(eventType != XmlPullParser.END_DOCUMENT){
 
                String tagName = xpp.getName();
                switch (eventType){
                    case XmlPullParser.START_TAG:
                        if("user".equalsIgnoreCase(tagName)){
                            inEntry = true;
                            currentUser = new User();
                        }
                        break;
                    case XmlPullParser.TEXT:
                        textValue = xpp.getText();
                        break;
                    case XmlPullParser.END_TAG:
                        if(inEntry){
                            if("user".equalsIgnoreCase(tagName)){
                                users.add(currentUser);
                                inEntry = false;
                            } else if("name".equalsIgnoreCase(tagName)){
                                currentUser.setName(textValue);
                            } else if("age".equalsIgnoreCase(tagName)){
                                currentUser.setAge(textValue);
                            }
                        }
                        break;
                    default:
                }
                eventType = xpp.next();
            }
        }
        catch (Exception e){
            status = false;
            e.printStackTrace();
        }
        return  status;
    }
}
```
Данный класс выполняет функции парсинга xml. Распарсенные данные будут храниться в переменной users. Непосредственно сам парсинг осуществляется с помощью функции parse. Основную работу выполняет передаваемый в качестве параметра объект XmlPullParser. Этот класс позволяет пробежаться по всему документу xml и получить его содержимое.

Когда данный объект проходит по документу xml, при обнаружении определенного тега он генерирует некоторое событие. Есть четыре события, которые описываются следующими константами:

- START_TAG: открывающий тег элемента

- TEXT: прочитан текст элемента

- END_TAG: закрывающий тег элемента

- END_DOCUMENT: конец документа

С помощью метода getEventType() можно получить первое событие и потом последовательно считывать документ, пока не дойдем до его конца. Когда будет достигнут конец документа, то событие будет представлять константу END_DOCUMENT:

```java
int eventType = xpp.getEventType();
while(eventType != XmlPullParser.END_DOCUMENT){
    //......................
    eventType = xpp.next();
}
```
Для перехода к следующему событию применяется метод next().

При чтении документа с помощью метода getName() можно получить название считываемого элемента.

```java
String tagName = xpp.getName();
```
И в зависимости от названия тега и события мы можем выполнить определенные действия. Например, если это открывающий тег элемента user, то создаем новый объект User и устанавливаем, что мы находимся внутри элемента user:

```java
case XmlPullParser.START_TAG:
    if("user".equalsIgnoreCase(tagName)){
        inEntry = true;
        currentUser = new User();
    }
break;  
```
Если событие TEXT, то считано содержимое элемента, которое мы можем прочитать с помощью метода getText():

```java
case XmlPullParser.TEXT:
    textValue = xpp.getText();
    break;
```
Если закрывающий тег, то все зависит от того, какой элемент прочитан. Если прочитан элемент user, то добавляем объект User в коллекцию ArrayList и сбрываем переменную inEntry, указывая, что мы вышли из элемента user:

```java
case XmlPullParser.END_TAG:
    if(inEntry){
        if("user".equalsIgnoreCase(tagName)){
            users.add(currentUser);
            inEntry = false;
```
Если прочитаны элементы name и age, то передаем их значения переменным name и age объекта User:

```java
else if("name".equalsIgnoreCase(tagName)){
    currentUser.setName(textValue);
} else if("age".equalsIgnoreCase(tagName)){
    currentUser.setAge(textValue);
}
```
Теперь изменим класс MainActivity, который будет загружать ресурс xml:

```java
package com.example.xmlapp;
 
import androidx.appcompat.app.AppCompatActivity;
 
import android.os.Bundle;
import android.util.Log;
 
import org.xmlpull.v1.XmlPullParser;
 
public class MainActivity extends AppCompatActivity {
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
 
        XmlPullParser xpp = getResources().getXml(R.xml.users);
        UserResourceParser parser = new UserResourceParser();
        if(parser.parse(xpp))
        {
            for(User prod: parser.getUsers()){
                Log.d("XML", prod.toString());
            }
        }
    }
}
```
Вначале получаем ресурс xml с помощью метода getXml(), в который передается название ресурса. Данный метод возвращает объект XmlPullParser, который затем используется для парсинга. Для простоты просто выводим данные в окне Logcat:

![](https://metanit.com/java/android/pics/xml2.png)

# Получение xml по сети

Рассмотрим получение данных в формате xml по сети. Допустим, на некотором сайте https://example.com находится файл users.xml со следующим содержимым:

```xml
<?xml version="1.0" encoding="utf-8"?>
<users>
    <user>
        <name>Tom</name>
        <age>36</age>
    </user>
    <user>
        <name>Alice</name>
        <age>32</age>
    </user>
    <user>
        <name>Bob</name>
        <age>28</age>
    </user>
</users>
```
То есть сам файл доступен по адресу https://example.com/users.xml. Но это необязательно должен быть именно файл, это может быть любой ресурс, который динамически генерирует данные в xml.

Возьмем стандартный проект Android и вначале определим в нем класс User, который будет представлять загружаемые данные:

```java
package com.example.xmlapp;
 
public class User {
    private String name;
    private String age;
 
    public String getName(){
        return name;
    }
    public String getAge(){
        return age;
    }
    public void setName(String name){
        this.name = name;
    }
    public void setAge(String age){
        this.age = age;
    }
    public String toString(){
        return  "User: " + name + " - " + age;
    }
}
```
Далее определим класс UserXmlParser:

```java
package com.example.xmlapp;
 
import org.xmlpull.v1.XmlPullParser;
import org.xmlpull.v1.XmlPullParserFactory;
import java.util.ArrayList;
import java.io.StringReader;
 
public class UserXmlParser {
 
    private ArrayList<User> users;
 
    public UserXmlParser(){
        users = new ArrayList<>();
    }
 
    public ArrayList<User> getUsers(){
        return  users;
    }
 
    public boolean parse(String xmlData){
        boolean status = true;
        User currentUser = null;
        boolean inEntry = false;
        String textValue = "";
 
        try{
            XmlPullParserFactory factory = XmlPullParserFactory.newInstance();
            factory.setNamespaceAware(true);
            XmlPullParser xpp = factory.newPullParser();
 
            xpp.setInput(new StringReader(xmlData));
            int eventType = xpp.getEventType();
            while(eventType != XmlPullParser.END_DOCUMENT){
 
                String tagName = xpp.getName();
                switch (eventType){
                    case XmlPullParser.START_TAG:
                        if("user".equalsIgnoreCase(tagName)){
                            inEntry = true;
                            currentUser = new User();
                        }
                        break;
                    case XmlPullParser.TEXT:
                        textValue = xpp.getText();
                        break;
                    case XmlPullParser.END_TAG:
                        if(inEntry){
                            if("user".equalsIgnoreCase(tagName)){
                                users.add(currentUser);
                                inEntry = false;
                            } else if("name".equalsIgnoreCase(tagName)){
                                currentUser.setName(textValue);
                            } else if("age".equalsIgnoreCase(tagName)){
                                currentUser.setAge(textValue);
                            }
                        }
                        break;
                    default:
                }
                eventType = xpp.next();
            }
        }
        catch (Exception e){
            status = false;
            e.printStackTrace();
        }
        return  status;
    }
}
```

То есть в итоге получится следующий проект:

![](https://metanit.com/java/android/pics/xml3.png)

Для парсинга xml здесь используется класс XmlPullParser, который уже рассматривался в прошлой теме. Единственное отличие заключается в том, что для создания объекта этого класса применяется класс XmlPullParserFactory:

```java
XmlPullParserFactory factory = XmlPullParserFactory.newInstance();
XmlPullParser xpp = factory.newPullParser();
```
Для работы определим простейший визуальный интефейс в файле activity_main.xml:

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    <TextView
        android:id="@+id/contentView"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        app:layout_constraintBottom_toTopOf="@id/usersList"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />
    <ListView
        android:id="@+id/usersList"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toBottomOf="@id/contentView" />
 
</androidx.constraintlayout.widget.ConstraintLayout>
```
Здесь определен элемент TextView для отображения некоторой дополнительной информации о состоянии загрузки файла и элемент ListView для отображения загруженных объектов.

Далее изменим класс MainActivity:

```java
package com.example.xmlapp;
 
import androidx.appcompat.app.AppCompatActivity;
 
import android.os.Bundle;
import android.widget.ArrayAdapter;
import android.widget.ListView;
import android.widget.TextView;
 
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.net.URL;
 
import javax.net.ssl.HttpsURLConnection;
 
public class MainActivity extends AppCompatActivity {
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
         
        ListView usersList = findViewById(R.id.usersList);
        TextView contentView = findViewById(R.id.contentView);
         
        contentView.setText("Загрузка...");
        new Thread(new Runnable() {
            public void run() {
                try{
                    String content = download("https://example.com/users.xml");
                    usersList.post(new Runnable() {
                        public void run() {
                            UserXmlParser parser = new UserXmlParser();
                            if(parser.parse(content))
                            {
                                ArrayAdapter<User> adapter = new ArrayAdapter(getBaseContext(),
                                        android.R.layout.simple_list_item_1, parser.getUsers());
                                usersList.setAdapter(adapter);
                                contentView.setText("Загруженно объектов: " + adapter.getCount());
                            }
                        }
                    });
                }
                catch (IOException ex){
                    contentView.post(new Runnable() {
                        public void run() {
                            contentView.setText("Ошибка: " + ex.getMessage());
                        }
                    });
                }
            }
        }).start();
    }
 
    private String download(String urlPath) throws IOException{
        StringBuilder xmlResult = new StringBuilder();
        BufferedReader reader = null;
        InputStream stream = null;
        HttpsURLConnection connection = null;
        try {
            URL url = new URL(urlPath);
            connection = (HttpsURLConnection) url.openConnection();
            stream = connection.getInputStream();
            reader = new BufferedReader(new InputStreamReader(stream));
            String line;
            while ((line=reader.readLine()) != null) {
                xmlResult.append(line);
            }
            return xmlResult.toString();
        } finally {
            if (reader != null) {
                reader.close();
            }
            if (stream != null) {
                stream.close();
            }
            if (connection != null) {
                connection.disconnect();
            }
        }
    }
}
```
При создании MainActivity будет запускаться дополнительный поток, который вызывает метод download(). Этот метод с помощью класса HttpsURLConnection загужает файл users.xml и возвращает его содержимое в виде строки (Если необходимо загрузить файл xml по протоколу http, то вместо применяется класса HttpsURLConnection класс java.net.HttpURLConnection).

```java
String content = download("https://example.com/users.xml");
```
Затем загруженное содержимое передается в метод parse(), класса UserXmlParser, который формирует список объектов.

```java
UserXmlParser parser = new UserXmlParser();
if(parser.parse(content)){
    //...................
```
Затем загруженный список передается в адаптер ArrayAdapter, а через него в ListView для отображения на экране устройства:

```java
ArrayAdapter<User> adapter = new ArrayAdapter(getBaseContext(), android.R.layout.simple_list_item_1, parser.getUsers());
usersList.setAdapter(adapter);
```
В завершении надо добавить в файл манифеста AndroidManifest.xml разрешения на взаимодействие с сетью:

```xml
<uses-permission android:name="android.permission.INTERNET"/>
```
И после запуска приложения в окне Logcat мы увидим полученные с сервера данные:

![](https://metanit.com/java/android/pics/xml6.png)