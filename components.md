# Components

## Button

```kt
@Composable
fun StandardButton(
    text: String,
    onClicked: () -> Unit,
    enabled: Boolean = true,
    elevation: ButtonElevation? = null
) {
    Button(
        onClick = onClicked,
        enabled = enabled,
        colors = ButtonDefaults.buttonColors(backgroundColor = orange),
        elevation = elevation,
        shape = RoundedCornerShape(14.dp),
        modifier = Modifier
            .height(60.dp)
            .fillMaxWidth(),

    ) {
        Text(
            text = text,
            color = MaterialTheme.colors.primary,
            style = MaterialTheme.typography.button
        )
    }
}

@Preview
@Composable
fun PreviewStandardButton() {
    StandardButton(
        text = "Standard Button",
        onClicked = {}
    )
}
```

# ProductCard

```kt
@Composable
fun ProductCard(
    product: Product,
    productSelected: (product: Product) -> Unit
) {
    Card(
        modifier = Modifier
            .width(180.dp)
            .wrapContentHeight(),
        shape = RoundedCornerShape(24.dp),
        elevation = 4.dp
    ) {

        Column(
            modifier = Modifier
                .clickable { productSelected(product) }
                .fillMaxWidth()
                .wrapContentHeight()
                .padding(8.dp)
        ) {
            IconButton(
                onClick = { },
                modifier = Modifier.align(Alignment.End)
            ) {
                Icon(
                    imageVector = Icons.Outlined.FavoriteBorder,
                    contentDescription = "",
                    tint = orange
                )
            }
            Box(
                modifier = Modifier
                    .fillMaxWidth()
                    .wrapContentSize(),
                contentAlignment = Alignment.Center
            ) {
                AsyncImage(
                    model = ImageRequest.Builder(LocalContext.current)
                        .data(product.image)
                        .crossfade(true)
                        .build(),
                    placeholder = painterResource(R.drawable.product_placeholder),
                    contentDescription = "",
                    modifier = Modifier.size(100.dp),
                )
            }

            Column(
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(top = 10.dp),
                horizontalAlignment = Alignment.CenterHorizontally,
                verticalArrangement = Arrangement.Center
            ) {

                Text(
                    text = product.name,
                    fontWeight = FontWeight.Bold,
                    fontSize = 16.sp,
                    color = MaterialTheme.colors.primary
                )

                Text(
                    text = buildAnnotatedString {
                        withStyle(
                            style = SpanStyle(
                                orange,
                                fontWeight = FontWeight.Bold
                            )
                        ) {
                            append("$ " + product.price)
                        }
                    },
                    style = MaterialTheme.typography.subtitle1,
                    modifier = Modifier,
                    fontSize = 16.sp
                )
            }
        }
    }
}

@Preview
@Composable
fun PreviewProductSectionCard() {
    ProductCard(
        product = PreviewData.product,
        productSelected = {}
    )
}
```

# SummaryCard

```kt
@Composable
fun SummaryCard(
    numberItems: Int,
    totalItems: Double,
    shippingCost: Double,
    taxCost: Double,
    total: Double,
    modifier: Modifier = Modifier
) {
    Card(
        shape = RoundedCornerShape(15.dp),
        elevation = 0.dp,
        modifier = modifier
    ) {
        Summary(
            numberItems = numberItems,
            totalItems = totalItems,
            shippingCost = shippingCost,
            taxCost = taxCost,
            total = total
        )
    }
}

@Preview
@Composable
fun PreviewSummaryCard() {
    SummaryCard(
        numberItems = 3,
        totalItems = 12.00,
        shippingCost = 10.00,
        taxCost = 2.00,
        total = 24.00
    )
}
```

# CategoryCard

