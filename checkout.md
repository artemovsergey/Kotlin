# CheckoutScreen

```kt
@Composable
fun CheckoutScreen(
    summary: SummaryTotals,
    goToPlaceOrder: (Order) -> Unit,
    modifier: Modifier = Modifier,
    viewModel: CheckoutViewModel = hiltViewModel()
) {
    CheckoutUI(
        summary = summary,
        contactUiState = viewModel.contactUiState,
        onContactEvent = viewModel.onContactFormEvent,
        paymentUiState = viewModel.paymentUiState,
        onPaymentEvent = viewModel.onPaymentFormEvent,
        onContinueOrder = { goToPlaceOrder(viewModel.getOrder(summary)) },
        modifier = modifier
    )
}

@Composable
fun CheckoutUI(
    summary: SummaryTotals,
    contactUiState: ContactFormState,
    onContactEvent: (ContactFormEvent) -> Unit,
    paymentUiState: PaymentFormState,
    onPaymentEvent: (PaymentFormEvent) -> Unit,
    onContinueOrder: () -> Unit,
    modifier: Modifier = Modifier,
) {
    CheckoutElements(
        summary = summary,
        contact = contactUiState,
        onContactEvent = onContactEvent,
        payment = paymentUiState,
        onPaymentEvent = onPaymentEvent,
        onContinueOrder = onContinueOrder,
        modifier = modifier.background(lightGrayBackground)
    )
}

@Composable
fun CheckoutElements(
    summary: SummaryTotals,
    contact: ContactFormState,
    onContactEvent: (ContactFormEvent) -> Unit,
    payment: PaymentFormState,
    onPaymentEvent: (PaymentFormEvent) -> Unit,
    onContinueOrder: () -> Unit,
    modifier: Modifier = Modifier,
) {
    Column(
        modifier
            .verticalScroll(rememberScrollState())
            .padding(vertical = 16.dp, horizontal = 16.dp),
        verticalArrangement = Arrangement.spacedBy(20.dp)
    ) {
        SummaryCard(
            numberItems = summary.numberItems,
            totalItems = summary.totalItems,
            shippingCost = summary.shippingCost,
            taxCost = summary.taxCost,
            total = summary.total,
            modifier = modifier
        )
        CheckoutSection(
            title = "Contact Information"
        ) {
            ContactInformation(
                nameStateVsEvent = StateVsEvent(
                    value = contact.username,
                    onValueChange = {
                        onContactEvent(ContactFormEvent.OnNameChange(it))
                    }),
                phoneStateVsEvent = StateVsEvent(
                    value = contact.phone,
                    onValueChange = {
                        onContactEvent(ContactFormEvent.OnPhoneChange(it))
                    }),
                addressStateVsEvent = StateVsEvent(
                    value = contact.address,
                    onValueChange = {
                        onContactEvent(ContactFormEvent.OnAddressChange(it))
                    }),
                modifier = modifier
            )
        }
        CheckoutSection(
            title = "Payment Information"
        ) {
            PaymentInformation(
                nameStateVsEvent = StateVsEvent(
                    value = payment.name,
                    onValueChange = {
                        onPaymentEvent(PaymentFormEvent.OnNameChange(it))
                    }),
                numberStateVsEvent = StateVsEvent(
                    value = payment.number,
                    onValueChange = {
                        onPaymentEvent(PaymentFormEvent.OnNumberChange(it))
                    }),
                monthStateVsEvent = StateVsEvent(
                    value = payment.month,
                    onValueChange = {
                        onPaymentEvent(PaymentFormEvent.OnMonthChange(it))
                    }),
                yearStateVsEvent = StateVsEvent(
                    value = payment.year,
                    onValueChange = {
                        onPaymentEvent(PaymentFormEvent.OnYearChange(it))
                    }),
                codeStateVsEvent = StateVsEvent(
                    value = payment.code,
                    onValueChange = {
                        onPaymentEvent(PaymentFormEvent.OnCodeChange(it))
                    }),
            )
        }
        StandardButton(
            text = stringResource(R.string.continue_order),
            onClicked = onContinueOrder,
            enabled = contact.successValidated && payment.successValidated
        )
    }
}

@Preview(showSystemUi = true)
@Composable
fun PreviewCheckoutScreen() {
    CheckoutElements(
        summary = PreviewData.summary,
        contact = ContactFormState(),
        onContactEvent = {},
        payment = PaymentFormState(),
        onPaymentEvent = {},
        onContinueOrder = {}
    )
}
```

# CheckoutViewModel

