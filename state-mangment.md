# Managing a Screen State

Here we are talking about 3 separate roles inside the application. we have the screen, the state and the state manager.

We also have a widget which is more of a view to group things together, that is why I will not mention it here.

## Screen

A screen is a **routed** widget inside the app. no design should ever be inside this class, it is just a razer-thin component to hold date for the app to navigate to.

An example of this might be:

```dart
@provide
class LoginScreen extends StatefulWidget {
  final LoginStateManager _stateManager;

  LoginScreen(this._stateManager);

  @override
  LoginScreenState createState() => LoginScreenState();
}

class LoginScreenState extends State<LoginScreen> {
  LoginState _currentStates;
  StreamSubscription _stateSubscription;

  void refresh() {
    if (mounted) setState(() {});
  }

  @override
  void initState() {
    super.initState();
    _currentStates = LoginStateInit(this);
    _stateSubscription = widget._stateManager.stateStream.listen((event) {
      if (mounted) {
        setState(() {
          _currentStates = event;
        });
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: _currentStates.getUI(context),
      ),
    );
  }

  @override
  void dispose() {
    _stateSubscription.cancel();
    super.dispose();
  }
}

```

Notice that the screen have nothing to do with the actual design of the app,

This is inspired partially by the Activity-Fragment duality inside android, and by component-module duality in Angular 2+. what this means is that this is a widely used design pattern we have adopted inside our flutter app.

## States

States are located inside the `ui/states` folder. we always should have at least 2 states for any kind of screen:

1. `AbstractState` to encapsulate the state declaration

   ```dart
   abstract class RegisterState {
     final RegisterScreenState screen;
     RegisterState(this.screen);
   
     Widget getUI(BuildContext context);
   }
   ```

   These always follow the same style, just different names.

2. `ImplementedState` to show the actual state

   ```dart
   class RegisterStatePhoneCodeSent extends RegisterState {
     bool loading = false;
   
     RegisterStatePhoneCodeSent(RegisterScreenState screen) : super(screen);
   
     @override
     Widget getUI(BuildContext context) {
       return Form(
         child: Flex(
           direction: Axis.vertical,
           mainAxisAlignment: MainAxisAlignment.spaceBetween,
           crossAxisAlignment: CrossAxisAlignment.center,
           children: [
             OutlinedButton(
               onPressed: retryEnabled
                   ? () {
                       screen.retryPhone();
                     }
                   : null,
               child: Text(S.of(context).resendCode),
             ),
               //...
           ],
         ),
       );
     }
   }
   
   ```

It's good here to acknowledge something: As of now, we have to use screen for context, this is used for the translation and what not. as a future plan we shall implement without this dependencies even though architecturally speaking it is kind of OK for the states of a screen to depend on the screen it self.

Also, a state is where we design! this is important to understand because with this technique we have a state isolation, within this framework we can edit any state of the screen safely without accidently breaking any other state.

And since these states are separated we can manipulate when these show and when they should hide using the state manager.

If the transition between a certain state to another is not valid, we should double check via the state manager and the state itself that the transition between a state and another is valid!

## State Manager

This is where we control which state is visible on the screen at any given time. these managers usually follow the architecture:

```dart
@provide
class LoginStateManager {
    // dependencies to manage the state 
  final AuthService _authService;
  final ProfileService _profileService;
    
    // State Object, not modifiable
  final PublishSubject<LoginState> _loginStateSubject =
      PublishSubject<LoginState>();

    // State Getter
  Stream<LoginState> get stateStream => _loginStateSubject.stream;
    
  LoginScreenState _screenState;

  LoginStateManager(this._authService, this._profileService) {
          _loginStateSubject.add(LoginStateInit(_screenState));
  }

  void loginUser(String phoneNumber, LoginScreenState _loginScreenState) {
    _screenState = _loginScreenState;
    _authService.verifyWithPhone(false, phoneNumber, UserRole.ROLE_CAPTAIN);
  }
//...
```

A good note to remember here is that there is a UI change and a state change.

The difference here is that a state change is when a the user ability to perform a certain task changes i.e. the user is logged in.

While a UI change is a red line visible when an error typing an email address.