```kt
@Composable
fun CategoryCard(
    category: Category,
    categorySelected: (category: Category) -> Unit
) {
    Card(
        modifier = Modifier
            .width(120.dp)
            .wrapContentHeight(),
        shape = RoundedCornerShape(20.dp),
        elevation = 4.dp
    ) {

        Column(
            modifier = Modifier
                .clickable { categorySelected(category) }
                .fillMaxWidth()
                .wrapContentHeight()
                .padding(8.dp),
            horizontalAlignment = Alignment.CenterHorizontally,
            verticalArrangement = Arrangement.Center
        ) {
            AsyncImage(
                model = ImageRequest.Builder(LocalContext.current)
                    .data(category.image)
                    .crossfade(true)
                    .build(),
                placeholder = painterResource(R.drawable.product_placeholder),
                contentDescription = "",
                modifier = Modifier.size(60.dp),
            )
            Text(
                modifier = Modifier
                    .padding(
                        top = 4.dp,
                        bottom = 8.dp
                    )
                    .align(Alignment.CenterHorizontally),
                text = category.name
            )
        }
    }
}

@Preview
@Composable
fun CategoryCardPreview() {
    CategoryCard(
        category = PreviewData.category,
        categorySelected = {}
    )
}
```


# ProductItem

```kt
@Composable
fun ProductItem(
    product: Product,
    productSelected: (Product) -> Unit
) {
    Row(
        modifier = Modifier
            .clickable { productSelected(product) }
            .fillMaxWidth()
            .wrapContentHeight(),
        horizontalArrangement = Arrangement.SpaceBetween,
        verticalAlignment = Alignment.CenterVertically
    ) {
        Box(
            modifier = Modifier
                .padding(10.dp)
                .width(120.dp)
                .height(120.dp)
                .fillMaxWidth(0.2f)
                .clip(RoundedCornerShape(20.dp))
                .background(white),
            contentAlignment = Alignment.Center
        ) {
            AsyncImage(
                model = ImageRequest.Builder(LocalContext.current)
                    .data(product.image)
                    .crossfade(true)
                    .build(),
                placeholder = painterResource(R.drawable.product_placeholder),
                contentDescription = "",
                modifier = Modifier.size(100.dp),
            )
        }
        Column(
            modifier = Modifier
                .fillMaxWidth(0.8f)
        ) {

            Text(
                text = product.name,
                fontWeight = FontWeight.Bold,
                fontSize = 16.sp,
                color = titleTextColor
            )
            Spacer(modifier = Modifier.height(10.dp))
            Text(
                text = buildAnnotatedString {
                    withStyle(
                        style = SpanStyle(
                            orange,
                            fontWeight = FontWeight.Bold
                        )
                    ) {
                        append("$ " + product.price)
                    }
                },
                style = MaterialTheme.typography.subtitle1,
                modifier = Modifier,
                fontSize = 16.sp
            )
        }
        Box(
            modifier = Modifier
                .padding(top = 20.dp, end = 20.dp)
                .align(Alignment.Top)
        ) {
            Icon(
                imageVector = Icons.Outlined.FavoriteBorder,
                contentDescription = "",
                tint = orange
            )
        }
    }
}

@Preview
@Composable
fun PreviewProductItem() {
    ProductItem(
        product = PreviewData.product,
        productSelected = {}
    )
}
```

# CartViewItem

