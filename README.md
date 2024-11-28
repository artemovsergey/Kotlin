# Kotlin Jetpack Compose

# Stateless View

```kt
package com.example.ordernow.ui.components

import androidx.compose.material3.Button
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.getValue
import androidx.compose.runtime.setValue
import androidx.compose.ui.tooling.preview.Preview
import androidx.hilt.navigation.compose.hiltViewModel
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewmodel.compose.viewModel
import dagger.hilt.android.lifecycle.HiltViewModel
import javax.inject.Inject

@HiltViewModel
class CounterViewModel @Inject constructor() : ViewModel(){

    // определение состояния в модели представления
    var count by mutableStateOf(0)

    val text: String = ""

    fun pressButton():String {
        return "Количество: $count"
    }
}


@Composable
fun CounteStateLess(viewModel: CounterViewModel = hiltViewModel()){
    Button(onClick = { viewModel.count += 1}) {
        Text(text = viewModel.pressButton() )
    }
}

@Composable
@Preview
fun CounteStateLessPreview(){
    CounteStateLess()
}
```

# Statefull View

```kt
package com.example.ordernow.ui.components


import androidx.compose.material3.Button
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable

import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.getValue
import androidx.compose.runtime.setValue
import androidx.compose.ui.tooling.preview.Preview


@Composable
fun Counter(){

    var count by remember { mutableStateOf(0) }

    Button(onClick = { count += 1}) {
        Text(text = "Количество: ${count}")
    }

}

@Composable
@Preview
fun CounterPreview(){
    Counter()
}
```


# Presentation

## MainActivity

```kt
package com.example.ordernow.main

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.runtime.Composable
import androidx.compose.ui.tooling.preview.Preview
import dagger.hilt.android.AndroidEntryPoint

@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            OrderNowScreen()
        }
    }
}


@Preview
@Composable
fun OrderNowScreenPreview(){
    OrderNowScreen()
}
```

# MainApplication

```kt
package com.example.ordernow.main

import android.app.Application
import dagger.hilt.android.HiltAndroidApp

@HiltAndroidApp
class OrderNowApplication: Application()
```

# ApplicationScreen

```kt
package com.example.ordernow.main

import androidx.compose.material.Scaffold
import androidx.compose.runtime.Composable
import androidx.compose.ui.tooling.preview.Preview
import com.example.ordernow.common.navigation.OrderNowNavigationHost
import com.example.ordernow.ui.patterns.OrderNowBottomBar
import com.example.ordernow.ui.patterns.OrderNowTopBar



@Composable
fun OrderNowScreen() {

            val appState = rememberAppState()

            Scaffold(
                scaffoldState = appState.scaffoldState,
                topBar = { OrderNowTopBar(appState) },
                bottomBar = { OrderNowBottomBar(navController = appState.navController) }
            ) { contentPadding ->
                OrderNowNavigationHost(appState = appState, contentPadding)
            }
        }

@Preview(showSystemUi = true)
@Composable
fun OrderNowScreenPreview1(){
    OrderNowScreen()
}
```

# ApplicationState

```kt
package com.example.ordernow.main

import android.content.res.Resources
import androidx.compose.material.ScaffoldState
import androidx.compose.material.rememberScaffoldState
import androidx.compose.runtime.Composable
import androidx.compose.runtime.ReadOnlyComposable
import androidx.compose.runtime.remember
import androidx.compose.runtime.rememberCoroutineScope
import androidx.compose.ui.platform.LocalConfiguration
import androidx.compose.ui.platform.LocalContext
import androidx.navigation.NavHostController
import androidx.navigation.compose.currentBackStackEntryAsState
import androidx.navigation.compose.rememberNavController
import com.example.ordernow.common.navigation.OrderNowScreenRoute
import com.example.ordernow.domain.models.Category
import com.example.ordernow.domain.models.Product
import com.example.ordernow.domain.models.order.Order
import com.example.ordernow.domain.models.order.SummaryTotals
import kotlinx.coroutines.CoroutineScope

@Composable
fun rememberAppState(
    scaffoldState: ScaffoldState = rememberScaffoldState(),
    navController: NavHostController = rememberNavController(),
    resources: Resources = resources(),
    coroutineScope: CoroutineScope = rememberCoroutineScope()
) = remember(
    scaffoldState,
    navController,
    resources,
    coroutineScope
) {
    OrderNowState(scaffoldState, navController, resources, coroutineScope)
}

class OrderNowState(
    val scaffoldState: ScaffoldState,
    val navController: NavHostController,
    private val resources: Resources,
    coroutineScope: CoroutineScope
){
    private val screensWithArrowBack = OrderNowScreenRoute.withArrowBack.map { it.route }

    val shouldShowArrowBack: Boolean
        @Composable get() = navController
            .currentBackStackEntryAsState().value?.destination?.route in screensWithArrowBack

    lateinit var categorySelected: Category
    lateinit var productSelected: Product
    lateinit var summaryTotals: SummaryTotals
    lateinit var orderSelected: Order
}


@Composable
@ReadOnlyComposable
fun resources(): Resources {
    LocalConfiguration.current
    return LocalContext.current.resources
}
```


