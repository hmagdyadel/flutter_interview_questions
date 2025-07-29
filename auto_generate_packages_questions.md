# Complete Flutter Guide: Retrofit, JSON Serialization & Freezed

## 🔗 Retrofit (with Dio)

### What is Retrofit in Flutter and how does it work with Dio?

Retrofit is a code generation library that creates type-safe HTTP API clients. It works with Dio by:
- Generating boilerplate code for API calls automatically
- Providing clean annotation-based API definitions (@GET, @POST, etc.)
- Handling request/response serialization seamlessly
- Creating maintainable, testable API service classes
- Leveraging Dio's powerful features (interceptors, error handling, logging)

### How do you annotate a Retrofit method for GET requests with path and query parameters?

```dart
@RestApi(baseUrl: "https://api.example.com")
abstract class ApiService {
  factory ApiService(Dio dio) = _ApiService;

  // Path parameter
  @GET("/users/{id}")
  Future<User> getUser(@Path("id") int userId);

  // Query parameters
  @GET("/users")
  Future<List<User>> getUsers(
    @Query("page") int page,
    @Query("limit") int limit,
    @Query("search") String? search, // Optional
  );

  // Combined path and query
  @GET("/users/{id}/posts")
  Future<List<Post>> getUserPosts(
    @Path("id") int userId,
    @Query("status") String status,
  );

  // Query map for dynamic parameters
  @GET("/search")
  Future<SearchResult> search(@Queries() Map<String, dynamic> filters);
}
```

### What is the purpose of the `part 'file.g.dart'` directive in Retrofit setup?

The `part` directive connects your abstract API class with the generated implementation:
- Links the generated `_ApiService` class with your abstract `ApiService`
- Enables code generation tools to create the implementation file
- Allows access to private members between the original and generated code
- Must match the filename pattern: if your file is `api_service.dart`, the part should be `'api_service.g.dart'`

### How do you generate Retrofit code in Flutter?

**Step 1:** Add dependencies to `pubspec.yaml`:
```yaml
dependencies:
  retrofit: ^4.0.3
  dio: ^5.3.2
  json_annotation: ^4.8.1

dev_dependencies:
  retrofit_generator: ^8.0.4
  build_runner: ^2.4.6
  json_serializable: ^6.7.1
```

**Step 2:** Run the code generation command:
```bash
flutter packages pub run build_runner build
# Or for watching changes
flutter packages pub run build_runner watch
```

### How would you handle request/response errors using Retrofit and Dio interceptors?

```dart
class ApiService {
  late Dio _dio;
  late ApiClient _apiClient;

  ApiService() {
    _dio = Dio(BaseOptions(
      baseUrl: 'https://api.example.com',
      connectTimeout: Duration(seconds: 5),
      receiveTimeout: Duration(seconds: 3),
    ));

    // Error handling interceptor
    _dio.interceptors.add(
      InterceptorsWrapper(
        onRequest: (options, handler) {
          // Add auth token
          options.headers['Authorization'] = 'Bearer $token';
          handler.next(options);
        },
        onResponse: (response, handler) {
          // Handle successful responses
          handler.next(response);
        },
        onError: (error, handler) {
          // Handle different error types
          if (error.response?.statusCode == 401) {
            // Handle unauthorized
            _refreshToken();
          } else if (error.response?.statusCode == 404) {
            // Handle not found
            print('Resource not found');
          }
          handler.next(error);
        },
      ),
    );

    _apiClient = ApiClient(_dio);
  }
}
```

### What are the advantages of using Retrofit over writing Dio manually?

**Retrofit advantages:**
- **Type Safety:** Compile-time error checking for API calls
- **Code Generation:** Automatic boilerplate code creation
- **Clean Code:** Declarative API definitions using annotations
- **Maintainability:** Centralized API endpoint management
- **Testing:** Easier to mock and test API services
- **Documentation:** Self-documenting API interface
- **Consistency:** Standardized approach across team members

### How can you implement dynamic base URLs in Retrofit (e.g., for staging vs production)?

