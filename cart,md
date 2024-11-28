# CartScreen

```kt
@Composable
fun CartScreen(
    goToCheckout: (SummaryTotals) -> Unit,
    modifier: Modifier = Modifier,
    viewModel: CartViewModel = hiltViewModel(),
) {
    CartUI(
        cartState = viewModel.cartState,
        orderSummaryState = viewModel.orderSummaryState,
        updateQuantity = viewModel.updateQuantity(),
        removeItem = viewModel.removeItem(),
        checkoutSelected = { goToCheckout(viewModel.getSummaryOrder()) },
        modifier = modifier
            .fillMaxSize()
            .background(lightGrayBackground)
    )
}

@Composable
fun CartUI(
    cartState: CartState,
    orderSummaryState: OrderSummaryState,
    updateQuantity: (CartItem, Int) -> Unit,
    removeItem: (CartItem) -> Unit,
    checkoutSelected: () -> Unit,
    modifier: Modifier,
) {
    if (cartState.cartItems.isEmpty()) {
        Empty("No cart items found")
    } else {
        CartElements(
            cartItems = cartState.cartItems,
            numberItems = cartState.cartItems.size,
            totalItems = orderSummaryState.itemsTotal,
            shippingCost = orderSummaryState.shipping,
            taxCost = orderSummaryState.tax,
            total = orderSummaryState.allTotal,
            checkoutSelected = checkoutSelected,
            checkoutReady = cartState.readyForCheckout,
            removeItem = removeItem,
            decreaseItemCount = updateQuantity,
            increaseItemCount = updateQuantity,
            modifier = modifier
        )
    }
}

@Composable
fun CartElements(
    cartItems: List<CartItem>,
    numberItems: Int,
    totalItems: Double,
    shippingCost: Double,
    taxCost: Double,
    total: Double,
    checkoutSelected: () -> Unit,
    checkoutReady: Boolean,
    removeItem: (CartItem) -> Unit,
    decreaseItemCount: (CartItem, Int) -> Unit,
    increaseItemCount: (CartItem, Int) -> Unit,
    modifier: Modifier = Modifier,
    contentPadding: PaddingValues = PaddingValues(10.dp),
) {
    LazyColumn(
        modifier = modifier,
        contentPadding = contentPadding
    ) {
        items(cartItems, key = { it.product.id }) { cartItem ->
            CartViewItem(
                cartItem = cartItem,
                removeItem = removeItem,
                count = cartItem.quantity,
                decreaseItemCount = { decreaseItemCount(cartItem, -1) },
                increaseItemCount = { increaseItemCount(cartItem, 1) }
            )
        }
        item {
            CartSummary(
                numberItems = numberItems,
                totalItems = totalItems,
                shippingCost = shippingCost,
                taxCost = taxCost,
                total = total
            )
            CartBottom(
                checkoutSelected = checkoutSelected,
                checkoutReady = checkoutReady
            )
        }
    }
}


@Preview(showSystemUi = true)
@Composable
fun CartScreenPreview() {
    CartElements(
        cartItems = PreviewData.cartProducts(),
        numberItems = 3,
        totalItems = 12.00,
        shippingCost = 10.00,
        taxCost = 2.00,
        total = 24.00,
        checkoutSelected = {},
        checkoutReady = true,
        removeItem = {},
        decreaseItemCount = { _, _ -> },
        increaseItemCount = { _, _ -> }
    )
}
```


# CartViewModel

```kt
@HiltViewModel
class CartViewModel @Inject constructor(
    private val cartRepository: ICartRepository,
) : ViewModel() {

    var cartState by mutableStateOf(CartState())
        private set

    var orderSummaryState by mutableStateOf(OrderSummaryState())
        private set

    init {
        viewModelScope.launch {
            cartRepository
                .getCartItems().collect {
                    updateStates(cartItems = it)
                }
        }
    }

    fun removeItem(): (CartItem) -> Unit = { cartItem ->
        viewModelScope.launch {
            cartRepository
                .removeItemCart(cartItem).collect {
                    updateStates(cartItems = it)
                }
        }
    }

    fun updateQuantity(): (CartItem, Int) -> Unit = { cartItem, operator ->
        if (!(operator < 0 && cartItem.quantity == 1)) {
            viewModelScope.launch {
                cartRepository
                    .updateCartItems(cartItem, operator).collect {
                        updateStates(cartItems = it)
                    }
            }
        }
    }

    private fun updateStates(cartItems: List<CartItem>) {
        cartState = cartState.copy(cartItems = cartItems)
        orderSummaryState = orderSummaryState.copy(cartItems = cartItems)
    }

    fun getSummaryOrder(): SummaryTotals {
        return summaryStateToModel(orderSummaryState)
    }

    private fun summaryStateToModel(
        orderSummaryState: OrderSummaryState,
    ): SummaryTotals {
        return SummaryTotals(
            numberItems = orderSummaryState.cartItems.size,
            totalItems = orderSummaryState.itemsTotal,
            shippingCost = orderSummaryState.shipping,
            taxCost = orderSummaryState.tax,
            total = orderSummaryState.allTotal
        )
    }
}
```

# CartState

```kt
data class CartState(
    val cartItems: List<CartItem> = emptyList()
)

val CartState.readyForCheckout: Boolean
    get() = cartItems.isNotEmpty()
```


# OrderSummaryState

```kt
data class OrderSummaryState(
    val cartItems: List<CartItem> = emptyList(),
    val shipping: Double = 0.00,
    val tax: Double = 0.00
)

val OrderSummaryState.itemsTotal: Double
    get() =
        cartItems
            .map { item -> item.product.price * item.quantity }
            .fold(0.0) { total, next -> total + next }

val OrderSummaryState.allTotal: Double
    get() = itemsTotal + shipping + tax
```

  # CartBottom

```kt
@Composable
fun CartBottom(
    checkoutSelected: () -> Unit,
    checkoutReady: Boolean
) {
    StandardButton(
        text = stringResource(id = AppText.checkout),
        onClicked = checkoutSelected,
        enabled = checkoutReady
    )
}

@Preview
@Composable
fun PreviewCartBottom() {
    CartBottom(
        checkoutSelected = {},
        checkoutReady = true
    )
}
```


# CartSummary

```kt
@Composable
fun CartSummary(
    numberItems: Int,
    totalItems: Double,
    shippingCost: Double,
    taxCost: Double,
    total: Double
) {

    Column(
        modifier = Modifier.fillMaxWidth()
    ) {
        Divider(color = lightGrey, thickness = 2.dp)
        Spacer(modifier = Modifier.padding(8.dp))
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
fun PreviewCartSummary() {
    CartSummary(
        numberItems = 3,
        totalItems = 12.00,
        shippingCost = 10.00,
        taxCost = 2.00,
        total = 24.00
    )
}
```