# Domain

## Models

```kotlin
package com.example.ordernow.domain.models

data class Product(
    val id: String,
    val name: String,
    val description: String? = null,
    val image: String,
    val price: Double,
    val category: Category
)
```

## Interfaces

```kotlin
package com.example.ordernow.domain.interfaces

import com.example.ordernow.domain.models.Category
import com.example.ordernow.domain.models.Product
import kotlinx.coroutines.flow.Flow

interface IProductRepository {
    fun getProducts(): Flow<List<Product>>
    fun getProducts(category: Category): Flow<List<Product>>
}
```

# Repository

```kt
package com.example.ordernow.data.datasources.impl.mock

import com.example.ordernow.domain.interfaces.IProductRepository
import com.example.ordernow.domain.models.Category
import com.example.ordernow.domain.models.Product
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.flowOf

class ProductMocked : IProductRepository {

    val products: List<Product> =
        listOf(
            DataMocked.product1,
            DataMocked.product2,
            DataMocked.product3
        )

    override fun getProducts(): Flow<List<Product>> {
        return flowOf(products)
    }

    override fun getProducts(category: Category): Flow<List<Product>> {
        return flowOf(products.filter { it.category.id == category.id })
    }
}
```

# DI

```kt
package com.example.ordernow.common.di

import com.example.ordernow.data.datasources.ICategoryRepository
import com.example.ordernow.data.datasources.IPaymentRepository

import com.example.ordernow.data.datasources.impl.mock.CartMocked
import com.example.ordernow.data.datasources.impl.mock.CategoryMocked
import com.example.ordernow.data.datasources.impl.mock.PaymentMocked
import com.example.ordernow.data.datasources.impl.mock.ProductMocked
import com.example.ordernow.domain.interfaces.ICartRepository
import com.example.ordernow.domain.interfaces.IProductRepository
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.components.SingletonComponent
import javax.inject.Singleton

@Module
@InstallIn(SingletonComponent::class)
object AppModule {

    @Provides
    @Singleton
    fun provideProductRepository(): IProductRepository {
        return ProductMocked()
    }

    @Provides
    @Singleton
    fun provideCategoryRepository(): ICategoryRepository {
        return CategoryMocked()
    }

    @Provides
    @Singleton
    fun provideCartRepository(): ICartRepository {
        return CartMocked()
    }

    @Provides
    @Singleton
    fun providePaymentRepository(): IPaymentRepository {
        return PaymentMocked()
    }

}
```

# NavigationHost

