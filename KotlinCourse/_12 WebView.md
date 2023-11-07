# Работа с сетью. WebView

# WebView

WebView представляет простейший элемент для рендеринга html-кода, базирующийся на движке WebKit. Благодаря этому мы можем использовать WebView как примитивный веб-браузер, просматривая через него контент из сети интернет. Использование движка WebKit гарантирует, что отображение контента будет происходить примерно такжe, как и в других браузерах, построенных на этом движке - Google Chrome и Safari.

Некоторые основные методы класса WebView:

- boolean canGoBack(): возвращает true, если перед текущей веб-страницей в истории навигации WebView еще есть страницы

- boolean canGoForward(): возвращает true, если после текущей веб-страницей в истории навигации WebView еще есть страницы

- void clearCache(boolean includeDiskFiles): очищает кэш WebView

- void clearFormData(): очищает данный автозаполнения полей форм

- void clearHistory(): очищает историю навигации WebView

- String getUrl(): возвращает адрес текущей веб-страницы

- void goBack(): переходит к предыдущей веб-странице в истории навигации

- void goForward(): переходит к следующей веб-странице в истории навигации

- void loadData(String data, String mimeType, String encoding): загружает в веб-браузере данные в виде html-кода, используя указанный mime-тип и кодировку

- void loadDataWithBaseURL (String baseUrl, String data, String mimeType, String encoding, String historyUrl): также загружает в веб-браузере данные в виде html-кода, используя указанный mime-тип и кодировку, как и метод loadData(). Однако кроме того, в качестве первого параметра принимает валидный адрес, с которым ассоциируется загруженные данные.

Зачем нужен этот метод, если есть loadData()? Содержимое, загружаемое методом loadData(), в качестве значения для window.origin будет иметь значение null, и таким образом, источник загружаемого содержимого не сможет пройти проверку на достоверность. Метод loadDataWithBaseURL() с валидными адресами (протокол может быть и HTTP, и HTTPS) позволяет установить источник содержимого.

- void loadUrl(String url): загружает веб-страницу по определенному адресу

- void postUrl(String url, byte[] postData): отправляет данные с помощью запроса типа "POST" по определенному адресу

- void zoomBy(float zoomFactor): изменяет масштаб на опредленный коэффициент

- boolean zoomIn(): увеличивает масштаб

- boolean zoomOut(): уменьшает масштаб

Работать с WebView очень просто. Определим данный элемент в разметке layout:

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
 
    <WebView
        android:id="@+id/webBrowser"
        android:layout_width="0dp"
        android:layout_height="0dp"
 
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"/>
 
</androidx.constraintlayout.widget.ConstraintLayout>
```

Для получения доступа к интернету из приложения, необходимо указать в файле манифеста AndroidManifest.xml соответствующее разрешение:

```xml
<uses-permission android:name="android.permission.INTERNET"/>
```

Для получения доступа к интернету из приложения, необходимо указать в файле манифеста AndroidManifest.xml соответствующее разрешение:

```java
package com.example.viewapp;
 
import androidx.appcompat.app.AppCompatActivity;
 
import android.os.Bundle;
import android.webkit.WebView;
 
public class MainActivity extends AppCompatActivity {
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
 
        WebView browser=findViewById(R.id.webBrowser);
        browser.loadUrl("https://metanit.com");
    }
}
```

![](https://metanit.com/java/android/pics/webview.png)

Вместо определения элемента в layout мы можем создать WebView в коде Activity:

```java
WebView browser = new WebView(this);
setContentView(browser);
browser.loadUrl("http://metanit.com");
```
Кроме загрузки конкретной страницы из интернета с помощью метод loadData():

```java
WebView browser= findViewById(R.id.webBrowser);
browser.loadData("<html><body><h2>Hello, Android!</h2></body></html>", "text/html", "UTF-8");
```

Первым параметром метод принимает строку кода html, во втором - тип содержимого, а в третьем - кодировку.


![](https://metanit.com/java/android/pics/webview2.png)


# JavaScript

По умолчанию в WebView отключен javascript, чтобы его включить надо применить метод setJavaScriptEnabled(true) объекта WebSettings:

```java
import android.webkit.WebSettings;
//.....................................
WebView browser = findViewById(R.id.webBrowser);
WebSettings webSettings = browser.getSettings();
webSettings.setJavaScriptEnabled(true);
```

# Загрузка данных и класс HttpURLConnection

На сегодняшний день если не все, то большинство Android-устройств имеют доступ к сети интернет. А большое количество мобильных приложений так или иначе взаимодействуют с средой интернет: загружают файлы, авторизуются и получают информацию с внешних веб-сервисов и т.д. Рассмотрим, как мы можем использовать в своем приложении доступ к сети интернет.

Среди стандартных элементов нам доступен виджет WebView, который может загружать контент с определенного url-адреса. Но этим возможности работы с сетью в Android не ограничиваются. Для получения данных с определенного интернет-ресурса мы можем использовать классы HttpUrlConnection (для протокола HTTP) и HttpsUrlConnection (для протокола HTTPS) из стандартной библиотеки Java.

Итак, создадим новый проект с пустой MainActivity. Первым делом для работы с сетью нам надо установить в файле манифеста AndroidManifest.xml соответствующее разрешение:

```xml
<uses-permission android:name="android.permission.INTERNET"/>
```

В файле activity_main.xml, который представляет разметку для MainActivity, определим следующий код:

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
 
    <Button
        android:id="@+id/downloadBtn"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="Загрузка"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />
    <WebView
        android:id="@+id/webView"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toBottomOf="@id/downloadBtn"
        app:layout_constraintBottom_toTopOf="@id/scrollView" />
    <ScrollView
        android:id="@+id/scrollView"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toBottomOf="@id/webView"
        app:layout_constraintBottom_toBottomOf="parent">
 
        <TextView android:id="@+id/content"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />
    </ScrollView>
 
</androidx.constraintlayout.widget.ConstraintLayout>
```
Здесь определена кнопка для загрузки данных, а сами данные для примера загружаются одновременно в виде строки в текстовое поле и в элемент WebView. Так как данных может быть очень много, то текстовое поле помещено в элемент ScrollView.

