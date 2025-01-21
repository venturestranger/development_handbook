# Frontend Development Guide

###### Outline:

1. Hybrid Folder Hierarchy
2. Folder Content Examples
3. Explanatory Pipeline



## Hybrid Folder Hierarchy:

```
lib/
├── core/
│   ├── data/
│   │   ├── models/
│   │   ├── services/
│   │   ├── utils/
│   │   └── repositories/ (implementation)
│   ├── domain/
│   │   ├── entities/
│   │   ├── use_cases/
│   │   └── repositories/ (abstraction)
│   └── presentation/
│       ├── widgets/
│       ├── themes/
│       └── localization/
├── features/
│   ├── authentication/
│   │   ├── data/
│   │   │   └── models/
│   │   ├── domain/
│   │   │   └── use_cases/
│   │   └── presentation/
│   │       ├── screens/
│   │       └── widgets/
│   ├── chat/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   └── profile/
│       ├── data/
│       ├── domain/
│       ├── presentation/
│       └── views/
└── main.dart
```



### Folder Content Examples:

---

### 1. `/data/models/`
This folder contains **data transfer objects (DTOs)** and raw representations of data, typically used for network communication or database storage.

**Examples**:  
- `user_model.dart`: Defines the structure of a user as retrieved from an API.  
- `post_model.dart`: Represents a blog post or article in JSON format.  
- `auth_response.dart`: Maps responses from an authentication API to an object.

---

### 2. `/data/services/` 
This folder contains **classes that handle low-level operations** related to external systems like APIs, local databases, or file systems.

**Examples**:  
- `api_service.dart`: Handles HTTP requests to an external API.  
- `database_service.dart`: Manages SQLite or other local database operations.  
- `file_service.dart`: Handles file I/O operations (e.g., reading/writing files).  

---

### 3. `/data/utils/` 
This folder contains **helper functions, constants, and utilities** that are reusable across the project.

**Examples**:  
- `constants.dart`: Contains constants like API endpoints, default values, or app-wide keys.  
- `validators.dart`: Provides functions for validating inputs (e.g., email format validation).  
- `date_utils.dart`: Utilities for date formatting and parsing.

---

### 4. `/data/repositories/` 

This folder contains **implementation of interfaces (see p. 7)**, they also define the functionality of data contracts. These interfaces are **NOT AGNOSTIC** of the implementation details.

**Examples**:  

- `auth_repository.dart`: Implements methods like `login(email, password)` or `logout()`.  
- `post_repository.dart`: Implements functions like `fetchPosts()` or `createPost(Post post)`.  
- `profile_repository.dart`: Implements functions like `getProfile()` or `updateProfile(Profile profile)`.

---

### 5. `/domain/entities/`
This folder contains **plain Dart objects (POJOs)** that represent the core business logic without dependencies on data or UI layers.

**Examples**:  
- `user.dart`: A user entity with core business attributes (e.g., `id`, `name`, `email`).  
- `post.dart`: Represents a blog post or article in the domain context.  
- `auth.dart`: Represents authentication-related data (e.g., `token`, `expiry`).

---

### 6. `/domain/use_cases/` 
This folder contains **classes encapsulating specific business logic** or workflows. Each class represents a single use case in the application.

**Examples**:  
- `login_use_case.dart`: Handles the login process by interacting with a repository.  
- `fetch_posts_use_case.dart`: Retrieves a list of posts and performs any necessary processing.  
- `update_profile_use_case.dart`: Updates a user’s profile details.

---

### 7. `/domain/repositories/` 
This folder contains **interfaces** defining the contracts that the data layer must implement. These interfaces are **AGNOSTIC** of the implementation details.

**Examples**:  

- `auth_repository.dart`: Defines methods like `login(email, password)` or `logout()`.  
- `post_repository.dart`: Declares functions like `fetchPosts()` or `createPost(Post post)`.  
- `profile_repository.dart`: Specifies functions like `getProfile()` or `updateProfile(Profile profile)`.

---

### 8. `/presentation/widgets/`
This folder contains **reusable UI components** (stateless or stateful widgets) that are used across different screens.

