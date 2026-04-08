[![pub package](https://img.shields.io/pub/v/get.svg?label=get&color=blue)](https://pub.dev/packages/get)
![building](https://github.com/jonataslaw/get/workflows/build/badge.svg)

# GetX -- State Management & Dependency Injection

A high-performance, lightweight state management and dependency injection solution for Flutter. This package provides reactive and simple state managers, a powerful DI system, and controller lifecycle management -- without any dependency on navigation, HTTP, or other unrelated concerns.

> **Migration note:** This version removes navigation (`GetMaterialApp`, `Get.to`, named routes), HTTP (`GetConnect`), animations, internationalization, `GetPlatform`, `GetResponsiveView`, context extensions, and general utilities. Use Flutter's built-in `MaterialApp` and `Navigator` for routing. Controllers are disposed via `Get.delete()`, widget unmount (`autoRemove`), or `permanent: true`.

- [Installing](#installing)
- [Quick Start](#quick-start)
- [State Management](#state-management)
  - [Reactive State Manager](#reactive-state-manager)
  - [More details about state management](#more-details-about-state-management)
- [Dependency Management](#dependency-management)
  - [More details about dependency management](#more-details-about-dependency-management)
- [Reactive Variables (.obs)](#reactive-variables-obs)
- [StateMixin](#statemixin)
- [ObxValue](#obxvalue)
- [Useful Widgets](#useful-widgets)
  - [GetView](#getview)
  - [GetWidget](#getwidget)
  - [GetxService](#getxservice)
- [Testing](#testing)
- [Community](#community)

# Installing

Add Get to your pubspec.yaml file:

```yaml
dependencies:
  get:
```

Import get in files that it will be used:

```dart
import 'package:get/get.dart';
```

# Quick Start

Create a controller with reactive variables and use `Obx` to rebuild the UI automatically when they change.

- Step 1:
  Create your business logic class and place all variables, methods and controllers inside it.
  You can make any variable observable using a simple ".obs".

```dart
class Controller extends GetxController {
  var count = 0.obs;
  increment() => count++;
}
```

- Step 2:
  Create your View using StatelessWidget. Use `Get.put()` to register the controller, and `Obx` to reactively rebuild when the value changes.

```dart
class Home extends StatelessWidget {
  const Home({super.key});

  @override
  Widget build(BuildContext context) {
    final Controller c = Get.put(Controller());

    return Scaffold(
      appBar: AppBar(title: Obx(() => Text("Clicks: ${c.count}"))),
      body: Center(child: Text("Press the button")),
      floatingActionButton: FloatingActionButton(
        onPressed: c.increment,
        child: Icon(Icons.add),
      ),
    );
  }
}
```

- Step 3:
  Use a standard `MaterialApp` -- no special root widget required.

```dart
void main() => runApp(MaterialApp(home: Home()));
```

That's it. No special app widget, no context tricks -- just a controller and a reactive widget.

# State Management

Get has two different state managers: the simple state manager (GetBuilder) and the reactive state manager (GetX/Obx).

## Reactive State Manager

Reactive programming can alienate many people because it is said to be complicated. GetX turns reactive programming into something quite simple:

- You won't need to create StreamControllers.
- You won't need to create a StreamBuilder for each variable.
- You will not need to create a class for each state.
- You will not need to create a get for an initial value.
- You will not need to use code generators.

Reactive programming with Get is as easy as using setState.

Let's imagine that you have a name variable and want that every time you change it, all widgets that use it are automatically changed.

This is your count variable:

```dart
var name = 'Jonatas Borges';
```

To make it observable, you just need to add ".obs" to the end of it:

```dart
var name = 'Jonatas Borges'.obs;
```

And in the UI, when you want to show that value and update the screen whenever the values changes, simply do this:

```dart
Obx(() => Text("${controller.name}"));
```

That's all. It's _that_ simple.

### More details about state management

**See a more in-depth explanation of state management [here](./documentation/en_US/state_management.md). There you will see more examples and also the difference between the simple state manager and the reactive state manager.**

## Dependency Management

Get has a simple and powerful dependency manager that allows you to retrieve the same class as your Bloc or Controller with just 1 line of code, no Provider context, no inheritedWidget:

```dart
Controller controller = Get.put(Controller()); // Rather Controller controller = Controller();
```

Instead of instantiating your class within the class you are using, you are instantiating it within the Get instance, which will make it available throughout your App.
So you can use your controller (or class Bloc) normally.

**Tip:** Get dependency management is decoupled from other parts of the package. If your app is already using a state manager (any one, it doesn't matter), you don't need to rewrite it all -- you can use this dependency injection with no problems at all.

```dart
controller.fetchApi();
```

Imagine that you need data that was left behind in your controller. You just need to ask Get to "find" your controller, you don't need any additional dependencies:

```dart
Controller controller = Get.find();
// Get will find your controller and deliver it to you.
// You can have 1 million controllers instantiated, Get will always give you the right controller.
```

And then you will be able to recover your controller data:

```dart
Text(controller.textFromApi);
```

### More details about dependency management

**See a more in-depth explanation of dependency management [here](./documentation/en_US/dependency_management.md)**

# Reactive Variables (.obs)

`.obs`ervables (also known as _Rx_ Types) have a wide variety of internal methods and operators.

> Is very common to _believe_ that a property with `.obs` **IS** the actual value... but make no mistake!
> We avoid the Type declaration of the variable, because Dart's compiler is smart enough, and the code
> looks cleaner, but:

```dart
var message = 'Hello world'.obs;
print( 'Message "$message" has Type ${message.runtimeType}');
```

Even if `message` _prints_ the actual String value, the Type is **RxString**!

So, you can't do `message.substring( 0, 4 )`.
You have to access the real `value` inside the _observable_:
The most "used way" is `.value`, but, did you know that you can also use...

```dart
final name = 'GetX'.obs;
// only "updates" the stream, if the value is different from the current one.
name.value = 'Hey';

// All Rx properties are "callable" and returns the new value.
// but this approach does not accepts `null`, the UI will not rebuild.
name('Hello');

// is like a getter, prints 'Hello'.
name() ;

/// numbers:

final count = 0.obs;

// You can use all non mutable operations from num primitives!
count + 1;

// Watch out! this is only valid if `count` is not final, but var
count += 1;

// You can also compare against values:
count > 2;

/// booleans:

final flag = false.obs;

// switches the value between true/false
flag.toggle();


/// all types:

// Sets the `value` to null.
flag.nil();

// All toString(), toJson() operations are passed down to the `value`
print( count ); // calls `toString()` inside  for RxInt

final abc = [0,1,2].obs;
// Converts the value to a json Array, prints RxList
// Json is supported by all Rx types!
print('json: ${jsonEncode(abc)}, type: ${abc.runtimeType}');

// RxMap, RxList and RxSet are special Rx types, that extends their native types.
// but you can work with a List as a regular list, although is reactive!
abc.add(12); // pushes 12 to the list, and UPDATES the stream.
abc[3]; // like Lists, reads the index 3.


// equality works with the Rx and the value, but hashCode is always taken from the value
final number = 12.obs;
print( number == 12 ); // prints > true

/// Custom Rx Models:

// toJson(), toString() are deferred to the child, so you can implement override on them, and print() the observable directly.

class User {
    String name, last;
    int age;
    User({this.name, this.last, this.age});

    @override
    String toString() => '$name $last, $age years old';
}

final user = User(name: 'John', last: 'Doe', age: 33).obs;

// `user` is "reactive", but the properties inside ARE NOT!
// So, if we change some variable inside of it...
user.value.name = 'Roi';
// The widget will not rebuild!,
// `Rx` don't have any clue when you change something inside user.
// So, for custom classes, we need to manually "notify" the change.
user.refresh();

// or we can use the `update()` method!
user.update((value){
  value.name='Roi';
});

print( user );
```

# StateMixin

Another way to handle your `UI` state is use the `StateMixin<T>` .
To implement it, use the `with` to add the `StateMixin<T>`
to your controller which allows a T model.

```dart
class Controller extends GetxController with StateMixin<User> {}
```

The `change()` method changes the State whenever we want.
Just pass the data and the status in this way:

```dart
change(data, status: RxStatus.success());
```

RxStatus allows these statuses:

```dart
RxStatus.loading();
RxStatus.success();
RxStatus.empty();
RxStatus.error('message');
```

To represent it in the UI, use:

```dart
class OtherClass extends GetView<Controller> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: controller.obx(
        (state) => Text(state.name),
        // here you can put your custom loading indicator, but
        // by default would be Center(child:CircularProgressIndicator())
        onLoading: CustomLoadingIndicator(),
        onEmpty: Text('No data found'),
        // here also you can set your own error widget, but by
        // default will be an Center(child:Text(error))
        onError: (error) => Text(error),
      ),
    );
  }
}
```

# ObxValue

Similar to `Obx`, but designed for local reactive state. You pass an Rx instance and it
updates automatically:

```dart
ObxValue((data) => Switch(
        value: data.value,
        onChanged: data, // Rx has a _callable_ function! You could use (flag) => data.value = flag,
    ),
    false.obs,
),
```

# Useful Widgets

## GetView

A `const Stateless` Widget that has a getter `controller` for a registered `Controller`, that's all.

```dart
class AwesomeController extends GetxController {
  final String title = 'My Awesome View';
}

// ALWAYS remember to pass the `Type` you used to register your controller!
class AwesomeView extends GetView<AwesomeController> {
  @override
  Widget build(BuildContext context) {
    return Container(
      padding: EdgeInsets.all(20),
      child: Text(controller.title), // just call `controller.something`
    );
  }
}
```

## GetWidget

Most people have no idea about this Widget, or totally confuse the usage of it.
The use case is very rare, but very specific: It `caches` a Controller.
Because of the _cache_, can't be a `const Stateless`.

> So, when do you need to "cache" a Controller?

If you use, another "not so common" feature of **GetX**: `Get.spawn()`.

`Get.spawn(()=>Controller())` will generate a new `Controller` each time you call
`Get.find<Controller>()`.

That's where `GetWidget` shines... as you can use it, for example,
to keep a list of Todo items. So, if the widget gets "rebuilt", it will keep the same controller instance.

## GetxService

This class is like a `GetxController`, it shares the same lifecycle ( `onInit()`, `onReady()`, `onClose()`).
But has no "logic" inside of it. It just notifies **GetX** Dependency Injection system, that this subclass
**can not** be removed from memory.

So is super useful to keep your "Services" always reachable and active with `Get.find()`. Like:
`ApiService`, `StorageService`, `CacheService`.

```dart
Future<void> main() async {
  await initServices();
  runApp(SomeApp());
}

/// Initialize services before running the app.
void initServices() async {
  print('starting services ...');
  await Get.putAsync(() => DbService().init());
  print('All services started...');
}

class DbService extends GetxService {
  Future<DbService> init() async {
    print('$runtimeType delays 2 sec');
    await Future.delayed(Duration(seconds: 2));
    print('$runtimeType ready!');
    return this;
  }
}
```

The only way to actually delete a `GetxService` is with `Get.reset()` which is like a
"Hot Reboot" of your app. So remember, if you need absolute persistence of a class instance during the
lifetime of your app, use `GetxService`.

# Testing

You can test your controllers like any other class, including their lifecycles:

```dart
class Controller extends GetxController {
  @override
  void onInit() {
    super.onInit();
    name.value = 'name2';
  }

  @override
  void onClose() {
    name.value = '';
    super.onClose();
  }

  final name = 'name1'.obs;

  void changeName() => name.value = 'name3';
}

void main() {
  test('Test the state of the reactive variable "name" across all of its lifecycles', () {
    final controller = Controller();
    expect(controller.name.value, 'name1');

    // If you are using GetX DI, you can test everything,
    // including the state of the application after each lifecycle.
    Get.put(controller); // onInit was called
    expect(controller.name.value, 'name2');

    // Test your functions
    controller.changeName();
    expect(controller.name.value, 'name3');

    // onClose was called
    Get.delete<Controller>();

    expect(controller.name.value, '');
  });
}
```

### Tips

#### Mockito or mocktail
If you need to mock your GetxController/GetxService, you should extend GetxController, and mixin it with Mock, that way:

```dart
class NotificationServiceMock extends GetxService with Mock implements NotificationService {}
```

#### Using Get.reset()
If you are testing widgets, or test groups, use Get.reset at the end of your test or in tearDown to reset all settings from your previous test.

# Community

## Community channels

GetX has a highly active and helpful community. If you have questions, or would like any assistance regarding the use of this framework, please join our community channels, your question will be answered more quickly, and it will be the most suitable place. This repository is exclusive for opening issues, and requesting resources, but feel free to be part of GetX Community.

| **Slack**                                                                                                                   | **Discord**                                                                                                                 | **Telegram**                                                                                                          |
| :-------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------- |
| [![Get on Slack](https://img.shields.io/badge/slack-join-orange.svg)](https://communityinviter.com/apps/getxworkspace/getx) | [![Discord Shield](https://img.shields.io/discord/722900883784073290.svg?logo=discord)](https://discord.com/invite/9Hpt99N) | [![Telegram](https://img.shields.io/badge/chat-on%20Telegram-blue.svg)](https://t.me/joinchat/PhdbJRmsZNpAqSLJL6bH7g) |

## How to contribute

_Want to contribute to the project? We will be proud to highlight you as one of our collaborators. Here are some points where you can contribute and make Get (and Flutter) even better._

- Adding documentation to the readme.
- Write articles or make videos teaching how to use Get.
- Offering PRs for code/tests.
- Including new functions.

Any contribution is welcome!

## Articles and videos

- [Complete GetX State Management](https://www.youtube.com/watch?v=CNpXbeI_slw) - State management video by Amateur Coder.
- [The Flutter GetX Ecosystem ~ State Management](https://medium.com/flutter-community/the-flutter-getx-ecosystem-state-management-881c7235511d) - State management by [Aachman Garg](https://github.com/imaachman).
- [The Flutter GetX Ecosystem ~ Dependency Injection](https://medium.com/flutter-community/the-flutter-getx-ecosystem-dependency-injection-8e763d0ec6b9) - Dependency Injection by [Aachman Garg](https://github.com/imaachman).
- [Flutter State Management with GetX -- Complete App](https://www.appwithflutter.com/flutter-state-management-with-getx/) - by App With Flutter.