```kt
@HiltViewModel
class CheckoutViewModel @Inject constructor() : ViewModel() {

    var contactUiState by mutableStateOf(ContactFormState())
        private set

    var paymentUiState by mutableStateOf(PaymentFormState())
        private set

    val onContactFormEvent: (ContactFormEvent) -> Unit = { event ->
        contactUiState = when (event) {
            is ContactFormEvent.OnNameChange -> {
                contactUiState.copy(username = event.name)
            }
            is ContactFormEvent.OnPhoneChange -> {
                contactUiState.copy(phone = event.phone)
            }
            is ContactFormEvent.OnAddressChange -> {
                contactUiState.copy(address = event.address)
            }
        }
    }

    val onPaymentFormEvent: (PaymentFormEvent) -> Unit = { event ->
        paymentUiState = when (event) {
            is PaymentFormEvent.OnNameChange -> {
                paymentUiState.copy(name = event.name)
            }
            is PaymentFormEvent.OnNumberChange -> {
                paymentUiState.copy(number = event.number)
            }
            is PaymentFormEvent.OnMonthChange -> {
                paymentUiState.copy(month = event.month)
            }
            is PaymentFormEvent.OnYearChange -> {
                paymentUiState.copy(year = event.year)
            }
            is PaymentFormEvent.OnCodeChange -> {
                paymentUiState.copy(code = event.code)
            }
        }
    }

    fun getOrder(summaryTotals: SummaryTotals): Order {
        return orderStateToModel(
            contactUiState,
            paymentUiState,
            summaryTotals)
    }

    private fun orderStateToModel(
        contact: ContactFormState,
        payment: PaymentFormState,
        summary: SummaryTotals,
    ): Order {
        return Order(
            name = contact.username,
            phone = contact.phone,
            address = contact.address,
            cardInformation = CardInformation(
                name = payment.name,
                number = payment.number,
                month = payment.month,
                year = payment.year,
                code = payment.code),
            summary = summary)
    }
}
```

# ContactFormState

```kt
data class ContactFormState(
    val username: String = "",
    val phone: String = "",
    val address: String = "",
)

val ContactFormState.successValidated: Boolean
    get() = username.length > 1
            && phone.length > 1
            && address.length > 1
```

# PaymentFormState

```kt
data class PaymentFormState(
    val name: String = "",
    val number: String = "",
    val month: String = "",
    val year: String = "",
    val code: String = ""
)

val PaymentFormState.successValidated: Boolean
    get() = name.length > 1
            && number.length > 1
            && month.length > 1
            && year.length > 1
            && code.length > 2
```

# ContactFormEvent

```kt
sealed class ContactFormEvent {
    data class OnNameChange(val name: String): ContactFormEvent()
    data class OnPhoneChange(val phone: String): ContactFormEvent()
    data class OnAddressChange(val address: String): ContactFormEvent()
}
```

# PaymentFormEvent

```kt
sealed class PaymentFormEvent {
    data class OnNameChange(val name: String): PaymentFormEvent()
    data class OnNumberChange(val number: String): PaymentFormEvent()
    data class OnMonthChange(val month: String): PaymentFormEvent()
    data class OnYearChange(val year: String): PaymentFormEvent()
    data class OnCodeChange(val code: String): PaymentFormEvent()
}
```


# CheckoutSection

```kt
@Composable
fun CheckoutSection(
    title: String,
    modifier: Modifier = Modifier,
    content: @Composable () -> Unit
) {
    Column(modifier) {
        Row(
            verticalAlignment = Alignment.CenterVertically,
            modifier = Modifier
                .padding(bottom = 8.dp)
        ) {
            Text(
                text = title,
                style = MaterialTheme.typography.h6,
                color = orange,
                maxLines = 1,
                overflow = TextOverflow.Ellipsis,
                modifier = Modifier
                    .weight(1f)
                    .wrapContentWidth(Alignment.Start)
            )
        }
        content()
    }
}

@Preview
@Composable
fun PreviewContactInformation() {
    CheckoutSection(
        title = "Contact information"
    ) {
        ContactInformation(
            nameStateVsEvent = StateVsEvent(),
            phoneStateVsEvent = StateVsEvent(),
            addressStateVsEvent = StateVsEvent()
        )
    }
}

@Preview
@Composable
fun PreviewPaymentInformation() {
    CheckoutSection(
        title = "Payment information"
    ) {
        PaymentInformation(
            nameStateVsEvent = StateVsEvent(),
            numberStateVsEvent = StateVsEvent(),
            monthStateVsEvent = StateVsEvent(),
            yearStateVsEvent = StateVsEvent(),
            codeStateVsEvent = StateVsEvent(),
        )
    }
}
```

