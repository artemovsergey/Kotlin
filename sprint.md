# Model

```kt
data class User (val id: Number, val name: String){}
```

# Interface

```kt
interface IUserRepository {
    fun getAll(): Flow<List<User>>
    fun get(id: Number) : Flow<User>
}
```


# Repository

```kt
class UserRepository : IUserRepository
{
    private val user1 = User(1, "user1")
    private val users: List<User> = listOf(user1)

    override fun getAll(): Flow<List<User>> {
        return flowOf(users)
    }

    override fun get(id: Number): Flow<User> {

        var user = users.find { it.id == id }

        return if(user != null){
            flowOf(user)
        } else{
            throw Exception("Нет пользователя с id = $id")
        }
    }
}
```



# UserListScreen

```kt
@Composable
fun UsersListScreen(
    modifier: Modifier = Modifier,
    viewModel: UsersListViewModel = UsersListViewModel()
) {

    val users by viewModel.getUsers().collectAsState(initial = emptyList())

    UsersContent(users = users)
}

@Composable
fun UsersContent(users: List<User>) {

    Text(text = "${users.count()}")
    Column {
        users.forEach { u ->
            UserItem(u)
        }
    }
}

@Composable
fun UserItem(user: User) {

    Row(
        modifier = Modifier.padding(16.dp),
        verticalAlignment = Alignment.CenterVertically
    ) {
        Text(text = user.name)
        Spacer(modifier = Modifier.width(8.dp))
        Text(text = user.id.toString())
    }
}

@Preview(showBackground = true, showSystemUi = true)
@Composable
fun Preview(){

    val users2: List<User> = listOf(User(1,"user1"))
    SampleAppTheme {
        // A surface container using the 'background' color from the theme
        Surface(
            modifier = Modifier.fillMaxSize(),
            color = MaterialTheme.colorScheme.background
        ) {
            UsersContent(users = users2)
        }
    }
}
```

# UserViewModel

```kt
class UsersListViewModel : ViewModel() {

    fun getUsers() : Flow<List<User>> {
        val repo = UserRepository()
        return repo.getAll()
    }

}
```