```kt
package com.example.ordernow.common.navigation

import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.PaddingValues
import androidx.compose.foundation.layout.padding

import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.tooling.preview.Preview
import androidx.hilt.navigation.compose.hiltViewModel
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import com.example.ordernow.domain.models.Category
import com.example.ordernow.domain.models.Product
import com.example.ordernow.domain.models.order.Order
import com.example.ordernow.domain.models.order.SummaryTotals
import com.example.ordernow.main.OrderNowState
import com.example.ordernow.main.rememberAppState
import com.example.ordernow.ui.features.cart.CartScreen
import com.example.ordernow.ui.features.checkout.CheckoutScreen
import com.example.ordernow.ui.features.home.HomeScreen
import com.example.ordernow.ui.features.home.HomeViewModel
import com.example.ordernow.ui.features.placeorder.PlaceOrderScreen
import com.example.ordernow.ui.features.productdetail.ProductDetailScreen
import com.example.ordernow.ui.features.productlist.ProductListScreen

@Composable
fun OrderNowNavigationHost(
    appState: OrderNowState,
    contentPadding: PaddingValues
) {

    val vm  = hiltViewModel<HomeViewModel>()

    NavHost(navController = appState.navController,
        startDestination = NavigationBarSection.Home.route,
        modifier = Modifier.padding(contentPadding)) {

        val homeRoute = OrderNowScreenRoute.Home.route
        val listRoute = OrderNowScreenRoute.ProductList.route
        val detailRoute = OrderNowScreenRoute.ProductDetail.route
        val cartRoute = OrderNowScreenRoute.Cart.route
        val checkout = OrderNowScreenRoute.Checkout.route
        val placeOrder = OrderNowScreenRoute.PlaceOrder.route

        val productSelectedInHome: (Product) -> Unit = { product: Product ->
            appState.productSelected = product
            appState.navigateSaved(detailRoute, homeRoute)
        }
        val categorySelectedInHome: (Category) -> Unit = { category: Category ->
            appState.categorySelected = category
            appState.navigateSaved(listRoute, homeRoute)
        }
        val productSelectedInList: (Product) -> Unit = { product: Product ->
            appState.productSelected = product
            appState.navigateSaved(detailRoute, listRoute)
        }

        val goToCartScreen: () -> Unit = {
            appState.navigateSaved(cartRoute, detailRoute)
        }

        val goToCheckoutScreen: (SummaryTotals) -> Unit = { summary ->
            appState.summaryTotals = summary
            appState.navigateSaved(checkout, cartRoute)
        }

        val goToPlaceOrderScreen: (Order) -> Unit = { order ->
            appState.orderSelected = order
            appState.navigateSaved(placeOrder, checkout)
        }

        // Home Screen Graph
        composable(NavigationBarSection.Home.route) {
            HomeScreen(
                categorySelected = categorySelectedInHome,
                productSelected = productSelectedInHome
            )
        }

        // Cart Screen Graph
        composable(NavigationBarSection.Cart.route) {
            CartScreen(
                goToCheckout = goToCheckoutScreen
            )
        }

        // Product List Screen Graph
        composable(OrderNowScreenRoute.ProductList.route) {
            ProductListScreen(
                category = appState.categorySelected,
                productSelected = productSelectedInList
            )
        }

        // Product Detail Screen Graph
        composable(OrderNowScreenRoute.ProductDetail.route) {
            ProductDetailScreen(
                product = appState.productSelected,
                goToCart = goToCartScreen
            )
        }

        // Checkout Screen Graph
        composable(OrderNowScreenRoute.Checkout.route) {
            CheckoutScreen(
                summary = appState.summaryTotals,
                goToPlaceOrder = goToPlaceOrderScreen
            )
        }

        // Process Order Screen Graph
        composable(OrderNowScreenRoute.PlaceOrder.route) {
            PlaceOrderScreen(
                order = appState.orderSelected
            )
        }
    }
}


@Preview(showSystemUi = true)
@Composable
fun OrderNowNavigationHostPreview(){
    val appstate = rememberAppState();
    Column(){
        OrderNowNavigationHost(appstate, PaddingValues())
    }

}
```


# NavigationScreenRoute

```kt
package com.example.ordernow.common.navigation

sealed class OrderNowScreenRoute (val route: String) {

    companion object {
        val withArrowBack = listOf(
            ProductDetail,
            Checkout,
            PlaceOrder
        )
    }

    object Home : OrderNowScreenRoute("home")
    object Cart : OrderNowScreenRoute("cart")
    object ProductList : OrderNowScreenRoute("product_list")
    object ProductDetail : OrderNowScreenRoute("product_detail")
    object Checkout : OrderNowScreenRoute("checkout")
    object PlaceOrder : OrderNowScreenRoute("place_order")
}
```

# NavigationBarSection

```kt
package com.example.ordernow.common.navigation

import androidx.annotation.StringRes
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Home
import androidx.compose.material.icons.filled.ShoppingCart
import androidx.compose.ui.graphics.vector.ImageVector
import com.example.ordernow.R.string as AppText

sealed class NavigationBarSection(
    @StringRes val title: Int,
    val icon: ImageVector,
    val route: String
) {
    companion object {
        val sections = listOf(
            Home,
            Cart
        )
    }

    object Home : NavigationBarSection(
        title = AppText.home,
        icon = Icons.Filled.Home,
        route = "home"
    )

    object Cart : NavigationBarSection(
        title = AppText.cart,
        icon = Icons.Filled.ShoppingCart,
        route = "cart"
    )
}
```