# CheckoutInformation

```kt
@Composable
fun ContactInformation(
    nameStateVsEvent: StateVsEvent,
    phoneStateVsEvent: StateVsEvent,
    addressStateVsEvent: StateVsEvent,
    modifier: Modifier = Modifier
) {
    Column(
        modifier = modifier
    ) {
        ContactField(
            label = "User name",
            value = nameStateVsEvent.value,
            onValueChange = nameStateVsEvent.onValueChange
        )
        Spacer(Modifier.padding(5.dp))
        ContactField(
            label = "Phone number",
            value = phoneStateVsEvent.value,
            onValueChange = phoneStateVsEvent.onValueChange
        )
        Spacer(Modifier.padding(5.dp))
        ContactField(
            label = "Address",
            value = addressStateVsEvent.value,
            onValueChange = addressStateVsEvent.onValueChange
        )
    }
}

@Composable
fun ContactField(
    label: String,
    value: String,
    onValueChange: (String) -> Unit,
    modifier: Modifier = Modifier
) {
    TextField(
        label = { Text(label) },
        value = value,
        onValueChange = onValueChange,
        colors = TextFieldDefaults.textFieldColors(
            backgroundColor = MaterialTheme.colors.surface
        ),
        modifier = modifier.fillMaxWidth()
    )
}

@Preview
@Composable
fun PreviewContactInformation(

) {
    ContactInformation(
        nameStateVsEvent = StateVsEvent(),
        phoneStateVsEvent = StateVsEvent(),
        addressStateVsEvent = StateVsEvent()
    )
}
```

# PaymentsInformation

```kt
@Composable
fun PaymentInformation(
    nameStateVsEvent: StateVsEvent,
    numberStateVsEvent: StateVsEvent,
    monthStateVsEvent: StateVsEvent,
    yearStateVsEvent: StateVsEvent,
    codeStateVsEvent: StateVsEvent,
    modifier: Modifier = Modifier,
) {
    Column(
        modifier = modifier
    ) {
        CardField(
            label = "Name on card",
            value = nameStateVsEvent.value,
            onValueChange = nameStateVsEvent.onValueChange,
            idIcon = R.drawable.ic_baseline_person_24
        )
        Spacer(Modifier.padding(5.dp))
        CardField(
            label = "0000-0000-0000-0000",
            value = numberStateVsEvent.value,
            onValueChange = numberStateVsEvent.onValueChange,
            idIcon = R.drawable.ic_baseline_credit_card_24
        )
        Spacer(Modifier.padding(5.dp))
        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.spacedBy(8.dp)
        ) {
            DateCard(
                label = "MM",
                value = monthStateVsEvent.value,
                onValueChange = monthStateVsEvent.onValueChange,
                modifier = modifier.weight(0.3f)
            )
            DateCard(
                label = "YY",
                value = yearStateVsEvent.value,
                onValueChange = yearStateVsEvent.onValueChange,
                modifier = modifier.weight(0.3f)
            )
            DateCard(
                label = "CVV",
                value = codeStateVsEvent.value,
                onValueChange = codeStateVsEvent.onValueChange,
                modifier = modifier.weight(0.3f)
            )
        }
    }
}

@Composable
fun CardField(
    label: String,
    value: String,
    onValueChange: (String) -> Unit,
    idIcon: Int,
    modifier: Modifier = Modifier,
) {
    TextField(
        label = { Text(label) },
        value = value,
        onValueChange = onValueChange,
        colors = TextFieldDefaults.textFieldColors(
            backgroundColor = MaterialTheme.colors.surface
        ),
        trailingIcon = {
            Icon(
                painter = painterResource(id = idIcon),
                contentDescription = null,
                tint = lightBlack
            )
        },
        modifier = modifier.fillMaxWidth()
    )
}

@Composable
fun DateCard(
    label: String,
    value: String,
    onValueChange: (String) -> Unit,
    modifier: Modifier,
) {
    TextField(
        label = { Text(label) },
        value = value,
        onValueChange = onValueChange,
        colors = TextFieldDefaults.textFieldColors(
            backgroundColor = MaterialTheme.colors.surface
        ),
        modifier = modifier
    )
}

@Preview
@Composable
fun PreviewPaymentInformation(

) {
    PaymentInformation(
        nameStateVsEvent = StateVsEvent(),
        numberStateVsEvent = StateVsEvent(),
        monthStateVsEvent = StateVsEvent(),
        yearStateVsEvent = StateVsEvent(),
        codeStateVsEvent = StateVsEvent(),
    )
}
```
