# Authentication Module

Authentication in our apps follow a very specific set of steps.
These steps are repeated for every login and done using the module `auth_module`.

## Setup

In order to use the Authentication Module, First setup the project correctly with Firebase [Firebase Initilization](./init-firebase.md)

Then copy the `auth_module` from the boilerplate into your own project.

Then fix all the dependecies error, these are the imports.

Now you can use it. Auth is used in 1 of 2 ways, either you want the user to login via the predefined UI or you want to just listen to the Authentication Events, depending on where you are in the application.

## Using the UI

to start using the UI, first add the navigation component into the widget tree.
This can be done by injecting `AuthModule` into `MyApp` widget and then importing the route into `fullRoutesMap` like

```dart
fullRoutesMap.addAll(AuthModule.getRoutes());
```

Now you can access the UI from wherever you are using the simple command

```dart
Navigator.of(context).pushNamed(AuthorizationRoutes.AUTH_SCREEN)
```

## Using the service

To start using the service, it's auto injected, nothing is required to use it.
Then start listening to the auth events which comes in the following form:

1. NOT_LOGGED_IN
2. UNVERIFIED
3. AUTHORIZED
4. CODE_SENT
5. CODE_TIMEOUT

You can listen to these changes anyware using the stream `authListener`.
Note that these states are all of type enum `AuthState`.

### Public Facing Methods

In the auth service we have many methods which can be used to log users in and out.
Currently we support 5 login methods, and we are planning to add 3 more in the future.

Those are

1. Email Password Authorization.
2. Email Magic Link.
3. Phone Number
4. Google
5. Apple

All of which can be access by:

1. Injecting the `AuthService` into the class.
2. Requesting the method
3. Listening to changes of the Authorization Status from the `authListener`.

Note: We support multiple users types in our API, that's why we are requiring the `UserRole` in all of these functions. If you have 1 user type, just user `UserRole.ROLE_OWNER` or modify the enum to suite your needs.

Note 2: We auto-register users, so you should always include the "I agree on the privacy policy and terms of service" check box when logging in.

Note 3: In case something happen "Other than 403 Error" the error is delivered into firebase crashlytics console. as long as it is activated correctly and enabled there.

Happy Authorization! :joy:
