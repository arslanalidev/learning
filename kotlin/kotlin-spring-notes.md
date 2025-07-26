# 🌱 Kotlin + Spring Framework Notes

*Quick revision guide for Kotlin Spring development*

---

## 🚀 Getting Started

### Why Kotlin + Spring?
- **Concise syntax** - Less boilerplate than Java
- **Null safety** - Prevents NPE at compile time
- **Interoperable** - Works seamlessly with existing Java code
- **Coroutines** - Built-in async programming support

### Basic Setup
```kotlin
// build.gradle.kts
dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.jetbrains.kotlin:kotlin-reflect")
    implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
}
```

---

## 🏗️ Core Concepts

### 1. Controllers
```kotlin
@RestController
@RequestMapping("/api")
class UserController {
    
    @GetMapping("/users/{id}")
    fun getUser(@PathVariable id: Long): User {
        return userService.findById(id)
    }
    
    @PostMapping("/users")
    fun createUser(@RequestBody user: User): User {
        return userService.save(user)
    }
}
```

**Key Points:**
- Use `@RestController` for REST APIs
- Path variables with `@PathVariable`
- Request body with `@RequestBody`

### 2. Data Classes
```kotlin
@Entity
data class User(
    @Id @GeneratedValue
    val id: Long? = null,
    val name: String,
    val email: String
)
```

**Benefits:**
- Auto-generates `equals()`, `hashCode()`, `toString()`
- Perfect for DTOs and entities
- Immutable by default

### 3. Services & Dependency Injection
```kotlin
@Service
class UserService(
    private val userRepository: UserRepository
) {
    fun findById(id: Long): User = 
        userRepository.findById(id).orElseThrow()
}
```

**Remember:**
- Constructor injection is preferred
- No need for `@Autowired` in constructor

---

## 🗄️ Database Integration

### JPA Repositories
```kotlin
@Repository
interface UserRepository : JpaRepository<User, Long> {
    fun findByEmail(email: String): User?
    fun findByNameContaining(name: String): List<User>
}
```

### Configuration
```kotlin
@Configuration
class DatabaseConfig {
    
    @Bean
    @Primary
    fun dataSource(): DataSource {
        return HikariDataSource().apply {
            jdbcUrl = "jdbc:postgresql://localhost/mydb"
            username = "user"
            password = "pass"
        }
    }
}
```

---

## ⚡ Advanced Features

### 1. Coroutines for Async
```kotlin
@Service
class AsyncUserService {
    
    suspend fun processUsers(): List<User> = withContext(Dispatchers.IO) {
        // Heavy database operation
        userRepository.findAll()
    }
}
```

### 2. Extension Functions
```kotlin
fun User.toDto(): UserDto = UserDto(
    id = this.id,
    name = this.name,
    email = this.email
)
```

### 3. Validation
```kotlin
data class CreateUserRequest(
    @field:NotBlank
    val name: String,
    
    @field:Email
    val email: String
)
```

---

## 🛡️ Security Basics

### WebSecurity Configuration
```kotlin
@Configuration
@EnableWebSecurity
class SecurityConfig {
    
    @Bean
    fun filterChain(http: HttpSecurity): SecurityFilterChain {
        return http
            .csrf().disable()
            .authorizeHttpRequests {
                it.requestMatchers("/api/public/**").permitAll()
                  .anyRequest().authenticated()
            }
            .build()
    }
}
```

---

## 🧪 Testing

### Unit Tests
```kotlin
@WebMvcTest(UserController::class)
class UserControllerTest {
    
    @Autowired
    lateinit var mockMvc: MockMvc
    
    @MockBean
    lateinit var userService: UserService
    
    @Test
    fun `should return user when exists`() {
        // Given
        val user = User(1L, "John", "john@email.com")
        every { userService.findById(1L) } returns user
        
        // When & Then
        mockMvc.get("/api/users/1")
            .andExpect { status { isOk() } }
            .andExpect { jsonPath("$.name") { value("John") } }
    }
}
```

---

## 💡 Best Practices

### ✅ Do's
- Use data classes for DTOs
- Leverage null safety (`?` and `!!`)
- Prefer immutable collections
- Use sealed classes for state management
- Write extension functions for common operations

### ❌ Don'ts
- Don't use `!!` unless absolutely sure
- Avoid mutable collections in APIs
- Don't ignore Kotlin conventions
- Don't mix Java and Kotlin styles unnecessarily

---

## 🔧 Common Patterns

### 1. Result Handling
```kotlin
sealed class Result<T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Error<T>(val message: String) : Result<T>()
}

fun handleUser(result: Result<User>) = when(result) {
    is Result.Success -> ResponseEntity.ok(result.data)
    is Result.Error -> ResponseEntity.badRequest().body(result.message)
}
```

### 2. Configuration Properties
```kotlin
@ConfigurationProperties(prefix = "app")
data class AppProperties(
    val name: String,
    val version: String,
    val database: DatabaseConfig
) {
    data class DatabaseConfig(
        val url: String,
        val maxConnections: Int
    )
}
```

---

## 📚 Quick Reference

| Feature | Kotlin Way | Notes |
|---------|------------|-------|
| Null Safety | `String?` | Use nullable types |
| Collections | `listOf()`, `mapOf()` | Immutable by default |
| Functions | `fun name(): Type` | Return type inference |
| Properties | `val` / `var` | Immutable / Mutable |
| String Templates | `"Hello $name"` | Embedded expressions |

---

## 🎯 Key Takeaways

1. **Kotlin makes Spring more concise** - Less boilerplate, more productivity
2. **Null safety prevents bugs** - Catch NPEs at compile time
3. **Data classes are perfect for DTOs** - Auto-generated methods
4. **Coroutines enable better async** - Non-blocking operations
5. **Interoperability is seamless** - Mix with existing Java code

---

*Happy coding with Kotlin + Spring! 🚀*