**Examples**:  
- `custom_button.dart`: A reusable button widget with custom styling.  
- `loading_spinner.dart`: A widget showing a loading indicator.  
- `error_message.dart`: A widget to display error messages consistently across the app.

---

### 9. `/presentation/themes/`  
This folder contains **theme and style-related configurations** like colors, typography, and app-wide themes.

**Examples**:  
- `app_theme.dart`: Defines the app's light and dark themes.  
- `colors.dart`: Contains color constants for the app (e.g., `primaryColor`, `accentColor`).  
- `text_styles.dart`: Specifies text styles used across the app (e.g., `heading1`, `bodyText`).

---

### 10. `/presentation/localization/` 
This folder contains **localization files and utilities** for supporting multiple languages.

**Examples**:  
- `app_localizations.dart`: Handles translation and loading of localized content.  
- `en.json`: A JSON file containing English translations.  
- `es.json`: A JSON file containing Spanish translations.

---

### Example Files for Each Folder

#### `/data/models/user_model.dart`
```dart
class UserModel {
  final String id;
  final String name;
  final String email;

  UserModel({required this.id, required this.name, required this.email});

  factory UserModel.fromJson(Map<String, dynamic> json) {
    return UserModel(
      id: json['id'],
      name: json['name'],
      email: json['email'],
    );
  }
}
```

#### `/data/services/api_service.dart`
```dart
import 'package:http/http.dart' as http;

class ApiService {
  final String baseUrl;

  ApiService(this.baseUrl);

  Future<http.Response> get(String endpoint) {
    return http.get(Uri.parse('$baseUrl$endpoint'));
  }
}
```

#### `/data/utils/constants.dart`
```dart
const String apiBaseUrl = "https://api.example.com/";
const int defaultTimeout = 5000;
```

#### `/data/repositories/user_repository.dart`
```dart
import '../../domain/repositories/user_repository.dart';

class UserRepositoryImpl implements UserRepository {
  final UserService userService;

  UserRepositoryImpl(this.userService);

  @override
  Future<User> getUserById(String id) async {
    ...
  }
}
```

#### `/domain/entities/user.dart`
```dart
class User {
  final String id;
  final String name;
  final String email;

  User({required this.id, required this.name, required this.email});
}
```

#### `/domain/use_cases/login_use_case.dart`
```dart
import '../repositories/auth_repository.dart';

class LoginUseCase {
  final AuthRepository authRepository;

  LoginUseCase(this.authRepository);

  Future<void> execute(String email, String password) async {
    return await authRepository.login(email, password);
  }
}
```

#### `/domain/repositories/auth_repository.dart`
```dart
abstract class AuthRepository {
  Future<void> login(String email, String password);
  Future<void> logout();
}
```

#### `/presentation/widgets/custom_button.dart`
```dart
import 'package:flutter/material.dart';

class CustomButton extends StatelessWidget {
  final String text;
  final VoidCallback onPressed;

  const CustomButton({required this.text, required this.onPressed});

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: onPressed,
      child: Text(text),
    );
  }
}
```

#### `/presentation/themes/colors.dart`
```dart
import 'package:flutter/material.dart';

const Color primaryColor = Color(0xFF6200EE);
const Color accentColor = Color(0xFF03DAC6);
```

#### `/presentation/localization/en.json`
```json
{
  "welcome": "Welcome",
  "login": "Login",
  "logout": "Logout"
}
```





## Explanatory Pipeline

Fetching user data from an API in a clean architecture involves a **sequence of actions** across different layers and components:

---

### **1. Data Layer**
#### **a. Create the Service**
- **Responsibility**: Fetch raw data from the API.  
- **Action**: Define a method in your service to make the network request.

```dart
class UserService {
  final String baseUrl;

  UserService(this.baseUrl);

  Future<Map<String, dynamic>> fetchUserById(String id) async {
    final response = await http.get(Uri.parse('$baseUrl/users/$id'));
    if (response.statusCode == 200) {
      return json.decode(response.body);
    } else {
      throw Exception('Failed to load user');
    }
  }
}
```

#### **b. Create the Model**
- **Responsibility**: Represent raw data and handle serialization.  
- **Action**: Define a model that maps the JSON response.

