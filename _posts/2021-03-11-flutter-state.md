---
title: "Flutter Widget State Lifecycle"
search: true
categories:
- flutter
tags:
- flutter
last_modified_at: 2021-03-11T22:51:00
---

# About Widget State Lifecycle

In flutter, everything rendered is called *"Widget"* and widgets may have their own *"State"*. This *"State"* has lifecycle throughout its creation to disposal. So understanding widget's lifecycle is very important to make application run as expected.

For further information I recommend reading flutter API document about [State](https://api.flutter.dev/flutter/widgets/State-class.html) class (and Widget class). Also, if you search *"flutter widget lifecycle"* there's plenty of information. So, rather than repeating about their theorical meanings I would like to make un example of how this lifecycle actually applied.

---
Let's say that there's MyApp which has its child widget that is constructed with it's state parameter 'number'.

```dart
class MyApp extends StatefulWidget {  
  @override  
  _MyAppState createState() => _MyAppState();  
}  
  
class _MyAppState extends State<MyApp> {  
  int number = 0;  
  
  @override  
  Widget build(BuildContext context) {  
    debugPrint('App build $number');
    return MaterialApp(  
      home: Scaffold(  
        body: SomeWidget(number),  
		floatingActionButton: FloatingActionButton(  
          onPressed: () {  
            setState(() {  
              number++;  
			});  
		  },  
		  child: Icon(Icons.add),  
		  ),  
		),  
	  );  
  }  
}
```

if child widget extends stateless widget like below:

```dart
class SomeWidget extends StatelessWidget {  
  final int number;  
  
  SomeWidget(this.number, {Key? key}) : super(key: key) {  
    debugPrint('Make Widget with $number');  
  }  
  
  @override  
  Widget build(BuildContext context) {  
    return Container(  
      alignment: Alignment.center,  
	  child: Column(  
        mainAxisAlignment: MainAxisAlignment.center,  
	    children: [  
          Text('parent number : $number'),  
		],  
	  ),  
	);  
  }  
}
```

Then it will just synchronized as its parents state changes. There's no surprise.

```
flutter: App build 0
flutter: Make Widget with 0
flutter: App build 1
flutter: Make Widget with 1
flutter: App build 2
flutter: Make Widget with 2
```

Then, what if the child widget is StatefulWidget and it has its own number variable in its state?

```dart
class SomeWidget extends StatefulWidget {
  final int parentNumber;

  SomeWidget(this.parentNumber, {Key? key}) : super(key: key) {
    debugPrint('Make Widget with $parentNumber');
  }

  @override
  _SomeWidgetState createState() {
    debugPrint('createState');
    return _SomeWidgetState();
  }
}

class _SomeWidgetState extends State<SomeWidget> {
  int childNumber = 0;

  @override
  void initState() {
    childNumber = widget.parentNumber;
    debugPrint('initState');
    super.initState();
  }

  @override
  void didUpdateWidget(covariant SomeWidget oldWidget) {
    debugPrint('didUpdateWidget');
    super.didUpdateWidget(oldWidget);
  }

  @override
  void dispose() {
    debugPrint('dispose');
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    debugPrint('parent number : ${widget.parentNumber}');
    debugPrint('child number : ${childNumber}');
    return Container(
      alignment: Alignment.center,
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Text('parent number : ${widget.parentNumber}'),
          Text('child number : $childNumber'),
          IconButton(
              icon: Icon(Icons.add),
              onPressed: () {
                setState(() {
                  childNumber++;
                });
              }),
          IconButton(
              icon: Icon(Icons.remove),
              onPressed: () {
                setState(() {
                  childNumber--;
                });
              })
        ],
      ),
    );
  }
}
```

I put print method from constructor to build method to trace how this widget fires its method until renders.
On App start it just follow normal sequence.

```
flutter: App build 0
flutter: Make Widget with 0
flutter: didUpdateWidget
flutter: parent number : 0
flutter: child number : 0
```

modify child number.

```
flutter: parent number : 0
flutter: child number : 1
flutter: parent number : 0
flutter: child number : 2
```

works fine.

How about modifying parent number? Child widget takes parentNumber as arguments of its constructor.
Should it recreate its state?

```
flutter: App build 1
flutter: Make Widget with 1
flutter: didUpdateWidget
flutter: parent number : 1
flutter: child number : 2
```

The answer is NO. It rebuilds, but it doesn't create it's state. Instead, it calls *"didUpdateWidget"* method before build method
and maintain its state. That makes clear that in flutter widget tree, rebuild parent widget doesn't mean rebuild every children below it.
Instead, it calls didUpdateWidget method to check whether there's change between the old widget and current widget.

If I change didUpdateWidget method like below:

```dart
 @override
  void didUpdateWidget(covariant SomeWidget oldWidget) {
    debugPrint('didUpdateWidget');
    childNumber = widget.parentNumber;
    super.didUpdateWidget(oldWidget);
  }
```

then it syncrhonize its childNumber with its parentNumber.

``` 
flutter: App build 2
flutter: Make Widget with 2
flutter: didUpdateWidget
flutter: parent number : 2
flutter: child number : 2

flutter: App build 3
flutter: Make Widget with 3
flutter: didUpdateWidget
flutter: parent number : 3
flutter: child number : 3
```

Then, what if you want to re-create the state of child widget when the parent widget rebuilds?
According to the definition of didUAPI document,
###*During this time, a parent widget might rebuild and request that this location in the tree update to display a new widget with the same runtimeType and Widget.key. When this happens, the framework will update the widget property to refer to the new widget and then call the didUpdateWidget method with the previous widget as an argument*

So, there's key and runtimeType.

First, Let's make Key variable to pass to child widget.
```dart 
class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  int number = 0;
  Key? key;

  @override
  Widget build(BuildContext context) {
    debugPrint('App build $number');
    return MaterialApp(
      home: Scaffold(
        body: SomeWidget(number, key: key), //pass it!!
        floatingActionButton: FloatingActionButton(
          onPressed: () {
            setState(() {
              number++;
              key = ObjectKey(number); //creates new key
            });
          },
          child: Icon(Icons.add),
        ),
      ),
    );
  }
}
```

Every time its number changes it creates new ObjectKey and pass it to child widget.

``` 
flutter: App build 1
flutter: Make Widget with 1
flutter: createState
flutter: initState
flutter: parent number : 1
flutter: child number : 1
flutter: dispose

flutter: App build 2
flutter: Make Widget with 2
flutter: createState
flutter: initState
flutter: parent number : 2
flutter: child number : 2
flutter: dispose
```
As you see, child widget dispose and recreate from the beginning.

Secondly, Although it is more obvious, let's just create another childWidget all the same but with different type and modify MyApp Widget.

```dart
class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  int number = 0;

  @override
  Widget build(BuildContext context) {
    debugPrint('App build $number');
    return MaterialApp(
      home: Scaffold(
        body: (number.isEven) ? SomeWidget(number) : AnotherWidget(number), //change type!
        floatingActionButton: FloatingActionButton(
          onPressed: () {
            setState(() {
              number++;
            });
          },
          child: Icon(Icons.add),
        ),
      ),
    );
  }
}

class AnotherWidget extends StatefulWidget {
  final int parentNumber;

  AnotherWidget(this.parentNumber, {Key? key}) : super(key: key) {
    debugPrint('Make Another Widget with $parentNumber');
  }

  @override
  _AnotherWidget createState() {
    debugPrint('createState');
    return _AnotherWidget();
  }
}

class _AnotherWidget extends State<AnotherWidget> {
  int childNumber = 0;

  @override
  void initState() {
    childNumber = widget.parentNumber;
    debugPrint('initState');
    super.initState();
  }

  @override
  void didUpdateWidget(covariant AnotherWidget oldWidget) {
    debugPrint('didUpdateWidget');
    childNumber = widget.parentNumber;
    super.didUpdateWidget(oldWidget);
  }

  @override
  void dispose() {
    debugPrint('dispose');
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    debugPrint('parent number : ${widget.parentNumber}');
    debugPrint('child number : ${childNumber}');
    return Container(
      alignment: Alignment.center,
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Text('parent number : ${widget.parentNumber}'),
          Text('child number : $childNumber'),
          IconButton(
              icon: Icon(Icons.add),
              onPressed: () {
                setState(() {
                  childNumber++;
                });
              }),
          IconButton(
              icon: Icon(Icons.remove),
              onPressed: () {
                setState(() {
                  childNumber--;
                });
              })
        ],
      ),
    );
  }
}
```

Yes, it creates child widget from initState to build method.
``` 
flutter: App build 2
flutter: Make Widget with 2
flutter: createState
flutter: initState
flutter: parent number : 2
flutter: child number : 2
flutter: dispose

flutter: App build 3
flutter: Make Another Widget with 3
flutter: createState
flutter: initState
flutter: parent number : 3
flutter: child number : 3
flutter: dispose
```

---

##Conclusion

Rebuilding parent widget does NOT mean rebuilding whole children widgets of its widget tree.
In case of managing state of child widget according to the change of its parent widget we should know when exactly those methods are calling.
I've struggled a lot while using object that should be handled on dispose(e.g, Stream, AnimationController etc...).