```dart
class ApiConfig {
  static const String staging = 'https://staging-api.example.com';
  static const String production = 'https://api.example.com';
  
  static String get baseUrl {
    return kDebugMode ? staging : production;
  }
}

@RestApi()
abstract class ApiService {
  factory ApiService(Dio dio, {String? baseUrl}) = _ApiService;

  @GET("/users")
  Future<List<User>> getUsers();
}

// Usage
class ApiManager {
  late ApiService _apiService;
  
  ApiManager() {
    final dio = Dio();
    _apiService = ApiService(dio, baseUrl: ApiConfig.baseUrl);
  }
}
```

## 📄 json_annotation / json_serializable

### What is `@JsonSerializable()` used for in Flutter?

`@JsonSerializable()` is an annotation that generates JSON serialization code for Dart classes:
- **Automatic Generation:** Creates `toJson()` and `fromJson()` methods
- **Type Safety:** Ensures proper type conversion during serialization
- **Performance:** Generated code is faster than runtime reflection
- **Customization:** Allows configuration of serialization behavior
- **Error Handling:** Provides compile-time validation of JSON mappings

### What is the role of `@JsonKey(name: "some_key")` in a model class?

`@JsonKey` maps Dart property names to different JSON field names:
- **Field Mapping:** Maps `userName` to `"user_name"` in JSON
- **Nullable Handling:** Specifies default values and null handling
- **Custom Conversion:** Defines custom serialization logic
- **Field Exclusion:** Can ignore fields during serialization

```dart
@JsonSerializable()
class User {
  @JsonKey(name: 'user_id')
  final int id;
  
  @JsonKey(name: 'full_name')
  final String name;
  
  @JsonKey(name: 'email_address')
  final String email;
  
  @JsonKey(defaultValue: false)
  final bool isActive;
  
  @JsonKey(ignore: true)
  final String temporaryData;

  User({required this.id, required this.name, required this.email, 
        this.isActive = false, this.temporaryData = ''});
}
```

### How do you convert a model class to and from JSON using json_serializable?

```dart
@JsonSerializable()
class User {
  final int id;
  final String name;
  final String email;

  User({required this.id, required this.name, required this.email});

  // Generated methods (after build_runner)
  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
  Map<String, dynamic> toJson() => _$UserToJson(this);
}

// Usage:
// From JSON
final userJson = {'id': 1, 'name': 'John', 'email': 'john@example.com'};
final user = User.fromJson(userJson);

// To JSON
final json = user.toJson();
```

### What files are generated when using json_serializable, and how do you regenerate them?

**Generated Files:**
- `model_name.g.dart` - Contains the actual serialization implementation
- Private helper functions like `_$UserFromJson` and `_$UserToJson`

**Regeneration Commands:**
```bash
# One-time generation
flutter packages pub run build_runner build

# Watch for changes and auto-regenerate
flutter packages pub run build_runner watch

# Clean and rebuild
flutter packages pub run build_runner build --delete-conflicting-outputs
```

### How can you ignore a field from being serialized/deserialized in json_serializable?

```dart
@JsonSerializable()
class User {
  final int id;
  final String name;
  
  @JsonKey(ignore: true)
  final String password; // Never serialized
  
  @JsonKey(includeFromJson: false)
  final String computedField; // Only serialized to JSON
  
  @JsonKey(includeToJson: false)
  final String inputOnlyField; // Only deserialized from JSON

  User({required this.id, required this.name, 
        this.password = '', this.computedField = '', this.inputOnlyField = ''});
}
```

## ❄️ freezed / freezed_annotation

### What are the benefits of using `freezed` over traditional Dart classes?

**Freezed Benefits:**
- **Immutability:** All properties are final by default
- **Equality:** Automatic `==` and `hashCode` implementation
- **Copy Methods:** `copyWith()` for creating modified copies
- **Pattern Matching:** `when()` and `map()` methods for state handling
- **Union Types:** Multiple class variants in one sealed class
- **JSON Integration:** Seamless integration with json_serializable
- **Boilerplate Reduction:** Eliminates repetitive code writing

### How do you define immutable classes using `@freezed`?

```dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'user.freezed.dart';
part 'user.g.dart';

@freezed
class User with _$User {
  const factory User({
    required int id,
    required String name,
    required String email,
    @Default(false) bool isActive,
    @Default([]) List<String> tags,
  }) = _User;

  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
}

// Usage:
final user = User(id: 1, name: 'John', email: 'john@example.com');
final updatedUser = user.copyWith(isActive: true);
```

