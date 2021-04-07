# Naming conventions

1. Classes, we name the classes **files** using lower under scored names, i.e. `auth_service.dart`and `chat_screen.dart`
2. Classes names themselves follow an Upper-Camel-Case naming convention. i.e. `class AuthService` and `ChatScreen`
3. Each class MUST have in it's name it's location in the architecture. i.e. `auth_service.dart` and the class name `AuthService` have the last section of there names `Service` indicating the location in the architecture.
4. Use privates when ever possible
5. Optional: use $ at the end of the name when it's an observable.