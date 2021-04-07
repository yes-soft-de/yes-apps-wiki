# Adding Modules

This is the how, If you wanted to know the why please refer to [architecture](./architecture.md).

Please follow the [Naming Conventions](./naming-conventions) when creating the classes for any new module.

## Creating a Service Module

1. Create a new folder named `module_<module_name>`
2. Inside create the folders the corresponds to the thing you want to achieve from this module. For more info please refer to [the architecture](./architecture.md).
3. Done!, now import it and use it in the project.

## Creating a UI Module

When we want to use a UI module we follow these steps.

1. Create a new folder named `module_<module_name>`
2. Inside create the folders the corresponds to the thing you want to achieve from this module. For more info please refer to [the architecture](./architecture.md).
3. create the Module Class (This is the navigation route component) and import your pages classes here. Note that this class **Must** extend the `YesModule` class.
4. create a routes map inside the overridden method of `getRoutes()`
5. Import the module into the app component.
6. add the route map into the `fullRoutesMap` map object.
7. When navigating into this Module, use the static routes from the `<module_name>_routes.dart` class.

### Example of a Module Class

```dart

@provide
@singleton
class OrdersModule extends YesModule {
  final OwnerOrdersScreen _ordersScreen;
  final NewOrderScreen _newOrderScreen;
  final OrderStatusScreen _orderStatus;
  final CaptainOrdersScreen _captainOrdersScreen;
  final UpdateScreen _updateScreen;
  final TermsScreen _termsScreen;
  OrdersModule(this._newOrderScreen, this._orderStatus, this._ordersScreen,
      this._captainOrdersScreen, this._updateScreen,this._termsScreen) {
    YesModule.RoutesMap.addAll(getRoutes());
  }

  Map<String, WidgetBuilder> getRoutes() {
    return {
      OrdersRoutes.NEW_ORDER_SCREEN: (context) => _newOrderScreen,
      OrdersRoutes.OWNER_ORDERS_SCREEN: (context) => _ordersScreen,
      OrdersRoutes.ORDER_STATUS_SCREEN: (context) => _orderStatus,
      OrdersRoutes.CAPTAIN_ORDERS_SCREEN: (context) => _captainOrdersScreen,
      OrdersRoutes.UPDATE_SCREEN: (context) => _updateScreen,
      OrdersRoutes.TERMS_SCREEN: (context) => _termsScreen
    };
  }
}
```

And we import that in the `main.dart` using after injecting it in the constructor using:

```dart
fullRoutesList.addAll(widget._ordersModule.getRoutes());
```

And that's all, now we have a UI navigated module.
