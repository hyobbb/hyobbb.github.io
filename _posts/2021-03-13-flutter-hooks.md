---
title: "Flutter Hooks"
search: true
categories:
- flutter
tags:
- flutter
last_modified_at: 2021-03-13T22:51:00
---

# State Management of Widget

Most common way of handling state of widget is using [```setState```](https://api.flutter.dev/flutter/widgets/State/setState.html) method of ```State``` of ```StatefulWidget```.
However, using StatefulWidget for all the time generates a lot of boiler plate code. Therefore, I normally use the package named *[flutter hooks](https://pub.dev/packages/flutter_hooks)*.
This is a package that implement [*React Hooks*](https://reactjs.org/docs/hooks-intro.html) of React Native.
I strongly recommend you to read their docs before(or whenever) using this package.

---

##About Hook

Here is the simple example of Widget that use Hook.
```dart
class SomeWidget extends HookWidget {
  @override
  Widget build(BuildContext context) {
    final number = useState(0);
    return Container(
      alignment: Alignment.center,
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Text('number : $number'),
          IconButton(
            icon: Icon(Icons.add),
            onPressed: () {
              number.value++;
            }),
          IconButton(
            icon: Icon(Icons.remove),
            onPressed: () {
              number.value--;
            })
        ],
      ),
    );
  }
}
```
Instead of using StatefulWidget, now we can use HookWidget which inherit StatelessWidget.
```dart
/// A [Widget] that can use [Hook]
///
/// It's usage is very similar to [StatelessWidget].
/// [HookWidget] do not have any life-cycle and implements
/// only a [build] method.
///
/// The difference is that it can use [Hook], which allows
/// [HookWidget] to store mutable data without implementing a [State].
abstract class HookWidget extends StatelessWidget {
  /// Initializes [key] for subclasses.
  const HookWidget({Key? key}) : super(key: key);

  @override
  _StatelessHookElement createElement() => _StatelessHookElement(this);
}

class _StatelessHookElement extends StatelessElement with HookElement {
  _StatelessHookElement(HookWidget hooks) : super(hooks);
}
```

As the description explains, it looks almost the same as StatelessWidget and HookWidget use HookElement that is *An [Element] that uses a [HookWidget] as its configuration.*
It means that to understand about hooks we should know about how flutter renders widget.
I suggest you to watch this video on **"How Flutter Renders Widget"** by Google flutter team.

[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/996ZgFRENMs/0.jpg)](https://www.youtube.com/watch?v=996ZgFRENMs)

Simply,

1. Flutter creates widgets
2. Widget creates element and elements hold its reference and handle its lifecycle. 
3. Finally, renderObject actually paint things on the screen. 

So, HookElement has List of hooks and whenever we call hook with *use* method it appends hook to its hook-list.
Later, when widget run build method again it can be used to update widget.
That is why hooks can be used to manage the state of state*less* widget.


---
##Examples


There are many existing hooks that is already implemented in the package, and it is possible to make your own custom hook if it is needed.
I will just write down several hooks that I frequently use, and how does it implemented with StatefulWidget.

###1.useState
```dart
class SomeWidget extends HookWidget {
  @override
  Widget build(BuildContext context) {
    final number = useState<int>(0);
    return Container(
      alignment: Alignment.center,
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Text('number : $number'),
          IconButton(
              icon: Icon(Icons.add),
              onPressed: () {
                number.value++;
              }),
          IconButton(
              icon: Icon(Icons.remove),
              onPressed: () {
                number.value--;
              })
        ],
      ),
    );
  }
}
```
useState uses ValueNotifier so whenever the value changes it rebuild the widget with updated value.
It can be implemented like below.

```dart
class SomeWidget extends StatefulWidget {
  @override
  _SomeWidgetState createState() => _SomeWidgetState();
}

class _SomeWidgetState extends State<SomeWidget> {
  int number = 0;

  @override
  Widget build(BuildContext context) {
    return Container(
      alignment: Alignment.center,
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Text('number : $number'),
          IconButton(
            icon: Icon(Icons.add),
            onPressed: () {
              setState(() {
                number++;
              });
            }),
          IconButton(
            icon: Icon(Icons.remove),
            onPressed: () {
              setState(() {
                number--;
              });
            })
        ],
      ),
    );
  }
}
```

###2.useEffect
```dart
class MyText extends HookWidget {
  final int number;

  const MyText(this.number);

  @override
  Widget build(BuildContext context) {
    String? text;
    useEffect(() {
      text = 'my number is $number';
      return () {
        print('dispose!!');
      };
    }, const []);
    return Text(text ?? 'nothing to show');
  }
}
```
useEffect is one of the most efficient function that can be used in flutter hooks in my opinion.
It can handle InitState, didUpdateWidget, and dispose of StatefulWidget.
This example can be implemented like this:

```dart
class MyText extends StatefulWidget {
  final int number;

  const MyText(this.number);

  @override
  _MyTextState createState() => _MyTextState();
}

class _MyTextState extends State<MyText> {
  String? text;

  @override
  void initState() {
    text = 'my number is ${widget.number}';
    super.initState();
  }
  
  @override
  void dispose() {
    print('dispose!!');
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Text(text ?? 'nothing to show');
  }
}
```

What if you want to update its text whenever the number changes?
In StatefulWidget you have to override didUpdateWidget. (If you don't understand why new number argument doesn't create new widget, check this [article of state lifecycle](https://hyobbb.github.io/flutter/flutter-state/). )

```dart
  @override
  void didUpdateWidget() {
    text = 'my number is ${widget.number}';
    super.didUpdateWidget();
  }
```

in useEffect you just put the "key", so that when its value changes useEffect called. You can put multiple keys and any values of those keys are changed it will call useEffect.
```dart
useEffect(() {
      text = 'my number is $number';
      return () {
        //on dispose
      };
    }, [number] /*here  put the key!!*/);
```

*Caution: if you don't specify this key and leave it null, then useEffect will be called everytime it rebuilds not just once like initState.*

###3.useController
```dart
class SomeAnimationWidget extends HookWidget {
  
  const SomeAnimationWidget();
  @override
  Widget build(BuildContext context) {
    final controller = useAnimationController(duration: Duration(seconds: 3));
    useEffect((){
      controller.forward();
      return null;
    },[]);
    return AnimatedBuilder(
      animation: controller,
      builder: (context, child) {
        return Container(
          color: Colors.red,
          width: controller.value*100,
          height: controller.value*100,
        );
      },
    );
  }
}
```
With hooks, you can use may kind of controllers easily.
I used animationController in this example, but you can use textEditingController, scrollController, or whatever controller you want.
Also, useAnimationController automatically dispose the controller so I didn't write any additional code for return function of useEffect.

This example is equal to below:

```dart
class SomeAnimationWidget extends StatefulWidget {
  @override
  _SomeAnimationWidgetState createState() => _SomeAnimationWidgetState();
}

class _SomeAnimationWidgetState extends State<SomeAnimationWidget>
    with SingleTickerProviderStateMixin {
  late AnimationController controller;

  @override
  void initState() {
    controller =
        AnimationController(duration: Duration(seconds: 3), vsync: this)
          ..forward();
    super.initState();
  }

  @override
  void dispose() {
    controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: controller,
      builder: (context, child) {
        return Container(
          color: Colors.red,
          width: controller.value * 100,
          height: controller.value * 100,
        );
      },
    );
  }
}
```
###4. HookBuilder
```dart
class MyStatelessWidget extends StatelessWidget {
  
  const MyStatelessWidget();
  @override
  Widget build(BuildContext context) {
    return Container(
      child: Center(child: HookBuilder(
        builder: (context) {
          final text = useState<String>('what');
          return GestureDetector(
              onTap: () => text.value += 'touched', child: Text(text.value));
        },
      )),
    );
  }
}
```
If you don't want to rebuild whole widget with your hooks you can use HookBuilder to use hooks only at specific location inside of your widget tree.

###Additional Tip

As HookWidget inherit StatelessWidget you can use ```const``` widget.
It is always good practice to use const widget if possible for your app to perform better. 
When your widget tree has tons of widgets to rebuild it will appreciate those const widgets.

---

##Conclusion

Flutter hooks is wonderful package to implement dynamic rendering widgets without boiler plate codes.
However, I didn't study enough on the basic principle of how it really works before, and it cost a lot when I had to build complex widgets because I made many mistakes that cause some critical issues for the lack of understanding.
Honestly, although I am writing about how to use it but still I can't say that I understand the whole mechanism.
There's no other way, just keep digging!