```kt
@Composable
fun CartViewItem(
    cartItem: CartItem,
    removeItem: (CartItem) -> Unit,
    count: Int,
    decreaseItemCount: () -> Unit,
    increaseItemCount: () -> Unit,
    backgroundColor: Color = Color.Transparent
) {
    Column {
        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.SpaceBetween,
            verticalAlignment = Alignment.CenterVertically
        ) {
            Box(
                modifier = Modifier
                    .width(100.dp)
                    .height(100.dp)
                    .fillMaxWidth(0.2f)
                    .clip(RoundedCornerShape(20.dp))
                    .background(backgroundColor),
                contentAlignment = Alignment.Center
            ) {
                AsyncImage(
                    model = ImageRequest.Builder(LocalContext.current)
                        .data(cartItem.product.image)
                        .crossfade(true)
                        .build(),
                    placeholder = painterResource(R.drawable.product_placeholder),
                    contentDescription = null,
                    modifier = Modifier.padding(8.dp),
                )
            }
            Column(
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(horizontal = 16.dp),
                horizontalAlignment = Alignment.Start,
                verticalArrangement = Arrangement.SpaceEvenly
            ) {
                Text(
                    text = cartItem.product.name,
                    fontSize = 18.sp,
                    color = titleTextColor,
                    fontWeight = FontWeight.Bold
                )
                Spacer(modifier = Modifier.height(10.dp))

                Row(
                    modifier = Modifier.fillMaxWidth(),
                    horizontalArrangement = Arrangement.SpaceBetween,
                    verticalAlignment = Alignment.CenterVertically
                ) {
                    Text(
                        text = buildAnnotatedString {
                            withStyle(
                                style = SpanStyle(
                                    titleTextColor
                                )
                            ) {
                                append("$ " + cartItem.product.price + "/per item")
                            }
                        },
                        style = MaterialTheme.typography.subtitle1,
                        modifier = Modifier,
                        fontSize = 16.sp

                    )
                    Box(
                        modifier = Modifier
                            .size(35.dp, 35.dp)
                            .clip(CircleShape)
                            .background(backgroundColor),
                        contentAlignment = Alignment.Center
                    ) {
                        IconButton(onClick = { removeItem(cartItem) }) {
                            Icon(
                                imageVector = Icons.Default.Delete,
                                contentDescription = null,
                                tint = orange
                            )
                        }
                    }
                }
            }
        }
        Quantity(
            count = count,
            decreaseItemCount = decreaseItemCount,
            increaseItemCount = increaseItemCount,
            price = cartItem.product.price * cartItem.quantity
        )
    }
}

@Preview
@Composable
fun PreviewCartViewItem() {
    CartViewItem(
        cartItem = PreviewData.cart_product_1,
        removeItem = {},
        count = 1,
        decreaseItemCount = {},
        increaseItemCount = {}
    )
}  
```

# Quantity

```kt
@Composable
fun Quantity(
    count: Int,
    decreaseItemCount: () -> Unit,
    increaseItemCount: () -> Unit,
    price: Double
) {

    Row(
        modifier = Modifier
            .fillMaxWidth()
            .padding(horizontal = 16.dp),
        horizontalArrangement = Arrangement.SpaceBetween,
        verticalAlignment = Alignment.CenterVertically
    ) {

        QuantitySelector(
            count = count,
            decreaseItemCount = decreaseItemCount,
            increaseItemCount = increaseItemCount
        )

        PriceView(price)
    }
}

@Composable
fun QuantitySelector(
    count: Int,
    decreaseItemCount: () -> Unit,
    increaseItemCount: () -> Unit,
) {
    Box(
        modifier = Modifier
            .wrapContentHeight()
    ) {
        Row(
            modifier = Modifier
                .padding(1.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            IconButton(onClick = decreaseItemCount) {
                Icon(
                    painter = painterResource(id = R.drawable.minus),
                    contentDescription = null,
                    tint = lightBlack
                )
            }
            Text(
                text = "$count",
                color = lightBlack,
                fontWeight = FontWeight.Bold,
                fontSize = 18.sp
            )

            IconButton(onClick = increaseItemCount) {
                Icon(
                    painter = painterResource(id = R.drawable.plus),
                    contentDescription = null,
                    tint = lightBlack
                )
            }
        }
    }
}


@Composable
fun PriceView(price: Double) {
    Text(
        text = "$ $price",
        color = orange,
        fontSize = 24.sp,
        fontWeight = FontWeight.Bold
    )
}

@Preview
@Composable
fun PreviewQuantity() {
    Quantity(1, {}, {}, PreviewData.product.price)
}

@Preview
@Composable
fun PreviewQuantitySelector() {
    QuantitySelector(1, {}, {})
}

@Preview
@Composable
fun PreviewPriceView() {
    PriceView(price = PreviewData.product.price)
}
```

# Empty

