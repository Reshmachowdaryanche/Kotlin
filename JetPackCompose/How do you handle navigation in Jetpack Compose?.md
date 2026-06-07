
### How do you handle navigation in Jetpack Compose?

Jetpack Compose navigation is handled using the Navigation Compose library. We create a `NavController` to manage navigation actions and a `NavHost` to define the navigation graph. Each screen is declared as a `composable` destination with a unique route.

For type-safe navigation, I usually define routes using a sealed class. Arguments can be passed through route parameters and retrieved from the `BackStackEntry`. Back navigation is handled using `navController.popBackStack()`.

**Example:**

```kotlin
sealed class Screen(val route: String) {

    object Home : Screen("home")

    object Profile : Screen("profile/{name}") {

        fun createRoute(name: String): String {
            return "profile/$name"
        }
    }
}
```

Navigation graph:

```kotlin
@Composable
fun NavigationApp() {

    val navController = rememberNavController()

    NavHost(
        navController = navController,
        startDestination = Screen.Home.route
    ) {

        composable(Screen.Home.route) {

            HomeScreen(
                onNavigateToProfile = { name ->
                    navController.navigate(
                        Screen.Profile.createRoute(name)
                    )
                }
            )
        }

        composable(
            route = Screen.Profile.route,
            arguments = listOf(
                navArgument("name") {
                    type = NavType.StringType
                }
            )
        ) { backStackEntry ->

            val name =
                backStackEntry.arguments?.getString("name")
                    ?: "Unknown"

            ProfileScreen(
                name = name,
                onBackClick = {
                    navController.popBackStack()
                }
            )
        }
    }
}
```

Flow:

```text
HomeScreen
    |
    | navigate("profile/Prasad")
    V
ProfileScreen(name = "Prasad")
    |
    | popBackStack()
    V
HomeScreen
```

### Key Points

* `NavController` manages navigation actions.
* `NavHost` contains all navigation destinations.
* `composable()` defines individual screens.
* Sealed classes provide type-safe route management.
* Arguments are passed using route parameters such as `"profile/{name}"`.
* Arguments are retrieved using `backStackEntry.arguments`.
* Back navigation is handled using `navController.popBackStack()`.

### Short Interview Answer (1-minute version)

> In Jetpack Compose, navigation is handled using Navigation Compose. I create a `NavController` using `rememberNavController()` and define destinations inside a `NavHost`. Each screen is represented by a composable route. I typically use a sealed class to manage routes in a type-safe manner. For passing data between screens, I use navigation arguments, such as `"profile/{name}"`, and retrieve them from the `BackStackEntry`. Back navigation is handled using `navController.popBackStack()`. This approach keeps navigation scalable, maintainable, and aligned with Compose best practices.