# NavigationState

```kt
package com.example.ordernow.common.navigation

import com.example.ordernow.main.OrderNowState


fun OrderNowState.popUp(): () -> Unit = {
    navController.popBackStack()
}

fun OrderNowState.navigate(route: String) {
    navController.navigate(route) {
        launchSingleTop = true
    }
}

fun OrderNowState.navigateAndPopUp(route: String, popUp: String) {
    navController.navigate(route) {
        launchSingleTop = true
        popUpTo(popUp) { inclusive = true }
    }
}

fun OrderNowState.navigateSaved(route: String, popUp: String) {
    navController.navigate(route) {
        launchSingleTop = true
        restoreState = true
        popUpTo(popUp) { saveState = true }
    }
}

fun OrderNowState.clearAndNavigate(route: String) {
    navController.navigate(route) {
        launchSingleTop = true
        popUpTo(0) { inclusive = true }
    }
}
```


# build.gradle(app)

```
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.jetbrains.kotlin.android)

    id("kotlin-kapt")
    id("com.google.dagger.hilt.android")
}

android {
    namespace = "com.example.ordernow"
    compileSdk = 35

    defaultConfig {
        applicationId = "com.example.ordernow"
        minSdk = 29
        targetSdk = 35
        versionCode = 1
        versionName = "1.0"

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
        vectorDrawables {
            useSupportLibrary = true
        }
    }

    buildTypes {
        release {
            isMinifyEnabled = false
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_1_8
        targetCompatibility = JavaVersion.VERSION_1_8
    }
    kotlinOptions {
        jvmTarget = "1.8"
    }
    buildFeatures {
        compose = true
    }
    composeOptions {
        kotlinCompilerExtensionVersion = "1.5.1"
    }
    packaging {
        resources {
            excludes += "/META-INF/{AL2.0,LGPL2.1}"
        }
    }
}

dependencies {
    
    implementation("androidx.constraintlayout:constraintlayout-compose:1.0.1")
    // Работа с изображениями

    implementation("io.coil-kt:coil-compose:2.0.0")

    implementation("androidx.compose.material:material:1.4.2")

    // Navigation
    implementation("androidx.hilt:hilt-navigation-compose:1.0.0")
    implementation("androidx.navigation:navigation-compose:2.5.1")

    // Di
    implementation("com.google.dagger:hilt-android:2.51.1")
    implementation(libs.litert.support.api)

    kapt("com.google.dagger:hilt-android-compiler:2.51.1")

    // Viewmodels
    implementation("androidx.lifecycle:lifecycle-viewmodel-ktx:2.5.0")
    
    // ViewModel utilities for Compose
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.5.0")

    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.lifecycle.runtime.ktx)
    implementation(libs.androidx.activity.compose)
    implementation(platform(libs.androidx.compose.bom))
    implementation(libs.androidx.ui)
    implementation(libs.androidx.ui.graphics)
    implementation(libs.androidx.ui.tooling.preview)
    implementation(libs.androidx.material3)
    testImplementation(libs.junit)
    androidTestImplementation(libs.androidx.junit)
    androidTestImplementation(libs.androidx.espresso.core)
    androidTestImplementation(platform(libs.androidx.compose.bom))
    androidTestImplementation(libs.androidx.ui.test.junit4)
    debugImplementation(libs.androidx.ui.tooling)
    debugImplementation(libs.androidx.ui.test.manifest)

}

kapt {
    correctErrorTypes = true
}
```

# build.gradle(project)

```
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.jetbrains.kotlin.android) apply false
    id("com.google.dagger.hilt.android") version "2.51.1" apply false
}
```

# Разрешение путей на кириллице

gradle.properties(project)
```
android.overridePathCheck=true
```

# TestData

