# ProductListScreen

```kt
@Composable
fun ProductListScreen(
    category: Category,
    productSelected: (Product) -> Unit,
    modifier: Modifier = Modifier,
    viewModel: ProductListViewModel = hiltViewModel()
) {
    val products by viewModel.productListState(category).collectAsState()

    ProductListContent(
        products,
        productSelected,
        modifier = modifier.background(lightGrayBackground)
    )
}

@Composable
fun ProductListContent(
    products: List<Product>,
    productSelected: (Product) -> Unit,
    modifier: Modifier = Modifier
) {
    ProductItemList(
        products = products,
        productSelected = productSelected,
        modifier = modifier
    )
}

@Preview(showSystemUi = true)
@Composable
fun ProductListScreenPreview() {
    ProductListContent(
        products = PreviewData.products(),
        productSelected = {}
    )
}
```

# ProductListViewModel

```kt
@HiltViewModel
class ProductListViewModel @Inject constructor(
    private val productRepository: IProductRepository
) : ViewModel() {

    val productListState = { category: Category ->
        productRepository.getProducts(category)
            .stateIn(viewModelScope, SharingStarted.Eagerly, emptyList())
    }
}
```

# ProductList

```kt
@Composable
fun ProductItemList(
    products: List<Product>,
    productSelected: (Product) -> Unit,
    modifier: Modifier = Modifier,
) {
    LazyColumn(modifier = modifier) {
        items(products, key = { it.id }) { product ->
            ProductItem(product, productSelected)
        }
    }
}

@Preview
@Composable
fun PreviewProductListScreen() {
    ProductItemList(
        products = PreviewData.products(),
        productSelected = {}
    )
}
```
