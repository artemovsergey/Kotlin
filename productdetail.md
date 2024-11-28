# ProductDetailScreen

```kt
@Composable
fun ProductDetailScreen(
    product: Product,
    goToCart: () -> Unit,
    modifier: Modifier = Modifier,
    viewModel: ProductDetailViewModel = hiltViewModel(),
) {
    BodyDetail(
        product = product,
        addToCart = { count -> viewModel.saveItemCart(product, count) },
        showAlert = viewModel.showAlert,
        onGoToCart = goToCart,
        onPopupDismissed = viewModel.onPopupDismissed(),
        modifier = modifier
            .fillMaxSize()
            .background(lightGrayBackground)
    )
}

@Preview(showSystemUi = true)
@Composable
fun ProductDetailScreenPreview() {
    BodyDetail(
        product = PreviewData.product,
        addToCart = {},
        showAlert = false,
        onGoToCart = {},
        onPopupDismissed = {}
    )
}
```


# ProductDetailViewModel

```kt
@HiltViewModel
class ProductDetailViewModel @Inject constructor(
    private val cartRepository: ICartRepository
) : ViewModel() {

    var showAlert by mutableStateOf(false)
        private set

    fun saveItemCart(product: Product, quantity: Int) {
        viewModelScope.launch {
            val cartItem = CartItem(
                quantity = quantity,
                product = product
            )
            cartRepository
                .saveItemCart(cartItem).collect {
                    showAlert = true
                }
        }
    }

    fun onPopupDismissed(): () -> Unit = {
        showAlert = false
    }
}
```

# HeaderSection

```kt
@Composable
fun HeaderSection(product: Product) {
    Column(
        modifier = Modifier.fillMaxWidth(),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Divider(
            color = white,
            modifier = Modifier
                .height(5.dp)
                .width(30.dp)
        )
    }

    Column(
        modifier = Modifier.padding(horizontal = 16.dp),
        horizontalAlignment = Alignment.Start
    ) {
        Text(
            text = product.category.name,
            fontSize = 16.sp,
            color = Color.Gray,
        )
        Spacer(modifier = Modifier.height(4.dp))
        Text(
            text = product.name,
            fontSize = 24.sp,
            color = lightBlack,
            fontWeight = FontWeight.Bold
        )
        Text(
            text = buildAnnotatedString {
                withStyle(
                    style = SpanStyle(
                        titleTextColor
                    )
                ) {
                    append("$ " + product.price + "/per item")
                }
            },
            style = MaterialTheme.typography.subtitle1,
            modifier = Modifier,
            fontSize = 16.sp

        )
        Spacer(modifier = Modifier.height(4.dp))
        product.description?.let {
            Spacer(modifier = Modifier.height(4.dp))
            Text(
                text = it,
                fontSize = 18.sp,
                color = Color.Gray
            )
        }
    }
}

@Preview
@Composable
fun PreviewHeaderSection() {
    HeaderSection(product = PreviewData.product)
}
```

# BottomSection

```kt
@Composable
fun BottomSection(
    onBuyClicked: () -> Unit
) {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .padding(bottom = 20.dp),
        horizontalArrangement = Arrangement.Center
    ) {
        StandardButton(
            text = stringResource(AppText.add_to_cart),
            onClicked = onBuyClicked
        )
    }
}


@Preview
@Composable
fun PreviewBottomSection() {
    BottomSection(
        onBuyClicked = {}
    )
}
```

# BodyDetail

```kt
@Composable
fun BodyDetail(
    product: Product,
    addToCart: (Int) -> Unit,
    showAlert: Boolean,
    onGoToCart: () -> Unit,
    onPopupDismissed: () -> Unit,
    modifier: Modifier = Modifier
) {
    Box(
        modifier = modifier
            .verticalScroll(rememberScrollState())
    ) {
        ConstraintLayout {

            val (ProductImage, BuyForm) = createRefs()
            val (count, updateCount) = remember { mutableStateOf(1) }

            Box(contentAlignment = Alignment.Center,
                modifier = Modifier
                    .height(280.dp)
                    .constrainAs(ProductImage) {
                        top.linkTo(BuyForm.top)
                        bottom.linkTo(BuyForm.top)
                        start.linkTo(parent.start)
                        end.linkTo(parent.end)
                    }
            ) {
                HeaderView(product)
            }
            Surface(
                color = lightGrayBackground,
                shape = RoundedCornerShape(40.dp).copy(
                    bottomStart = ZeroCornerSize,
                    bottomEnd = ZeroCornerSize
                ),
                modifier = Modifier
                    .fillMaxSize()
                    .padding(top = 100.dp)
                    .constrainAs(BuyForm) {
                        bottom.linkTo(parent.bottom)
                        start.linkTo(parent.start)
                        end.linkTo(parent.end)
                    },
            ) {
                Column(
                    modifier = Modifier
                        .fillMaxSize()
                        .padding(horizontal = 16.dp)
                ) {
                    Spacer(modifier = Modifier.padding(5.dp))
                    HeaderSection(product)
                    Spacer(modifier = Modifier.padding(10.dp))
                    Divider(color = lightGrey, thickness = 2.dp)
                    Quantity(
                        count = count,
                        decreaseItemCount = { if (count > 1) updateCount(count - 1) },
                        increaseItemCount = { updateCount(count + 1) },
                        price = product.price * count
                    )
                    Spacer(modifier = Modifier.padding(10.dp))
                    BottomSection(onBuyClicked = { addToCart(count) })
                }
            }
        }
    }
    if (showAlert) {
        AlertDialog(
            onDismissRequest = onPopupDismissed,
            title = {
                Text(text = stringResource(AppText.title_detail_alert))
            },
            text = {
                Text(text = stringResource(AppText.text_detail_alert))
            },
            confirmButton = {
                Button(
                    onClick = onGoToCart,
                    colors = ButtonDefaults.buttonColors(
                        backgroundColor = orange
                    )
                ) {
                    Text(
                        text = stringResource(AppText.go_to_cart),
                        color = white
                    )
                }
            },
            dismissButton = {
                Button(
                    onClick = onPopupDismissed,
                    colors = ButtonDefaults.buttonColors(
                        backgroundColor = orange
                    )
                ) {
                    Text(
                        text = stringResource(AppText.continue_shopping),
                        color = white
                    )
                }
            })
    }
}

@Composable
fun HeaderView(product: Product) {
    Column(
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally,
        modifier = Modifier
            .fillMaxWidth()
            .fillMaxHeight()
            .background(white)
            .padding(top = 20.dp)
    ) {
        Box(
            modifier = Modifier
                .fillMaxHeight()
        ) {
            AsyncImage(
                model = ImageRequest.Builder(LocalContext.current)
                    .data(product.image)
                    .crossfade(true)
                    .build(),
                contentScale = ContentScale.Fit,
                placeholder = painterResource(R.drawable.product_placeholder),
                contentDescription = "",
                modifier = Modifier.size(230.dp),
            )
        }
    }
}

@Preview
@Composable
fun PreviewBodyDetail() {
    BodyDetail(
        product = PreviewData.product,
        addToCart = {},
        showAlert = false,
        onGoToCart = {},
        onPopupDismissed = {}
    )
}
```

