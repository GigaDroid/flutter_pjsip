# flutter_pjsip

This is a Flutter plugin for the [PJSIP](https://www.pjsip.org/) library. It
provides a Dart API for the PJSIP library, and it is implemented using Dart FFI.

## Getting started as user

You can use this plugin as a user by adding it to your `pubspec.yaml`:

```bash
flutter pub add flutter_pjsip
```

Documentation for the API will be available later. TODO: Add link to API docs.

## Getting started as hacker

This project based on Flutter
[FFI plugin](https://docs.flutter.dev/development/platform-integration/c-interop),
a specialized package that includes native code directly invoked with Dart FFI.

For simplicity we use [Melos](https://pub.dev/packages/melos) to manage
multi-package project. Melos is a Dart tool for managing Dart projects with
multiple packages, inspired by Lerna for JavaScript.

## Melos magic

### Install melos

```bash
dart pub global activate melos
```

### Prepare the project for hacking

By running the following command, melos will boostrap the project, get PJSIP source code, generate FFI bindings, and build PJSIP for all platforms:

```bash
melos prepare
```

### Install dependencies

Alternatively, you can execute prepare steps one by one. First, you need to install dependencies for all packages:

```bash
melos bs
```

Here `bs` means `bootstrap`.

### Get PJSIP source code

```bash
melos run get-pjsip
```

### Generate FFI bindings

```bash
melos run gen-ffi-bindings
```

### Build PJSIP

You can build PJSIP for all platforms at once or for a specific platform:

```bash
# All platforms:
melos run build
# iOS:
melos run build-ios
# Android:
melos run build-android
```

### Analyze all packages

```bash
melos run analyze
```

### Clean up everything

This will clean up all packages and remove all generated files:

```bash
melos clean
```

## Project structure

We forced to move actual project to `packages/flutter_pjsip` folder because of Melos limitations (it's not compatible with the latest `file` package which is used by `ffi` package). So, we have the following structure:

- `/` - root folder with `melos.yaml` file
- `/scripts` - scripts for melos
- `/packages/flutter_pjsip` - actual project
- `/packages/flutter_pjsip/example` - example project

The actual project `flutter_pjsip` project uses the following structure:

- `src`: Contains the native source code, and a CmakeFile.txt file for building
  that source code into a dynamic library.

- `lib`: Contains the Dart code that defines the API of the plugin, and which
  calls into the native code using `dart:ffi`.

- platform folders (`android`, `ios`): Contains the build files
  for building and bundling the native code library with the platform application.

- `android/libs` and `ios/Frameworks`: Contains the native libraries and
  frameworks that are bundled with the platform application.

## Building and bundling native code

The `pubspec.yaml` specifies FFI plugins as follows:

```yaml
plugin:
  platforms:
    some_platform:
      ffiPlugin: true
```

This configuration invokes the native build for the various target platforms
and bundles the binaries in Flutter applications using these FFI plugins.

This can be combined with dartPluginClass, such as when FFI is used for the
implementation of one platform in a federated plugin:

```yaml
plugin:
  implements: some_other_plugin
  platforms:
    some_platform:
      dartPluginClass: SomeClass
      ffiPlugin: true
```

A plugin can have both FFI and method channels:

```yaml
plugin:
  platforms:
    some_platform:
      pluginClass: SomeName
      ffiPlugin: true
```

The native build systems that are invoked by FFI (and method channel) plugins are:

- For Android: Gradle, which invokes the Android NDK for native builds.
  - See the documentation in android/build.gradle.
- For iOS and MacOS: Xcode, via CocoaPods.
  - See the documentation in ios/flutter_pjsip.podspec.
  - See the documentation in macos/flutter_pjsip.podspec.
- For Linux and Windows: CMake.
  - See the documentation in linux/CMakeLists.txt.
  - See the documentation in windows/CMakeLists.txt.

## Binding to native code

To use the native code, bindings in Dart are needed.
To avoid writing these by hand, they are generated from the header file
(`src/flutter_pjsip.h`) by `package:ffigen`.
Regenerate the bindings by running `flutter pub run ffigen --config ffigen.yaml`.

## Invoking native code

Very short-running native functions can be directly invoked from any isolate.

Longer-running functions should be invoked on a helper isolate to avoid
dropping frames in Flutter applications.

## Flutter help

For help getting started with Flutter, view our
[online documentation](https://flutter.dev/docs), which offers tutorials,
samples, guidance on mobile development, and a full API reference.
