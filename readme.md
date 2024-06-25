# Kotlin Jetpack Compose

# Ресурсы

- https://github.com/android/compose-samples

# MainActivity.kt

```kt
package com.example.kotlinexample

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.tooling.preview.Preview
import com.example.kotlinexample.ui.theme.KotlinExampleTheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
           UserInfo(name = "Сергей", age = 35 )
        }
    }
}

@Composable
fun UserInfo(name: String, age: Int) {
    Text(
        text = "Здравствуйте,  $name. Ваш возраст:  $age лет"
    )
}

@Preview(showBackground = true)
@Composable
fun GreetingPreview() {
    UserInfo(name = "Сергей", age = 35 )
}
```