```kt
@Composable
fun Empty(
    userMessage: String
) {
    Column(
        Modifier.fillMaxSize(),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Image(
            painter = painterResource(R.drawable.ic_baseline_shopping_cart_24),
            contentDescription = null,
            modifier = Modifier.size(100.dp),
            colorFilter = ColorFilter.tint(color = lightGrey)
        )
        Spacer(modifier = Modifier.height(20.dp))
        Text(
            text = userMessage,
            style = MaterialTheme.typography.subtitle1,
            color = orange
        )
    }
}
```

# Loading

```kt
@Composable
fun Loading(

) {
    Column(
        Modifier.fillMaxSize().background(lightGrayBackground),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        CircularProgressIndicator(
            modifier = Modifier.size(50.dp),
            color = orange,
            strokeWidth = 5.dp
        )
    }
}

@Preview
@Composable
fun PreviewLoading() {
    Loading()
}
```

# Result

```kt
@Composable
fun Results(
    userMessage: String,
    content: @Composable () -> Unit
) {
    Column(
        Modifier
            .background(lightGrayBackground)
            .fillMaxSize()
            .padding(vertical = 16.dp, horizontal = 16.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Image(
            painter = painterResource(R.drawable.ic_baseline_check_circle_24),
            contentDescription = null,
            modifier = Modifier.size(100.dp),
            colorFilter = ColorFilter.tint(color = orange)
        )
        Spacer(modifier = Modifier.height(20.dp))
        Text(
            text = userMessage,
            style = MaterialTheme.typography.subtitle1,
            color = orange
        )
        Spacer(modifier = Modifier.height(20.dp))
        content()
    }
}

@Preview
@Composable
fun PreviewResults() {
    Results(userMessage = "Thanks for your purchase."){
        OrderSummary(
            order = PreviewData.order
        )
    }
}
```

# SearchBar

```kt
@Composable
fun SearchBar(
    modifier: Modifier = Modifier
) {
    var search by remember { mutableStateOf("") }
    TextField(
        modifier = modifier
            .fillMaxWidth()
            .padding(top = 10.dp),
        colors = TextFieldDefaults.textFieldColors(
            backgroundColor = lightbox,
            focusedIndicatorColor = Color.Transparent,
            unfocusedIndicatorColor = Color.Transparent,
        ),
        value = search,
        shape = RoundedCornerShape(12.dp),
        singleLine = true,
        onValueChange = { search = it },
        placeholder = {
            Text(
                text = "Search Products",
                color = lightGrey
            )
        },
        leadingIcon = {
            Icon(
                imageVector = Icons.Filled.Search,
                contentDescription = "",
                tint = lightBlack
            )
        }
    )
}

@Preview
@Composable
fun PreviewSearchBar() {
    SearchBar()
}
```

# Summary

```kt
@Composable
fun Summary(
    numberItems: Int,
    totalItems: Double,
    shippingCost: Double,
    taxCost: Double,
    total: Double
) {
    Column(
        modifier = Modifier.padding(10.dp)
    ) {
        SummaryText(
            label = "Items($numberItems):",
            value = "$ $totalItems"
        )
        SummaryText(
            label = "Shipping:",
            value = "$ $shippingCost"
        )
        SummaryText(
            label = "Tax:",
            value = "$ $taxCost"
        )
        SummaryText(
            label = "Total:",
            value = "$ $total",
            fontSizeLabel = 16.sp,
            fontSizeValue = 18.sp
        )
    }
}

@Composable
fun SummaryText(
    label: String,
    value: String,
    fontSizeLabel: TextUnit = 14.sp,
    fontSizeValue: TextUnit = 16.sp,
) {
    Row(
        modifier = Modifier.fillMaxWidth(),
        horizontalArrangement = Arrangement.SpaceBetween,
        verticalAlignment = Alignment.CenterVertically
    ) {
        Text(
            text = label,
            fontSize = fontSizeLabel,
            color = lightBlack
        )

        Text(
            text = value,
            fontSize = fontSizeValue,
            color = titleTextColor,
            fontWeight = FontWeight.Bold
        )
    }
}

@Preview
@Composable
fun PreviewSummary() {
    Summary(
        numberItems = 3,
        totalItems = 12.00,
        shippingCost = 10.00,
        taxCost = 2.00,
        total = 24.00
    )
}
```

