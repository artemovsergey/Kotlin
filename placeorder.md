# PlaceorderScreen

```kt
@Composable
fun PlaceOrderScreen(
    order: Order,
    modifier: Modifier = Modifier,
    viewModel: PlaceOrderViewModel = hiltViewModel(),
) {
    OrderUI(
        order = order,
        paymentUiState = viewModel.paymentUiState,
        onPlaceOrder = { viewModel.makePayment() },
        modifier = modifier
            .fillMaxSize()
            .background(lightGrayBackground)
    )
}

@Composable
fun OrderUI(
    order: Order,
    paymentUiState: PaymentState,
    onPlaceOrder: () -> Unit,
    modifier: Modifier,
) {
    if (paymentUiState.isLoading) {
        Loading()
    } else {
        Column {
            if (paymentUiState.paymentResult != null) {
                val successful = paymentUiState.paymentResult.status
                if (successful) {
                    Results(userMessage = "Thanks for your purchase.") {
                        OrderSummary(
                            order = order
                        )
                    }
                }
            } else {
                OrderElements(
                    order = order,
                    onPlaceOrder = onPlaceOrder,
                    modifier = modifier
                )
            }
        }
    }
}

@Composable
fun OrderElements(
    order: Order,
    onPlaceOrder: () -> Unit,
    modifier: Modifier = Modifier,
) {
    Column(
        modifier
            .verticalScroll(rememberScrollState())
            .padding(vertical = 16.dp, horizontal = 16.dp),
        verticalArrangement = Arrangement.spacedBy(20.dp)
    ) {
        OrderSummary(
            order = order,
            modifier = modifier
        )
        StandardButton(
            text = stringResource(R.string.pay_order, order.summary.total.toString()),
            onClicked = onPlaceOrder
        )
    }
}

@Composable
fun OrderSummary(
    order: Order,
    modifier: Modifier = Modifier,
) {
    Column(
        modifier,
        verticalArrangement = Arrangement.spacedBy(20.dp)
    ) {
        OrderInformation(
            order = order
        )
        SummaryCard(
            numberItems = order.summary.numberItems,
            totalItems = order.summary.totalItems,
            shippingCost = order.summary.shippingCost,
            taxCost = order.summary.taxCost,
            total = order.summary.total,
            modifier = modifier
        )
    }
}

@Preview(showSystemUi = true)
@Composable
fun PreviewPlaceOrderScreen() {
    OrderElements(
        order = PreviewData.order,
        onPlaceOrder = {}
    )
}

@Preview(showSystemUi = true)
@Composable
fun PreviewOrderInformation() {
    OrderSummary(
        order = PreviewData.order
    )
}
```

# PlaceOrderViewModel

```kt
@HiltViewModel
class PlaceOrderViewModel @Inject constructor(
    private val paymentRepository: IPaymentRepository,
) : ViewModel() {

    var paymentUiState by mutableStateOf(PaymentState())
        private set

    fun makePayment() {
        viewModelScope.launch {
            paymentUiState = paymentUiState.copy(isLoading = true)
            try {
                val paymentResult = paymentRepository
                    .doPayment(order = PreviewData.order)
                paymentUiState =
                    paymentUiState.copy(
                        isLoading = false,
                        paymentResult = paymentResult
                    )
                println(paymentResult)
            } catch (ioe: IOException) {
                paymentUiState = paymentUiState.copy(
                    isLoading = false,
                    errorMessage = "Error with the transaction"
                )
            }
        }
    }
}
```

# PaymentState

```kt
data class PaymentState(
    val paymentInformation: Payment = Payment(),
    val isLoading: Boolean = false,
    val paymentResult: PaymentResult? = null,
    val errorMessage: String? = null
)
```

# OrderInformation

```kt
@Composable
fun OrderInformation(
    order: Order,
) {
    Column(
        modifier = Modifier.padding(10.dp)
    ) {
        OrderSummaryText(
            label = "Name: ",
            value = order.name
        )
        OrderSummaryText(
            label = "Phone:",
            value = order.phone
        )
        OrderSummaryText(
            label = "Address:",
            value = order.address
        )
    }
}

@Composable
fun OrderSummaryText(
    label: String,
    value: String,
    fontSizeLabel: TextUnit = 14.sp,
    fontSizeValue: TextUnit = 16.sp,
) {
    Row(
        modifier = Modifier.fillMaxWidth(),
        verticalAlignment = Alignment.CenterVertically
    ) {
        Text(
            text = label,
            fontSize = fontSizeLabel,
            color = lightBlack
        )
        Spacer(Modifier.padding(5.dp))
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
fun PreviewOrderInformation() {
    OrderInformation(
        order = PreviewData.order
    )
}

@Preview
@Composable
fun PreviewOrderSummaryText() {
    OrderSummaryText(
        label = "Name:",
        value = "Yair Carreno"
    )
}
```