### What is the purpose of the `when`, `maybeWhen`, and `map` methods generated by `freezed`?

These methods provide pattern matching for union types:

```dart
@freezed
class ApiState<T> with _$ApiState<T> {
  const factory ApiState.loading() = Loading<T>;
  const factory ApiState.success(T data) = Success<T>;
  const factory ApiState.error(String message) = Error<T>;
}

// when() - handles all cases (exhaustive)
final result = apiState.when(
  loading: () => 'Loading...',
  success: (data) => 'Success: $data',
  error: (message) => 'Error: $message',
);

// maybeWhen() - handles some cases with orElse fallback
final result = apiState.maybeWhen(
  success: (data) => 'Got data: $data',
  orElse: () => 'Other state',
);

// map() - similar to when but returns the same type
final newState = apiState.map(
  loading: (loading) => ApiState<String>.loading(),
  success: (success) => ApiState<String>.success(success.data.toString()),
  error: (error) => ApiState<String>.error(error.message),
);
```

### How do you combine `freezed` with `json_serializable` to handle JSON serialization in models?

```dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'user.freezed.dart';
part 'user.g.dart';

@freezed
class User with _$User {
  const factory User({
    required int id,
    required String name,
    @JsonKey(name: 'email_address') required String email,
    @Default(false) bool isActive,
  }) = _User;

  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
}

// Union types with JSON
@freezed
class ApiResponse<T> with _$ApiResponse<T> {
  const factory ApiResponse.success(T data) = Success<T>;
  const factory ApiResponse.error(String message, int code) = Error<T>;

  factory ApiResponse.fromJson(Map<String, dynamic> json, T Function(Object?) fromJsonT) => 
    _$ApiResponseFromJson(json, fromJsonT);
}
```

### What is the use of generic types in `@freezed` states (like `PeopleStates<T>`) and how are they helpful in Bloc or Cubit?

Generic freezed classes provide type-safe state management:

```dart
@freezed
class DataState<T> with _$DataState<T> {
  const factory DataState.initial() = Initial<T>;
  const factory DataState.loading() = Loading<T>;
  const factory DataState.success(T data) = Success<T>;
  const factory DataState.error(String message) = Error<T>;
}

// In Cubit/Bloc
class UserCubit extends Cubit<DataState<List<User>>> {
  UserCubit() : super(const DataState.initial());

  Future<void> loadUsers() async {
    emit(const DataState.loading());
    try {
      final users = await apiService.getUsers();
      emit(DataState.success(users));
    } catch (e) {
      emit(DataState.error(e.toString()));
    }
  }
}

// In UI
BlocBuilder<UserCubit, DataState<List<User>>>(
  builder: (context, state) {
    return state.when(
      initial: () => Text('Press button to load'),
      loading: () => CircularProgressIndicator(),
      success: (users) => ListView.builder(
        itemCount: users.length,
        itemBuilder: (context, index) => ListTile(title: Text(users[index].name)),
      ),
      error: (message) => Text('Error: $message'),
    );
  },
)
```

## 🔁 retrofit_generator

### What is the purpose of `retrofit_generator`, and how does it interact with Retrofit annotations?

`retrofit_generator` is the code generation engine that:
- **Processes Annotations:** Reads @GET, @POST, @Path, @Query annotations
- **Generates Implementation:** Creates concrete classes from abstract API definitions
- **Handles Serialization:** Integrates with json_serializable for automatic JSON handling
- **Creates Dio Integration:** Generates Dio-based HTTP calls with proper error handling
- **Type Safety:** Ensures compile-time validation of API contracts

### What steps are required to generate Retrofit service classes automatically?

**Step 1:** Define abstract API class with annotations:
```dart
@RestApi(baseUrl: "https://api.example.com")
abstract class ApiService {
  factory ApiService(Dio dio) = _ApiService;
  
  @GET("/users")
  Future<List<User>> getUsers();
}
```

**Step 2:** Add part directive:
```dart
part 'api_service.g.dart';
```

**Step 3:** Run build runner:
```bash
flutter packages pub run build_runner build
```

**Step 4:** Use generated service:
```dart
final dio = Dio();
final apiService = ApiService(dio);
final users = await apiService.getUsers();
```