```dart
class UserModel {
  final String id;
  final String name;
  final String email;

  UserModel({required this.id, required this.name, required this.email});

  factory UserModel.fromJson(Map<String, dynamic> json) {
    return UserModel(
      id: json['id'],
      name: json['name'],
      email: json['email'],
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'email': email,
    };
  }
}
```

#### **c. Implement the Repository**
- **Responsibility**: Abstract data fetching and convert models to domain entities.  
- **Action**: Implement the repository interface with a concrete implementation.

```dart
class UserRepositoryImpl implements UserRepository {
  final UserService userService;

  UserRepositoryImpl(this.userService);

  @override
  Future<User> getUserById(String id) async {
    final userJson = await userService.fetchUserById(id);
    final userModel = UserModel.fromJson(userJson);
    return User(
      id: userModel.id,
      name: userModel.name,
      email: userModel.email,
    );
  }
}
```

---

### **2. Domain Layer**
#### **a. Create the Entity**
- **Responsibility**: Represent the core business concept.  
- **Action**: Define a clean domain entity.

```dart
class User {
  final String id;
  final String name;
  final String email;

  User({required this.id, required this.name, required this.email});
}
```

#### **b. Define the Repository Interface**
- **Responsibility**: Provide a contract for the repository.  
- **Action**: Create an abstract interface that will be implemented in the data layer.

```dart
abstract class UserRepository {
  Future<User> getUserById(String id);
}
```

#### **c. Define the Use Case**
- **Responsibility**: Encapsulate the business logic for fetching a user.  
- **Action**: Implement the use case logic that calls the repository.

```dart
class GetUserUseCase {
  final UserRepository userRepository;

  GetUserUseCase(this.userRepository);

  Future<User> execute(String id) {
    return userRepository.getUserById(id);
  }
}
```

---

### **3. Presentation Layer**
#### **a. Create the ViewModel/Controller**
- **Responsibility**: Manage the state and handle UI interaction.  
- **Action**: Define a method to call the use case and update the state.

```dart
class UserViewModel {
  final GetUserUseCase getUserUseCase;

  UserViewModel(this.getUserUseCase);

  Future<void> fetchUser(String id) async {
    try {
      final user = await getUserUseCase.execute(id);
      // Update UI state with the fetched user
      print('User fetched: ${user.name}');
    } catch (e) {
      print('Error fetching user: $e');
    }
  }
}
```

#### **b. Bind to the UI**
- **Responsibility**: Display data to the user.  
- **Action**: Call the ViewModel or Controller and display the result in the UI.

```dart
class UserPage extends StatelessWidget {
  final UserViewModel viewModel;

  UserPage(this.viewModel);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('User')),
      body: FutureBuilder<void>(
        future: viewModel.fetchUser('1'),
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return Center(child: CircularProgressIndicator());
          } else if (snapshot.hasError) {
            return Center(child: Text('Error: ${snapshot.error}'));
          } else {
            return Center(child: Text('User data displayed here'));
          }
        },
      ),
    );
  }
}
```

---

### **Sequence of Actions**

1. **Presentation Layer**:
   - User interacts with the UI (e.g., presses a button to fetch user data).
   - The UI calls a method in the ViewModel/Controller.

2. **Domain Layer**:
   - The ViewModel/Controller calls the `GetUserUseCase`.
   - The use case calls the `UserRepository` interface to fetch the user.

3. **Data Layer**:
   - The `UserRepositoryImpl` implementation uses the `UserService` to make an API call.
   - The `UserService` fetches raw JSON data from the API.
   - The `UserModel` converts the raw JSON into a structured object.
   - The `UserRepositoryImpl` converts the `UserModel` into a `User` domain entity.

4. **Return to Presentation Layer**:
   - The `GetUserUseCase` returns the `User` entity to the ViewModel/Controller.
   - The ViewModel updates the UI state with the fetched data.

---

## Application State Management

Application uses four technologies to manage state and handle both cached and temporary data, such as user-related information, tokens, and preferences.

### Overview

1. **Provider**  
   - **Type**: In-memory, temporary storage.
   - **Purpose**: To manage in-session variables and buffers that need to trigger UI updates.
   - **Use Cases**: Managing application state throughout the session (e.g., user authentication state, theme settings).
   - **Example**: Changing the app's theme mode (light/dark) or monitoring user login status.

