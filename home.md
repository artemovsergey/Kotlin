# HomeScreen

```kt
@Composable
fun HomeScreen(
    categorySelected: (category: Category) -> Unit,
    productSelected: (product: Product) -> Unit,
    modifier: Modifier = Modifier,
    viewModel: HomeViewModel = hiltViewModel<HomeViewModel>()
) {

    val products by viewModel.productListState.collectAsState()
    val categories by viewModel.categoryListState.collectAsState()

    HomeElements(
        categories,
        products,
        categorySelected,
        productSelected,
        modifier = modifier.background(lightGrayBackground)
    )
}

@Composable
private fun HomeElements(
    categories: List<Category>,
    products: List<Product>,
    categorySelected: (category: Category) -> Unit,
    productSelected: (product: Product) -> Unit,
    modifier: Modifier = Modifier,
) {
    Column(
        modifier
            .verticalScroll(rememberScrollState())
            .padding(vertical = 16.dp)
    ) {
        SearchBar(Modifier.padding(horizontal = 16.dp))
        HomeSection(
            title = stringResource(AppText.categories),
            withArrow = false
        ) {
            CategorySection(
                categories = categories,
                categorySelected = categorySelected
            )
        }
        HomeSection(
            title = stringResource(AppText.recommended),
            withArrow = true
        ) {
            ProductSection(
                products = products,
                productSelected = productSelected
            )
        }
        HomeSection(
            title = stringResource(AppText.new_arrivals),
            withArrow = true
        ) {
            ProductSection(
                products = products,
                productSelected = productSelected
            )
        }
    }
}

@Preview(showSystemUi = true)
@Composable
fun PreviewHomeElements() {
    HomeElements(
        products = PreviewData.products(),
        categories = PreviewData.categories(),
        categorySelected = {},
        productSelected = {}
    )
}
```

# HomeViewModel

```kt
@HiltViewModel
class HomeViewModel @Inject constructor(
    private val productRepository: IProductRepository,
    private val categoryRepository: ICategoryRepository
) : ViewModel() {


      val productListState = productRepository.getProducts()
          .stateIn(viewModelScope, SharingStarted.Eagerly, emptyList())

      val categoryListState = categoryRepository.getCategories()
        .stateIn(viewModelScope, SharingStarted.Eagerly, emptyList())
}
```

# HomeSection

```kt
@Composable
fun HomeSection(
    title: String,
    withArrow: Boolean,
    modifier: Modifier = Modifier,
    content: @Composable () -> Unit
) {
    Column(modifier) {
        Row(
            verticalAlignment = Alignment.CenterVertically,
            modifier = Modifier
                .heightIn(min = 56.dp)
                .padding(horizontal = 16.dp)
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
            if (withArrow) {
                IconButton(
                    onClick = { },
                    modifier = Modifier.align(Alignment.CenterVertically)
                ) {
                    Icon(
                        imageVector = mirroringIcon(
                            ltrIcon = Icons.Outlined.ArrowForward,
                            rtlIcon = Icons.Outlined.ArrowBack
                        ),
                        tint = orange,
                        contentDescription = null
                    )
                }
            }
        }
        content()
    }
}
```


# ProductSection

```kt
@Composable
fun ProductSection(
    products: List<Product>,
    productSelected: (product: Product) -> Unit,
    modifier: Modifier = Modifier
) {
    LazyRow(
        modifier = modifier,
        contentPadding = PaddingValues(horizontal = 16.dp),
        horizontalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(products, key = { it.id }) { product ->
            ProductCard(product, productSelected)
        }
    }
}

@Preview
@Composable
fun PreviewProductSection() {
    ProductSection(
        products = PreviewData.products(),
        productSelected = {}
    )
}
```

# CategorySection

```kt
@Composable
fun CategorySection(
    categories: List<Category>,
    categorySelected: (category: Category) -> Unit,
    modifier: Modifier = Modifier
) {
    LazyRow(
        modifier = modifier,
        contentPadding = PaddingValues(horizontal = 16.dp),
        horizontalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(categories, key = { it.id }) { category ->
            CategoryCard(category, categorySelected)
        }
    }
}

@Preview
@Composable
fun PreviewCategorySection() {
    CategorySection(
        categories = PreviewData.categories(),
        categorySelected = {}
    )
}
```