### How do you handle pagination or optional query parameters in a Retrofit service method?

```dart
@RestApi(baseUrl: "https://api.example.com")
abstract class ApiService {
  factory ApiService(Dio dio) = _ApiService;

  // Pagination with optional parameters
  @GET("/users")
  Future<PaginatedResponse<User>> getUsers({
    @Query("page") int page = 1,
    @Query("limit") int limit = 10,
    @Query("search") String? search,
    @Query("sort") String? sortBy,
  });

  // Using nullable parameters
  @GET("/posts")
  Future<List<Post>> getPosts(
    @Query("userId") int? userId,
    @Query("category") String? category,
    @Query("published") bool? isPublished,
  );

  // Dynamic query parameters
  @GET("/search")
  Future<SearchResult> search(@Queries() Map<String, dynamic> filters);
}

// Usage:
final users = await apiService.getUsers(page: 2, search: "john");
final posts = await apiService.getPosts(userId: 123);
final results = await apiService.search({"tag": "flutter", "year": 2024});
```

### How does `retrofit_generator` handle nested models or lists in the response?

The generator automatically handles complex response structures:

```dart
// Nested model classes
@JsonSerializable()
class Address {
  final String street;
  final String city;
  final String country;
  
  Address({required this.street, required this.city, required this.country});
  factory Address.fromJson(Map<String, dynamic> json) => _$AddressFromJson(json);
}

@JsonSerializable()
class User {
  final int id;
  final String name;
  final Address address;
  final List<String> hobbies;
  
  User({required this.id, required this.name, required this.address, required this.hobbies});
  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
}

@RestApi(baseUrl: "https://api.example.com")
abstract class ApiService {
  factory ApiService(Dio dio) = _ApiService;

  // Handles nested User with Address
  @GET("/users/{id}")
  Future<User> getUser(@Path("id") int id);

  // Handles list of complex objects
  @GET("/users")
  Future<List<User>> getUsers();

  // Handles paginated response with nested data
  @GET("/users/paginated")
  Future<PaginatedResponse<User>> getPaginatedUsers();
}
```

## 🧪 Bonus: Real-World Scenario Questions

### How would you structure your Dio client with multiple base URLs and token refresh handling?

```dart
class ApiManager {
  static ApiManager? _instance;
  late Dio _authDio;
  late Dio _publicDio;
  late AuthService _authService;
  late PublicService _publicService;
  
  String? _accessToken;
  String? _refreshToken;

  ApiManager._internal() {
    _setupDioClients();
  }

  static ApiManager get instance {
    _instance ??= ApiManager._internal();
    return _instance!;
  }

  void _setupDioClients() {
    // Public API client (no auth required)
    _publicDio = Dio(BaseOptions(
      baseUrl: 'https://public-api.example.com',
      connectTimeout: Duration(seconds: 5),
      receiveTimeout: Duration(seconds: 3),
    ));

    // Authenticated API client
    _authDio = Dio(BaseOptions(
      baseUrl: 'https://api.example.com',
      connectTimeout: Duration(seconds: 5),
      receiveTimeout: Duration(seconds: 3),
    ));

    // Add auth interceptor
    _authDio.interceptors.add(
      InterceptorsWrapper(
        onRequest: (options, handler) async {
          if (_accessToken != null) {
            options.headers['Authorization'] = 'Bearer $_accessToken';
          }
          handler.next(options);
        },
        onError: (error, handler) async {
          if (error.response?.statusCode == 401) {
            // Token expired, try to refresh
            if (await _refreshTokens()) {
              // Retry original request
              final retryRequest = error.requestOptions;
              retryRequest.headers['Authorization'] = 'Bearer $_accessToken';
              try {
                final response = await _authDio.request(
                  retryRequest.path,
                  options: Options(
                    method: retryRequest.method,
                    headers: retryRequest.headers,
                  ),
                  data: retryRequest.data,
                  queryParameters: retryRequest.queryParameters,
                );
                handler.resolve(response);
                return;
              } catch (e) {
                // Refresh failed, logout user
                _logout();
              }
            }
          }
          handler.next(error);
        },
      ),
    );

    _authService = AuthService(_authDio);
    _publicService = PublicService(_publicDio);
  }

  Future<bool> _refreshTokens() async {
    if (_refreshToken == null) return false;
    
    try {
      final response = await _publicDio.post('/auth/refresh', data: {
        'refresh_token': _refreshToken,
      });
      
      _accessToken = response.data['access_token'];
      _refreshToken = response.data['refresh_token'];
      
      // Store tokens securely
      await _storeTokens();
      return true;
    } catch (e) {
      return false;
    }
  }

  void _logout() {
    _accessToken = null;
    _refreshToken = null;
    // Navigate to login screen
  }

  Future<void> _storeTokens() async {
    // Store tokens in secure storage
  }
}
```

