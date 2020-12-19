---
title: Flutter Environment Variable (.env) Tutorial
date: '2020-12-19 22:49:01'
categories:
- tech
layout: post
comments: true
---

![Photo by Konstantin Dyadun](/assets/photo-project-ideas_sunday-morning.jpg)

### What is environment variable?

An environment variable is a dynamic-named value that can affect the way running processes will behave on a computer. They are part of the environment in which a process runs. For example, a running process can query the value of the TEMP environment variable to discover a suitable location to store temporary files, or the HOME or USERPROFILE variable to find the directory structure owned by the user running the process. ([source](https://en.wikipedia.org/wiki/Environment_variable))

As a backend engineer, **environment variable** is very common and very easy to use. You just put that `.env` file in your root directory for most cases. But when it comes to flutter, i found it difficult to do that. The problem is, our code is an application that will be installed on different devices so it clears that environment variable is not the thing . Anyway, **environment variable** terminology is not applicable in mobile app, the correct terminology for that is **built in variable** in my opinion. Because we need to include those variable in build time so our app can access it without opening any `.env` file in different devices.

That's the story, until i found [this article about dart define](https://dartcode.org/docs/using-dart-define-in-flutter/). So i try to make an experiment about that, so we can have like this **environment variable** for flutter. Oh, i try this in linux. But i'm pretty sure you can have this alternative in you OS too!

### Steps
#### Prerequisites

In order to follow this tutorial, you should have this CLI listed bellow.

1. `cat` cat is a standard Unix utility that reads files sequentially, writing them to standard output. The name is derived from its function to concatenate files.
2. `grep` grep command is used to search text. It searches the given file for lines containing a match to the given strings or words.
3. `perl` https://www.perl.org/
4. `xargs` xargs (short for "eXtended ARGuments") is a command on Unix and most Unix-like operating systems used to build and execute commands from standard input. It converts input from standard input into arguments to a command.
5. `make` make is a build automation tool that automatically builds executable programs and libraries from source code by reading files called Makefiles which specify how to derive the target program. Though integrated development environments and language-specific compiler features can also be used to manage a build process, Make remains widely used, especially in Unix and Unix-like operating systems.

#### Create a Makefile

The first step is to create `Makefile` in your flutter root directory.

```make
ENV = $(shell cat .env | grep '^[A-Z]' | perl -ne 'print "--dart-define=$$_"' | xargs -d$$$("\n"))

run:
	flutter run $(ENV)
```

#### Create .env file

Now, create the .env file in your flutter root directory

```.env
APPLICATION_NAME=AWESOME APP
APPLICATION_TITLE=AWESOME TITLE
SERVER_LOCATION=http://192.168.0.1
SERVER_CLIENT_UID=some-client-uid
SERVER_CLIENT_SECRET=super-secret-key
```

#### Fetch those variable in your flutter application

We need to fetch those variables in our application

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    const String applicationName = String.fromEnvironment("APPLICATION_NAME");
    const String applicationTitle = String.fromEnvironment("APPLICATION_TITLE");
    const String serverLocation = String.fromEnvironment("SERVER_LOCATION");
    const String clientUID = String.fromEnvironment("SERVER_CLIENT_UID");
    const String clientSecret = String.fromEnvironment("SERVER_CLIENT_SECRET");

    print("APPLICATION NAME: " + applicationName);
    print("APPLICATION TITLE: " + applicationTitle);
    print("SERVER LOCATION: " + serverLocation);
    print("SERVER CLIENT UID: " + clientUID);
    print("SERVER CLIENT SECRET: " + clientSecret);

    return MaterialApp(
      title: applicationName,
      theme: ThemeData(
        // This is the theme of your application.
        //
        // Try running your application with "flutter run". You'll see the
        // application has a blue toolbar. Then, without quitting the app, try
        // changing the primarySwatch below to Colors.green and then invoke
        // "hot reload" (press "r" in the console where you ran "flutter run",
        // or simply save your changes to "hot reload" in a Flutter IDE).
        // Notice that the counter didn't reset back to zero; the application
        // is not restarted.
        primarySwatch: Colors.blue,
        // This makes the visual density adapt to the platform that you run
        // the app on. For desktop platforms, the controls will be smaller and
        // closer together (more dense) than on mobile platforms.
        visualDensity: VisualDensity.adaptivePlatformDensity,
      ),
      home: MyHomePage(
        title: applicationTitle,
        clientSecret: clientSecret,
        clientUID: clientUID,
        serverLocation: serverLocation,
      ),
    );
  }
}

class MyHomePage extends StatefulWidget {
  MyHomePage({
    Key key,
    @required this.title,
    @required this.serverLocation,
    @required this.clientUID,
    @required this.clientSecret,
  }) : super(key: key);

  // This widget is the home page of your application. It is stateful, meaning
  // that it has a State object (defined below) that contains fields that affect
  // how it looks.

  // This class is the configuration for the state. It holds the values (in this
  // case the title) provided by the parent (in this case the App widget) and
  // used by the build method of the State. Fields in a Widget subclass are
  // always marked "final".

  final String title;
  final String serverLocation;
  final String clientUID;
  final String clientSecret;

  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
    // This method is rerun every time setState is called, for instance as done
    // by the _incrementCounter method above.
    //
    // The Flutter framework has been optimized to make rerunning build methods
    // fast, so that you can just rebuild anything that needs updating rather
    // than having to individually change instances of widgets.
    return Scaffold(
      appBar: AppBar(
        // Here we take the value from the MyHomePage object that was created by
        // the App.build method, and use it to set our appbar title.
        title: Text(widget.title),
      ),
      body: Center(
        // Center is a layout widget. It takes a single child and positions it
        // in the middle of the parent.
        child: Column(
          // Column is also a layout widget. It takes a list of children and
          // arranges them vertically. By default, it sizes itself to fit its
          // children horizontally, and tries to be as tall as its parent.
          //
          // Invoke "debug painting" (press "p" in the console, choose the
          // "Toggle Debug Paint" action from the Flutter Inspector in Android
          // Studio, or the "Toggle Debug Paint" command in Visual Studio Code)
          // to see the wireframe for each widget.
          //
          // Column has various properties to control how it sizes itself and
          // how it positions its children. Here we use mainAxisAlignment to
          // center the children vertically; the main axis here is the vertical
          // axis because Columns are vertical (the cross axis would be
          // horizontal).
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text(
              'CLIENT UID: ' + widget.clientUID,
            ),
            Text(
              'CLIENT SECRET: ' + widget.clientSecret,
            ),
            Text(
              'SERVER LOCATION: ' + widget.serverLocation,
            ),
          ],
        ),
      ),
    );
  }
}
```

### Testing

Now we should run our application using `make run` instead of `flutter run` in order to load all of the environment variables we define in .env

```bash
make run
```

![Example](/assets/Screenshot%20from%202020-12-19%2022-47-31.png)

Just like that! We could do **environment variables like** implementation in flutter.

### Links

Example: [https://github.com/coronatorid/coronator](https://github.com/coronatorid/coronator)

Subscribe to my youtube channel: [https://www.youtube.com/channel/UC7GkDLtOSGMkr2HEVJ1rJPQ](https://www.youtube.com/channel/UC7GkDLtOSGMkr2HEVJ1rJPQ)
