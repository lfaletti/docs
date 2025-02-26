---
title: Add login to your Flutter app
default: true
description: This tutorial demonstrates how to add user login with Auth0 to an Android or iOS Flutter application using the Auth0 Flutter SDK
budicon: 448
topics:
  - quickstarts
  - native
  - flutter
  - dart
  - ios
  - android
contentType: tutorial
useCase: quickstart
files:
  - files/build
  - files/main
  - files/profile
github:
  path: sample
---

# Add login to your Flutter app

Auth0 allows you to quickly add authentication and access user profile information in your application. This guide demonstrates how to integrate Auth0 with a Flutter application using the [Auth0 Flutter SDK](https://github.com/auth0/auth0-flutter).

:::note
The Flutter SDK currently only supports Flutter applications running on Android or iOS platforms.
:::

## Getting started

This quickstart assumes you already have a [Flutter](https://flutter.dev/) application up and running. If not, check out the [Flutter "getting started" guides](https://docs.flutter.dev/get-started/install) to get started with a simple app.

You should also be familiar with the [Flutter command line tool](https://docs.flutter.dev/reference/flutter-cli).

<%= include('_configure_urls_interactive') %>

## Install the Auth0 Flutter SDK

Add the Auth0 Flutter SDK into the project:

```shell
flutter pub add auth0_flutter
```

## Configure Android

If you are not developing for the Android platform, skip this step.

The SDK requires manifest placeholders. Auth0 uses placeholders internally to define an `intent-filter`, which captures the authentication callback URL. You must set the Auth0 tenant domain and the callback URL scheme.

Modify `app/build.gradle` to include manifest placeholders for `auth0Domain` and `auth0Scheme` inside the `defaultConfig` section:

```groovy
apply plugin: 'com.android.application'

android {
    defaultConfig {
        applicationId "com.auth0.samples"
        minSdkVersion 21
        targetSdkVersion flutter.targetSdkVersion
        // ...

        // ---> Add the next line
        manifestPlaceholders += [auth0Domain: "${account.namespace}", auth0Scheme: "https"]
        // <---
    }
}
```

- `auth0Domain`: The domain of your Auth0 tenant. Generally, you find this in the Auth0 Dashboard under your Application's **Settings** in the Domain field. If you are using a custom domain, you should set this to the value of your custom domain instead.
- `auth0Scheme`: The scheme to use. Can be a custom scheme, or `https` if you want to use [Android App Links](https://auth0.com/docs/applications/enable-android-app-links). You can read more about setting this value in the [Auth0.Android SDK README](https://github.com/auth0/Auth0.Android#a-note-about-app-deep-linking).

:::note
You do not need to declare a specific `intent-filter` for your activity because you defined the manifest placeholders with your Auth0 **Domain** and **Scheme** values. The library handles the redirection for you.
:::

Run **Sync Project with Gradle Files** inside Android Studio to apply your changes.

## Configure iOS

If you are not developing for the iOS platform, skip this step.

You need to register your bundle identifier as a custom URL scheme so the callback and logout URLs can reach your app.

In Xcode, go to the **Info** tab of your app target settings. In the **URL Types** section, select the **＋** button to add a new entry. Then enter `auth0` into the **Identifier** field and `$(PRODUCT_BUNDLE_IDENTIFIER)` into the **URL Schemes** field.

<p><img src="/media/articles/native-platforms/ios-swift/url-scheme.png" alt="Custom URL Scheme"></p>

:::note
The minimum target platform supported by the SDK is **iOS 12**.
:::

## Add login to your app

[Universal Login](https://auth0.com/docs/authenticate/login/auth0-universal-login) is the easiest way to set up authentication in your application. We recommend using it for the best experience, best security, and the fullest array of features.

Integrate Auth0 Universal Login in your Flutter app by importing the SDK and instantiating the `Auth0` class using your Auth0 domain and Client ID values. See this example, which instantiates the class inside a widget state object:

```dart
import 'package:auth0_flutter/auth0_flutter.dart';

class MainView extends StatefulWidget {
  const MainView({Key? key}) : super(key: key);

  @override
  State<MainView> createState() => _MainViewState();
}

class _MainViewState extends State<MainView> {
  Credentials? _credentials;

  late Auth0 auth0;

  @override
  void initState() {
    super.initState();
    auth0 = Auth0('${account.namespace}', '${account.clientId}');
  }

  @override
  Widget build(BuildContext context) {
    // ...
  }
}
```

Next, redirect your users to the Auth0 Universal Login page using `webAuthentication().login()`. This is a `Future` and must be awaited for you to retrieve the user's tokens. See this example of a `ElevatedButton` widget that logs the user in when clicked. Note that `_credentials` is used to determine locally within your app whether or not a user is signed in:

```dart
if (_credentials == null) {
  ElevatedButton(
    onPressed: () async {
      final credentials =
          await auth0.webAuthentication().login();

      setState(() {
        _credentials = credentials;
      });
    },
    child: const Text("Log in")
  )
}
```

**Android**: if you are using a custom scheme, pass this scheme to the login method so that the SDK can route to the login page and back again correctly:

```dart
await auth0.webAuthentication(scheme: 'YOUR CUSTOM SCHEME').login();
```

When a user logs in, they are redirected back to your application. Then, you are able to access the ID and access tokens for this user.

::::checkpoint
:::checkpoint-default
Add a button to your app that calls `webAuthentication().login()` and logs the user into your app. Verify that you are redirected to Auth0 for authentication and then back to your application.

Verify that you can get access to the tokens on the result of calling `login`.
:::

:::checkpoint-failure
If your application did not launch successfully:

- Ensure you set the Allowed Callback URLs are correct
- Verify you saved your changes after entering your URLs
- Make sure the domain and client ID values are imported correctly
- If using Android, ensure that the manifest placeholders have been set up correctly, otherwise the redirect back to your app may not work

Still having issues? Check out our [documentation](https://auth0.com/docs) or visit our [community page](https://community.auth0.com) to get more help.
:::
::::

## Add logout to your app

To log users out, redirect them to the Auth0 logout endpoint to clear their login session by calling the Auth0 Flutter SDK `webAuthentication().logout()`. [Read more about logging out of Auth0](https://auth0.com/docs/authenticate/login/logout).

See this example of an `ElevatedButton` widget that logs the user out of the app. Note that `_credentials` is set to `null`, indicating that the user is no longer signed in to your app:

```dart
ElevatedButton(
  onPressed: () async {
    await auth0.webAuthentication().logout();

    setState(() {
      _credentials = null;
    });
  },
  child: const Text("Log out"))
```

**Android**: if you are using a custom scheme, pass this scheme to the logout method so that the SDK can route back to your app correctly:

```
await auth0.webAuthentication(scheme: 'YOUR CUSTOM SCHEME').logout();
```

::::checkpoint
:::checkpoint-default
Add a button to your app that calls `webAuthentication().logout()` and logs the user out of your application. When you select it, verify that your Flutter app redirects you to the logout endpoint and back again. You should not be logged in to your application.
:::

:::checkpoint-failure
If your application did not log out successfully:

- Ensure the Allowed Logout URLs are set properly
- Verify you saved your changes after entering your URLs

Still having issues? Check out our [documentation](https://auth0.com/docs) or visit our [community page](https://community.auth0.com) to get more help.
:::
::::

## Show user profile information

The user profile automatically retrieves user profile properties for you when you call `webAuthentication().login()`. The returned object from the login step contains a `user` property with all the user profile properties, which populates by decoding the ID token.

See this example of a `View` component that displays the user profile on the screen:

```dart
import 'package:auth0_flutter/auth0_flutter.dart';
import 'package:flutter/material.dart';

class ProfileView extends StatelessWidget {
  const ProfileView({Key? key, required this.user}) : super(key: key);

  final UserProfile user;

  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: [
        if (user.name != null) Text(user.name!),
        if (user.email != null) Text(user.email!)
      ],
    );
  }
}
```

::::checkpoint
:::checkpoint-default
Log in and inspect the `user` property on the result. Verify the current user's profile information, such as `email` or `name`.
:::
:::checkpoint-failure
If your application did not return user profile information:

- Verify the access token is valid

Still having issues? Check out our [documentation](https://auth0.com/docs) or visit our [community page](https://community.auth0.com) to get more help.
:::
::::