### How would you test your Retrofit-based API services?

```dart
// Mock API service for testing
class MockApiService extends Mock implements ApiService {}

void main() {
  group('API Service Tests', () {
    late MockApiService mockApiService;
    late UserRepository userRepository;

    setUp(() {
      mockApiService = MockApiService();
      userRepository = UserRepository(mockApiService);
    });

    test('should return users when API call is successful', () async {
      // Arrange
      final mockUsers = [
        User(id: 1, name: 'John', email: 'john@example.com'),
        User(id: 2, name: 'Jane', email: 'jane@example.com'),
      ];
      when(() => mockApiService.getUsers()).thenAnswer((_) async => mockUsers);

      // Act
      final result = await userRepository.getUsers();

      // Assert
      expect(result.length, 2);
      expect(result.first.name, 'John');
      verify(() => mockApiService.getUsers()).called(1);
    });

    test('should handle API errors gracefully', () async {
      // Arrange
      when(() => mockApiService.getUsers()).thenThrow(
        DioException(
          requestOptions: RequestOptions(path: '/users'),
          response: Response(
            requestOptions: RequestOptions(path: '/users'),
            statusCode: 500,
          ),
        ),
      );

      // Act & Assert
      expect(() => userRepository.getUsers(), throwsA(isA<ApiException>()));
    });
  });

  group('Integration Tests', () {
    late Dio dio;
    late ApiService apiService;

    setUp(() {
      dio = Dio();
      // Use test base URL
      dio.options.baseUrl = 'https://test-api.example.com';
      apiService = ApiService(dio);
    });

    test('should make real API call in integration test', () async {
      final users = await apiService.getUsers();
      expect(users, isNotNull);
      expect(users, isA<List<User>>());
    });
  });
}
```

### What are the challenges of using code generation (`.g.dart`) in a CI/CD pipeline, and how do you manage them?

**Challenges:**
- **Build Dependencies:** Generated files must be created before compilation
- **Version Control:** Deciding whether to commit generated files
- **Build Time:** Code generation adds time to CI/CD pipeline
- **Cache Management:** Generated files can break caching strategies
- **Merge Conflicts:** Generated files can cause merge issues

**Solutions:**

```yaml
# .github/workflows/flutter.yml
name: CI/CD Pipeline

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Flutter
      uses: subosito/flutter-action@v2
      with:
        flutter-version: '3.16.0'
    
    - name: Get dependencies
      run: flutter pub get
    
    - name: Generate code
      run: flutter packages pub run build_runner build --delete-conflicting-outputs
    
    - name: Run tests
      run: flutter test
    
    - name: Build APK
      run: flutter build apk --release

  # Optional: Cache generated files
  cache-generated:
    runs-on: ubuntu-latest
    steps:
    - name: Cache generated files
      uses: actions/cache@v3
      with:
        path: |
          **/*.g.dart
          **/*.freezed.dart
        key: ${{ runner.os }}-generated-${{ hashFiles('**/pubspec.lock') }}
```

**Git Strategy for Generated Files:**
```gitignore
# Option 1: Ignore generated files (recommended)
*.g.dart
*.freezed.dart

# Option 2: Commit generated files (alternative)
# Comment out the above lines and commit generated files
```

**Build Script for Team Consistency:**
```bash
#!/bin/bash
# scripts/generate.sh

echo "🔄 Cleaning previous builds..."
flutter clean

echo "📦 Getting dependencies..."
flutter pub get

echo "⚙️ Generating code..."
flutter packages pub run build_runner build --delete-conflicting-outputs

echo "✅ Code generation complete!"
```