2. **Secure Storage**  
   - **Type**: On-disk, secured, persistent storage.
   - **Purpose**: To store sensitive data like tokens and authorization information securely.
   - **Use Cases**: Saving user authentication tokens or other private data that must persist across sessions.

3. **Hive**  
   - **Type**: On-disk, persistent storage.
   - **Purpose**: To store complex data structures and cache API responses locally.
   - **Use Cases**: Saving user profiles, lists, or other objects that require structured storage.
   - **Example**: Caching API data for offline access or storing user details.

4. **SharedPreferences**  
   - **Type**: On-disk, persistent storage.
   - **Purpose**: To store simple key-value pairs, such as user preferences and environment settings.
   - **Use Cases**: Storing theme mode, language preferences, or flags indicating tutorial completion.
   - **Example**: Saving the user's preferred language or app theme.

### When to Use Each Technology

#### **Provider**
- Best for temporary, in-memory state management.
- Triggers reactive UI updates when state changes.
- Ideal for managing data that is only relevant during the app session.

#### **Secure Storage**
- Use for sensitive, persistent data that must remain secure.
- Ensures that tokens and authorization information are encrypted on disk.

#### **Hive**
- Use for local persistent storage of complex data structures.
- Suitable for storing small to medium-sized data that needs to persist across app restarts.
- Efficient for offline data caching and quick retrieval.

#### **SharedPreferences**
- Best for simple key-value pairs, such as user preferences or environmental flags.
- Lightweight and straightforward for basic storage needs.

### Practical Examples

1. **SharedPreferences**: Store simple user preferences.
   - Example: "Save the app theme mode (light or dark)."

2. **Hive**: Store complex or cached data.
   - Example: "Save a user's profile details or a list of items retrieved from an API."

3. **Secure Storage**: Store sensitive data securely.
   - Example: "Save the user's login token."

4. **Provider**: Manage temporary in-session states.
   - Example: "Track whether the user is logged in and update the UI accordingly."

## Summary

- Use **SharedPreferences** for lightweight, simple key-value storage (e.g., theme mode, language settings).
- Use **Hive** for complex data structures and offline caching (e.g., user profiles, cached API responses).
- Use **Secure Storage** for securely storing sensitive data (e.g., tokens, authentication details).
- Use **Provider** for managing temporary, reactive state that needs to update the UI (e.g., app-wide state like login status or theme settings).

```dart
import 'package:hive/hive.dart';
import 'package:hive_flutter/hive_flutter.dart';
import 'package:flutter_secure_storage/flutter_secure_storage.dart';
import 'package:shared_preferences/shared_preferences.dart';

// Setup Flutter Secure Storage and Hive
final FlutterSecureStorage secureStorage = FlutterSecureStorage();

Future<void> initApp() async {
  await Hive.initFlutter();
  await Hive.openBox('sharedBox');
}

Future<void> saveSharedPreferences(String key, String value) async {
  SharedPreferences prefs = await SharedPreferences.getInstance();
  await prefs.setString(key, value);
}

Future<String?> getSharedPreferences(String key) async {
  SharedPreferences prefs = await SharedPreferences.getInstance();
  return prefs.getString(key);
}

Future<void> deleteSharedPreferences(String key) async {
  SharedPreferences prefs = await SharedPreferences.getInstance();
  await prefs.remove(key);
}

Future<void> saveSecureData(String key, String value) async {
  await secureStorage.write(key: key, value: value);
}

Future<String?> getSecureData(String key) async {
  return await secureStorage.read(key: key);
}

Future<void> deleteSecureData(String key) async {
  await secureStorage.delete(key: key);
}

Future<void> saveToHive(String key, String value) async {
  var box = await Hive.openBox('sharedBox');
  await box.put(key, value);
}

Future<String?> getFromHive(String key) async {
  var box = await Hive.openBox('sharedBox');
  return box.get(key);
}

Future<void> deleteFromHive(String key) async {
  var box = await Hive.openBox('sharedBox');
  await box.delete(key);
}
```
