---
title: "Flutter Chatty Widget"
excerpt: "Simple Flutter widget that shows message in chat box on tap event."
sidebar:
  - title: "Language"
    text: "Dart"
  - title: "Description"
    text: "Simple Flutter widget that shows message in chat box on tap event."
---

A Flutter widget that shows message on tap event.

## Links
[Github](https://github.com/hyobbb/chatty_widget)   
[Pub.dev](https://pub.dev/packages/chatty_widget)

## Installation
In your `pubspec.yaml` root add:

```yaml
dependencies:
  chatty_widget: LATEST_VERSION_NUMBER
```

on use,


```dart
import 'package:chatty_widget/chatty_widget.dart';

ChattyWidget(
  child: Icon(Icons.add),
  text: 'This is Add Icon',
  textStyle: TextStyle(color: Colors.white),
  textBoxColor: Colors.black,
  chatBoxLocation: BoxLocation.Top,
  chatBoxWidth: 100,
  onTap: () {
    print('do something');
  },
)
```


## Demo

<img src="https://raw.githubusercontent.com/hyobbb/chatty_widget/master/chatty_widget_demo.gif" width="250" />

## Limit

Do Not use widgets like Buttons that has their own onTap callback function. It will be ignored.
Instead, just use onTap method to execute additional method while showing the chat box.