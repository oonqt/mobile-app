# Rebble app

A multi platform watch companion app for Pebble/RebbleOS devices

# Development

## Building the app
1. Checkout this repo
2. [Generate new Github token with `read:packages` permission](https://github.com/settings/tokens). This is required to fetch libpebblecommons from Github packages repository.
3. Create `local.properties` file in `android` folder. Write following to the file:

    ```
    GITHUB_ACTOR=<YOUR GITHUB USERNAME>
    GITHUB_TOKEN=<GENERATED TOKEN>
    ```

4. [Install flutter on your machine](https://flutter.dev/docs/get-started/install)
5. [Setup flutter in the IDE of your choice](https://flutter.dev/docs/get-started/editor)
6. Open this repo in the IDE set up in step 5

## Building mappings

To build all the mappings in this project (such as entity <> map mapping for SQL), you have to
run the following command:

`flutter pub run build_runner build --delete-conflicting-outputs`

## Building pigeons

Type safe communication between Flutter and native code is performed 
using [Pigeon](https://pub.dev/packages/pigeon). To add new communication interfaces, edit
[pigeons/pigeons.dart](pigeons/pigeons.dart) file and then re-compile interface
with the following command:

```
flutter pub run pigeon \
  --input pigeons/pigeons.dart \
  --dart_out lib/infrastructure/pigeons/pigeons.g.dart \
  --java_out ./android/app/src/main/java/io/rebble/cobble/pigeons/Pigeons.java \
  --java_package "io.rebble.cobble.pigeons"
```

# Architecture

See [Wiki](https://github.com/pebble-dev/mobile-app/wiki) for more info on app architecture.

## Using Cobble theming

App's components are styled through modified Material theme, in theory you should never specify
custom styles in your own component. If you have to, try to use colors that are defined in 
`ThemeData` (accessed by `WithCobbleTheme(context).theme`) or alternatively in 
`CobbleSchemeData` (`WithCobbleTheme(context).scheme`). Scheme is collection of colors, 
created by designer while the theme is higher-level grouping of these colours to provide meaningful 
base styles for components. If you start using Material component which isn't styled properly, 
take a look at Material theme and see if you can set styles there before setting styles directly on
component. There is limited set of text types, as defined by designer, if you need different text 
style, extends these types with `.copyWith` instead of creating your own.

## Using Navigator

We are using iOS-style tabbed navigation, where each tab has its own stack of screens. In practice
this means there might be multiple stacks (1 main stack and one each for tab) but only 1 stack is
active. In order to push page on an active stack import `CobbleNavigator` extension and then call
`context.push(SomeScreen())`. `SomeScreen` widget should also implement interface `CobbleScreen` and
use `CobbleScaffold.page` or `CobbleScaffold.tab`, which takes care of title and back button in 
navigation bar.

## Custom Cobble components

A lot of components were refactored in custom Widgets, like CobbleCard, CobbleTile, CobbleButton, etc.
and these components should serve you as building blocks upon which to build your UI. They are 
showcased in WidgetLibrary screen and in golden (aka snapshot) tests. All golden images (how widgets 
should look) are included in /test/components/goldens.

## Using localization

To use localized string, add it to all `.json` files in `/lang`, start build_runner to generate 
localized models (see [Building mappings](#building-mappings) above) and then use it as 
`tr.canBeNested.yourKey`. Generator also supports named  and positional parameters:  
`"key": "fixed value, named parameter -> {named}, positional parameter -> {}` and generates 
function instead of string. Use this function similar to string:  
`tr.canBeNested.yourKey('positional', named: 'named param')`.

App's localization is stored in /lang directory, one `.json` file for one language. Structure of 
these `.json` files is then converted to localized model with a help of `ModelGenerator`. Model
is in turn used to load and parse correct `.json` file at app's startup. Refer to 
[build.yaml](build.yaml) and [CobbleLocalizationDelegate](lib/localization/localization_delegate.dart)
for more info.