Поскольку загрузка данных может занять некоторое время, то обращение к интернет-ресурсу определим в отдельном потоке и для этого изменим код MainActivity следующим образом:

```java
package com.example.httpapp;
 
import androidx.appcompat.app.AppCompatActivity;
 
import android.os.Bundle;
import android.view.View;
import android.webkit.WebView;
import android.widget.Button;
import android.widget.TextView;
import android.widget.Toast;
 
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
 
        TextView contentView = findViewById(R.id.content);
        WebView webView = findViewById(R.id.webView);
        webView.getSettings().setJavaScriptEnabled(true);
        Button btnFetch = findViewById(R.id.downloadBtn);
        btnFetch.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                contentView.setText("Загрузка...");
                new Thread(new Runnable() {
                    public void run() {
                        try{
                            String content = getContent("https://stackoverflow.com/");
                            webView.post(new Runnable() {
                                public void run() {
                                    webView.loadDataWithBaseURL("https://stackoverflow.com/",content, "text/html", "UTF-8", "https://stackoverflow.com/");
                                    Toast.makeText(getApplicationContext(), "Данные загружены", Toast.LENGTH_SHORT).show();
                                }
                            });
                            contentView.post(new Runnable() {
                                public void run() {
                                    contentView.setText(content);
                                }
                            });
                        }
                        catch (IOException ex){
                            contentView.post(new Runnable() {
                                public void run() {
                                    contentView.setText("Ошибка: " + ex.getMessage());
                                    Toast.makeText(getApplicationContext(), "Ошибка", Toast.LENGTH_SHORT).show();
                                }
                            });
                        }
                    }
                }).start();
            }
        });
    }
    private String getContent(String path) throws IOException {
        BufferedReader reader=null;
        InputStream stream = null;
        HttpsURLConnection connection = null;
        try {
            URL url=new URL(path);
            connection =(HttpsURLConnection)url.openConnection();
            connection.setRequestMethod("GET");
            connection.setReadTimeout(10000);
            connection.connect();
            stream = connection.getInputStream();
            reader= new BufferedReader(new InputStreamReader(stream));
            StringBuilder buf=new StringBuilder();
            String line;
            while ((line=reader.readLine()) != null) {
                buf.append(line).append("\n");
            }
            return(buf.toString());
        }
        finally {
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

Непосредственно для самой загрузки определен метод getContent(), который будет загружать веб-страницу с помощью класса HttpsURLConnection и возвращать код загруженной страницы в виде строки.

Вначале создается элемент HttpsURLConnection:

```java
URL url=new URL(path);
connection =(HttpsURLConnection)url.openConnection();
connection.setRequestMethod("GET"); // установка метода получения данных -GET
connection.setReadTimeout(10000); // установка таймаута перед выполнением - 10 000 миллисекунд
connection.connect(); // подключаемся к ресурсу
```
После подключение происходит считывание со входного потока:

```java
stream = connection.getInputStream();
reader= new BufferedReader(new InputStreamReader(stream));
```
Используя входной поток, мы можем считать его в строку.

Этот метод getContent() затем будет вызываться в обработчике нажатия кнопки:

```java
Button btnFetch = (Button)findViewById(R.id.downloadBtn);
btnFetch.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        contentView.setText("Загрузка...");
        new Thread(new Runnable() {
            public void run() {
                try{
                    String content = getContent("https://stackoverflow.com/");
```

Поскольку загрузка может занять долгое время, то метод getContent() в отдельном потоке с помощью объектов Thread и Runnable. Для примера в данном случае обращение идет к ресурсу "https://stackoverflow.com/".

Запустим приложение и нажмем на кнопку. И при наличии интернета приложение загрузит гравную страницу с "https://stackoverflow.com/" и отобразит ее в WebView и TextView:

![](https://metanit.com/java/android/pics/http1.png)

Конечно, данный способ вряд ли подходит для просмотра интернет-страниц, однако таким образом, мы можем получать какие-либо данные (не интернет-страницы) от различных веб-сервисов, например, в формате xml или json (например, различные курсы валют, показатели погоды), используя специальные api, и затем после обработки показывать их пользователю.