```kotlin
package com.example.ordernow.data.datasources.impl.mock

import com.example.ordernow.domain.models.CartItem
import com.example.ordernow.domain.models.Category
import com.example.ordernow.domain.models.Product

class DataMocked {

    companion object Data {

        // Categories data mocked
        val category1 = Category(
            "100",
            "Clothes",
            "Clothes",
            "https://storage.googleapis.com/order_now_store_bucket/category-list/category_1.png"
        )
        val category2 = Category(
            "101",
            "Technology",
            "Technology",
            "https://storage.googleapis.com/order_now_store_bucket/category-list/category_2.png"
        )
        val category3 = Category(
            "102",
            "Furniture",
            "Furniture",
            "https://storage.googleapis.com/order_now_store_bucket/category-list/category_3.png"
        )
        val category4 = Category(
            "103",
            "Accessories",
            "Accessories",
            "https://storage.googleapis.com/order_now_store_bucket/category-list/category_4.png"
        )

        // Products data mocked
        val product1 = Product(
            "11",
            "Neck T-Shirt",
            "Price and other details may vary based on product size and color.",
            "https://storage.googleapis.com/order_now_store_bucket/product-list/product_1.png",
            16.0,
            category1
        )
        val product2 = Product(
            "21",
            "iPhone Black",
            "Personal setup available. Buy now with free delivery.",
            "https://storage.googleapis.com/order_now_store_bucket/product-list/product_2.png",
            200.5,
            category2
        )
        val product3 = Product(
            "31",
            "iWatch SE",
            "Personal setup available. Buy now with free delivery.",
            "https://storage.googleapis.com/order_now_store_bucket/product-list/product_3.png",
            130.5,
            category4
        )
        val product4 = Product(
            "41",
            "Gaming Headset",
            "Personal setup available. Buy now with free delivery.",
            "https://storage.googleapis.com/order_now_store_bucket/product-list/product_4.png",
            12.5,
            category2
        )
        val product5 = Product(
            "51",
            "Padded Jacket",
            "Price and other details may vary based on product size and color.",
            "https://storage.googleapis.com/order_now_store_bucket/product-list/product_5.png",
            78.0,
            category1
        )
        val product6 = Product(
            "61",
            "Solid Backpack",
            "Buy now with free delivery.",
            "https://storage.googleapis.com/order_now_store_bucket/product-list/product_6.png",
            27.7,
            category4
        )
        val product7 = Product(
            "71",
            "Fit Sport Shorts",
            "Price and other details may vary based on product size and color.",
            "https://storage.googleapis.com/order_now_store_bucket/product-list/product_7.png",
            35.0,
            category1
        )
        val product8 = Product(
            "81",
            "Interior Chair",
            "Buy now with free delivery.",
            "https://storage.googleapis.com/order_now_store_bucket/product-list/product_8.png",
            30.5,
            category3
        )
        val product9 = Product(
            "91",
            "AirPods Apple",
            "3 Generation. Personal setup available. Buy now with free delivery.",
            "https://storage.googleapis.com/order_now_store_bucket/product-list/product_9.png",
            200.5,
            category2
        )
        val product10 = Product(
            "101",
            "Case for MacBook Pro",
            "13 HardShell. Buy now with free delivery.",
            "https://storage.googleapis.com/order_now_store_bucket/product-list/product_10.png",
            12.5,
            category4
        )
        val product11 = Product(
            "201",
            "Case for MacBook Pro",
            "13 HardShell. Personal setup available. Buy now with free delivery.",
            "https://storage.googleapis.com/order_now_store_bucket/product-list/product_11.png",
            100.5,
            category2
        )
        val product12 = Product(
            "301",
            "Thermostat",
            "Digital Ecologic. Personal setup available. Buy now with free delivery.",
            "https://storage.googleapis.com/order_now_store_bucket/product-list/product_12.png",
            50.5,
            category3
        )
        val product13 = Product(
            "401",
            "Top Sneakers",
            "Price and other details may vary based on product size and color.",
            "https://storage.googleapis.com/order_now_store_bucket/product-list/product_13.png",
            20.9,
            category1
        )
        val product14 = Product(
            "501",
            "Jacket Men",
            "Brown Slit. Price and other details may vary based on product size and color.",
            "https://storage.googleapis.com/order_now_store_bucket/product-list/product_14.png",
            30.2,
            category1
        )
        val product15 = Product(
            "601",
            "H&M Polo Shirt",
            "Slim Fit. Price and other details may vary based on product size and color.",
            "https://storage.googleapis.com/order_now_store_bucket/product-list/product_15.png",
            40.3,
            category1
        )
        // CartItem data mocked
        val cartItem1 = CartItem(
            1,
            product1
        )
        val cartItem2 = CartItem(
            3,
            product2
        )
        val cartItem3 = CartItem(
            2,
            product3
        )
    }
}
```


