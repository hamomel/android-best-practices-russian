# Передовой опыт разработки для Android

Уроки от лучших Андроид разработчиков из команды Futurice. Не нужно изобретать велосипед, просто следуйте простым правилам в руководстве. Если вы интересуетесь разработкой для IOS или Windows Phone, обратите внимание на [**iOS Good Practices**](https://github.com/futurice/ios-good-practices) и [**Windows App Development Best Practices**](https://github.com/futurice/windows-app-development-best-practices).

[![Android Arsenal](https://img.shields.io/badge/Android%20Arsenal-android--best--practices-brightgreen.svg?style=flat)](https://android-arsenal.com/details/1/1091)

## Краткое содержание

#### Используйте Gradle и рекомендованную им структуру проекта
#### Храните пароли и важные данные в файле gradle.properties
#### Не пишите свой HTTP клиент, используйте библиотеки Volley или OkHttp
#### Используйте библиотеку Jackson для парсинга JSON данных
#### Не нужно использовать Guava и большое количество библиотек чтобы не привысить лимит методов (65k)
#### Используйте фрагменты для отображения пользовательского интерфейса
#### Используйте activity только для управления фрагментами
#### XML-разметка — это код, который нужно писать аккуратно
#### Чтобы не дублировать атрибуты в XML-разметке, используйте стили
#### Используйте несколько файлов со стилями, не нужно создавать один огромный файл
#### Сделайте файл colors.xml компактным. Не дублируйте цвета, задайте основную палитру
#### То же касается и dimens.xml, задайте только основные константы
#### Избегайте слишком глубокой иерархии элементов ViewGroup
#### Не нужно обрабатывать WebView на клиентской стороне, будьте внимательны к возможным утечкам
#### Используйте Robolectric для unit-тестов, а Robotium для UI-тестов
#### Используйте Genymotion в качестве эмулятора
#### Всегда используйте ProGuard или DexGuard


----------

### Android SDK

Разместите [Android SDK](https://developer.android.com/sdk/installing/index.html?pkg=tools) в домашней дирректории или другом месте, не связанном с приложжением. Some IDEs include the SDK when installed, and may place it under the same directory as the IDE. This can be bad when you need to upgrade (or reinstall) the IDE, or when changing IDEs. Also avoid putting the SDK in another system-level directory that might need sudo permissions, if your IDE is running under your user and not under root. Некоторые IDE включают в себя SDK при установке и могут размещать его в своей дирректории. Это не совсем правильно, так как в дальнейшем вам может понадобиться обновить, переустановить или сменить IDE. Не размещайте SDK в системные папки, так как для доступа к ним может понадобиться ипользование команды sudo, если вы вошли в систему как обычный пользователь. 

### Система сборки

Используйте [Gradle](http://tools.android.com/tech-docs/new-build-system) по умолчанию. У Ant меньше возможностей и его код менее компактный. Используя Gradle, вы сможете:

- Создавать различные сборки и варианты приложения
- Создавать простые задачи в виде скрипта
- Управлять и загружать зависимости
- Настраивать хранилище ключей
- И многое другое

Плагин Gradle находится в активной разработке командой Google и уже стал основной системой сборки для Android

### Project structure

Существует два варианта структуры проекта: старый Ant + Eclipse ADT и новый Gradle + Android Studio. Лучше использовать второй вариант. Если ваш проект все еще использует старую структуру, рекомендуем портировать проект.

Старая структура:

```
old-structure
├─ assets
├─ libs
├─ res
├─ src
│  └─ com/futurice/project
├─ AndroidManifest.xml
├─ build.gradle
├─ project.properties
└─ proguard-rules.pro
```

Новая структура:

```
new-structure
├─ library-foobar
├─ app
│  ├─ libs
│  ├─ src
│  │  ├─ androidTest
│  │  │  └─ java
│  │  │     └─ com/futurice/project
│  │  └─ main
│  │     ├─ java
│  │     │  └─ com/futurice/project
│  │     ├─ res
│  │     └─ AndroidManifest.xml
│  ├─ build.gradle
│  └─ proguard-rules.pro
├─ build.gradle
└─ settings.gradle
```

Главным отличием является то, что новая структура явно разделяет 'наборы ресурсов' (`main`, `androidTest`), используя одну из концепций Gradle. Например, вы можете добавить папки 'paid' и 'free' в вашу папку `src`, в которых будет отдельный исходный код для платной и бесплатной версий приложения.

Наличие папки `app` в верхнем уровне иерархии помогает отделить ваше приложение от библиотек (например, `library-foobar`), которые в нем используются. В таком случае файл `settings.gradle` хранит проекты, на которые может ссылаться `app/build.gradle`.


### Конфигурация Gradle

**Общая структура.** Следуйте [Рекомендациям Google для Gradle for Android](http://tools.android.com/tech-docs/new-build-system/user-guide)

**Маленькие задачи.** Вместо скриптов на shell, Python, Perl, вы можете создавать задачи в Gradle. Подробне в документации [Документация Gradle](http://www.gradle.org/docs/current/userguide/userguide_single.html#N10CBF).

**Пароли.** В файле `build.gradle` вашего приложения необходимо задать `signingConfigs` для релизной сборки. Here is what you should avoid:

_Не делайте так_. Эта информаци появится в системе контроля версий.

```groovy
signingConfigs {
    release {
        storeFile file("myapp.keystore")
        storePassword "password123"
        keyAlias "thekey"
        keyPassword "password789"
    }
}
```

Вместо этого, создайте файл `gradle.properties`, который _не_ будет добавлен в систему контроля версий:

```
KEYSTORE_PASSWORD=password123
KEY_PASSWORD=password789
```

Gradle автоматически импортирует этот файл, по этому вы можете использовать его в `build.gradle`:

```groovy
signingConfigs {
    release {
        try {
            storeFile file("myapp.keystore")
            storePassword KEYSTORE_PASSWORD
            keyAlias "thekey"
            keyPassword KEY_PASSWORD
        }
        catch (ex) {
            throw new InvalidUserDataException("You should define KEYSTORE_PASSWORD and KEY_PASSWORD in gradle.properties.")
        }
    }
}
```

**Старайтесь использовать зависимости Maven вместо импортирования jar-файлов.** Если вы явно включаете jar файл в ваш проект, его версия будет неизменна, например `2.1.1`. Скачивать и обновлять зависимости это непростая задача, которую Maven с легкостью решает, что также приветствуется в сборках Android Gradle. Например:

```groovy
dependencies {
    compile 'com.squareup.okhttp:okhttp:2.2.0'
    compile 'com.squareup.okhttp:okhttp-urlconnection:2.2.0'
}
```    

**Избегайте использования динамических зависимостей Maven**
Не нужно использовать динамические версии, такие как `2.1.+` так как это может привести к нарушению стабильности сборки, а также непредсказуемым различиям между разными сборками. Использование статических версий как  `2.1.1` помогает создать более стабильную и предсказуемую среду разработки.

### Среда разработки (IDE) и текстовые редакторы

**Искользуйте любой редактор, который удобно отображает структуру проекта.** Текстовый редактор это дело вкуса, так или иначе вам нужно будет настроить его в соответствии со структурой проекта и системой сборки.

На данный момент рекомендуемая IDE - [Android Studio](https://developer.android.com/sdk/installing/studio.html), разработанная командой Google, она лучше других сочитается с Gradle, использует новую структуру проекта по умолчанию, уже в стабильной сборке и создана непосредственно для Android разработки.

Вы можете использовать [Eclipse ADT](https://developer.android.com/sdk/installing/index.html?pkg=adt), но вам придется настроить его, так как там используется старая структура проекта и система сборки Ant. Вы даже можеде использовать простые редакторы как Vim, Sublime Text, или Emacs. В этом случае вам понадобится Gradle и команда `adb`. Если у вас не получается интегрировать Gradle в Eclipse, для сборки можно использовать командную строку, или начать использовать Android Studio. Это лучший вариант, учитывая что плагин ADT устарел.

Что бы вы не использовали, главное убедиться что Gradle и структура проекта следуют официальным инструкциям, и не добавлять в систему контроля версий специфические для редактора файлы. Например, не добавляйте файл `build.xml`. Особенно внимательно следите за обновлениями файла `build.gradle` если вы меняете настройки сборки в Ant. Будьте доброжелательными по отношению к другим разработчикам и не заставляйте их использовать непривычные инструменты.

### Библиотеки

**[Jackson](http://wiki.fasterxml.com/JacksonHome)** это Java-библиотека для конвертации объектов в JSON обратно. [Gson](https://code.google.com/p/google-gson/) так же хорошее решение этой задачи, но мы обнаружили что у Jackson выше производительность так как он поддерживает альтернативные способы обработки JSON: стриминг, модель дерева памяти, и традиционную связку JSON-POJO. Но имейте в виду что Jackson больше чем GSON, по этому, в зависимости от ситуации, вам нужно будет использовать GSON чтобы не привысить лимит в 65k методов. Альтернативные библиотеки: [Json-smart](https://code.google.com/p/json-smart/) и [Boon JSON](https://github.com/RichardHightower/boon/wiki/Boon-JSON-in-five-minutes)

**Сеть, кеширование и изображения.** Существует несколько проверенных временем библиотек для создания запросов к backend-серверам, которые следует использовать вместо создания собственного клиента. Используйте [Volley](https://android.googlesource.com/platform/frameworks/volley) или [Retrofit](http://square.github.io/retrofit/). Volley также поддерживает кеширование и загрузку изображений. Если вы предпочитаете Retrofit, используйте [Picasso](http://square.github.io/picasso/) для загрузки и кеширования картинок, и [OkHttp](http://square.github.io/okhttp/) для эффективных HTTP запросов. Все три библиотеки (Retrofit, Picasso и OkHttp) созданы одной компанией, поэтому они неплохо дополняют друг друга. [OkHttp так же может быть использован в сочитании с Volley](http://stackoverflow.com/questions/24375043/how-to-implement-android-volley-with-okhttp-2-0/24951835#24951835).

**RxJava** - библиотека для Reactive Programming, другими словами, для обработки асинхронных событий. Это очень мощная и многообещающая концепция, которая сначала может смутить своей необычностью. Мы рекомендуем подумать, перед тем как использовать эту библиотеку как фундамент архитектуры вашего приложения. Существуют проекты, созданные с использованием RxJava, и вы можете обратиться за помощью в использовании RxJava к однуму из этих людей: Timo Tuominen, Olli Salonen, Andre Medeiros, Mark Voit, Antti Lammi, Vera Izrailit, Juha Ristolainen. Мы написали несколько статей на эту тему: 
[[1]](http://blog.futurice.com/tech-pick-of-the-week-rx-for-net-and-rxjava-for-android), [[2]](http://blog.futurice.com/top-7-tips-for-rxjava-on-android), [[3]](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754), [[4]](http://blog.futurice.com/android-development-has-its-own-swift).

Если у вас нет опыта работы с Rx, то начните с использования API. Или вы можете начать с обработки простых событий пользовательского интерфейса, таких как обработка кликов или ввод текста. Если вы уверены в ваших навыках использования Rx и хотите использовать его во всей архитектуре приложения, напишите Javadocs о самых сложных моментах. Помните, что у программиста, не имеющего опыта  виспользовании RxJava, могут быть большие проблемы при работе с проектом. Помогите понять ваш код и Rx.


**[Retrolambda](https://github.com/evant/gradle-retrolambda)** - Java библиотека fдля использования Lambda-выражений в Android и других платформах с JDK ниже версии 8 . Это поможет сохранить компактность и читабельность кода особенно при использовании функционального стиля, например с RxJava. Для ее использования установите JDK8, выберите его как  SDK в настройках структуры проекта в Android Studio, и задайте переменные `JAVA8_HOME` и `JAVA7_HOME`, затем в корневом файле build.gradle:

```groovy
dependencies {
    classpath 'me.tatarka:gradle-retrolambda:2.4.1'
}
```

и в файлах build.gradle для каждого модуля:

```groovy
apply plugin: 'retrolambda'

android {
    compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
}

retrolambda {
    jdk System.getenv("JAVA8_HOME")
    oldJdk System.getenv("JAVA7_HOME")
    javaVersion JavaVersion.VERSION_1_7
}
```

Android Studio offers code assist support for Java8 lambdas. If you are new to lambdas, just use the following to get started:

- Any interface with just one method is "lambda friendly" and can be folded into the more tight syntax
- If in doubt about parameters and such, write a normal anon inner class and then let Android Studio fold it into a lambda for you.

**Beware of the dex method limitation, and avoid using many libraries.** Android apps, when packaged as a dex file, have a hard limitation of 65536 referenced methods [[1]](https://medium.com/@rotxed/dex-skys-the-limit-no-65k-methods-is-28e6cb40cf71) [[2]](http://blog.persistent.info/2014/05/per-package-method-counts-for-androids.html) [[3]](http://jakewharton.com/play-services-is-a-monolith/). You will see a fatal error on compilation if you pass the limit. For that reason, use a minimal amount of libraries, and use the [dex-method-counts](https://github.com/mihaip/dex-method-counts) tool to determine which set of libraries can be used in order to stay under the limit. Especially avoid using the Guava library, since it contains over 13k methods.

### Activities and Fragments

There is no consensus among the community nor Futurice developers how to best organize Android architectures with Fragments and Activities. Square even has [a library for building architectures mostly with Views](https://github.com/square/mortar), bypassing the need for Fragments, but this still is not considered a widely recommendable practice in the community.

Because of Android API's history, you can loosely consider Fragments as UI pieces of a screen. In other words, Fragments are normally related to UI. Activities can be loosely considered to be controllers, they are specially important for their lifecycle and for managing state. However, you are likely to see variation in these roles: activities might be take UI roles ([delivering transitions between screens](https://developer.android.com/about/versions/lollipop.html)), and [fragments might be used solely as controllers](http://developer.android.com/guide/components/fragments.html#AddingWithoutUI). We suggest to sail carefully, taking informed decisions since there are drawbacks for choosing a fragments-only architecture, or activities-only, or views-only. Here are some advices on what to be careful with, but take them with a grain of salt:

- Avoid using [nested fragments](https://developer.android.com/about/versions/android-4.2.html#NestedFragments) extensively, because [matryoshka bugs](http://delyan.me/android-s-matryoshka-problem/) can occur. Use nested fragments only when it makes sense (for instance, fragments in a horizontally-sliding ViewPager inside a screen-like fragment) or if it's a well-informed decision.
- Avoid putting too much code in activities. Whenever possible, keep them as lightweight containers, existing in your application primarily for the lifecycle and other important Android-interfacing APIs. Prefer single-fragment activities instead of plain activities - put UI code into the activity's fragment. This makes it reusable in case you need to change it to reside in a tabbed layout, or in a multi-fragment tablet screen. Avoid having an activity without a corresponding fragment, unless you are making an informed decision.
- Don't abuse Android-level APIs such as heavily relying on Intent for your app's internal workings. You could affect the Android OS or other applications, creating bugs or lag. For instance, it is known that if your app uses Intents for internal communication between your packages, you might incur multi-second lag on user experience if the app was opened just after OS boot.

### Java packages architecture

Java architectures for Android applications can be roughly approximated in [Model-View-Controller](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller). In Android, [Fragment and Activity are actually controller classes](http://www.informit.com/articles/article.aspx?p=2126865). On the other hand, they are explicity part of the user interface, hence are also views.

For this reason, it is hard to classify fragments (or activities) as strictly controllers or views. It's better to let them stay in their own `fragments` package. Activities can stay on the top-level package as long as you follow the advice of the previous section. If you are planning to have more than 2 or 3 activities, then make also an `activities` package.

Otherwise, the architecture can look like a typical MVC, with a `models` package containing POJOs to be populated through the JSON parser with API responses, and a `views` package containing your custom Views, notifications, action bar views, widgets, etc. Adapters are a gray matter, living between data and views. However, they typically need to export some View via `getView()`, so you can include the `adapters` subpackage inside `views`.

Some controller classes are application-wide and close to the Android system. These can live in a `managers` package. Miscellaneous data processing classes, such as "DateUtils", stay in the `utils` package. Classes that are responsible for interacting with the backend stay in the `network` package.

All in all, ordered from the closest-to-backend to the closest-to-the-user:

```
com.futurice.project
├─ network
├─ models
├─ managers
├─ utils
├─ fragments
└─ views
   ├─ adapters
   ├─ actionbar
   ├─ widgets
   └─ notifications
```

### Resources

**Naming.** Follow the convention of prefixing the type, as in `type_foo_bar.xml`. Examples: `fragment_contact_details.xml`, `view_primary_button.xml`, `activity_main.xml`.

**Organizing layout XMLs.** If you're unsure how to format a layout XML, the following convention may help.

- One attribute per line, indented by 4 spaces
- `android:id` as the first attribute always
- `android:layout_****` attributes at the top
- `style` attribute at the bottom
- Tag closer `/>` on its own line, to facilitate ordering and adding attributes.
- Rather than hard coding `android:text`, consider using [Designtime attributes](http://tools.android.com/tips/layout-designtime-attributes) available for Android Studio.

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    >

    <TextView
        android:id="@+id/name"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentRight="true"
        android:text="@string/name"
        style="@style/FancyText"
        />

    <include layout="@layout/reusable_part" />

</LinearLayout>
```

As a rule of thumb, attributes `android:layout_****` should be defined in the layout XML, while other attributes `android:****` should stay in a style XML. This rule has exceptions, but in general works fine. The idea is to keep only layout (positioning, margin, sizing) and content attributes in the layout files, while keeping all appearance details (colors, padding, font) in styles files.

The exceptions are:

- `android:id` should obviously be in the layout files
- `android:orientation` for a `LinearLayout` normally makes more sense in layout files
- `android:text` should be in layout files because it defines content
- Sometimes it will make sense to make a generic style defining `android:layout_width` and `android:layout_height` but by default these should appear in the layout files

**Use styles.** Almost every project needs to properly use styles, because it is very common to have a repeated appearance for a view. At least you should have a common style for most text content in the application, for example:

```xml
<style name="ContentText">
    <item name="android:textSize">@dimen/font_normal</item>
    <item name="android:textColor">@color/basic_black</item>
</style>
```

Applied to TextViews:

```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/price"
    style="@style/ContentText"
    />
```

You probably will need to do the same for buttons, but don't stop there yet. Go beyond and move a group of related and repeated `android:****` attributes to a common style.

**Split a large style file into other files.** You don't need to have a single `styles.xml` file. Android SDK supports other files out of the box, there is nothing magical about the name `styles`, what matters are the XML tags `<style>` inside the file. Hence you can have files `styles.xml`, `styles_home.xml`, `styles_item_details.xml`, `styles_forms.xml`. Unlike resource directory names which carry some meaning for the build system, filenames in `res/values` can be arbitrary.

**`colors.xml` is a color palette.** There should be nothing else in your `colors.xml` than just a mapping from a color name to an RGBA value. Do not use it to define RGBA values for different types of buttons.

*Don't do this:*

```xml
<resources>
    <color name="button_foreground">#FFFFFF</color>
    <color name="button_background">#2A91BD</color>
    <color name="comment_background_inactive">#5F5F5F</color>
    <color name="comment_background_active">#939393</color>
    <color name="comment_foreground">#FFFFFF</color>
    <color name="comment_foreground_important">#FF9D2F</color>
    ...
    <color name="comment_shadow">#323232</color>
```

You can easily start repeating RGBA values in this format, and that makes it complicated to change a basic color if needed. Also, those definitions are related to some context, like "button" or "comment", and should live in a button style, not in `colors.xml`.

Instead, do this:

```xml
<resources>

    <!-- grayscale -->
    <color name="white"     >#FFFFFF</color>
    <color name="gray_light">#DBDBDB</color>
    <color name="gray"      >#939393</color>
    <color name="gray_dark" >#5F5F5F</color>
    <color name="black"     >#323232</color>

    <!-- basic colors -->
    <color name="green">#27D34D</color>
    <color name="blue">#2A91BD</color>
    <color name="orange">#FF9D2F</color>
    <color name="red">#FF432F</color>

</resources>
```

Ask for this palette from the designer of the application. The names do not need to be color names as "green", "blue", etc. Names such as "brand_primary", "brand_secondary", "brand_negative" are totally acceptable as well. Formatting colors as such will make it easy to change or refactor colors, and also will make it explicit how many different colors are being used. Normally for a aesthetic UI, it is important to reduce the variety of colors being used.

**Treat dimens.xml like colors.xml.** You should also define a "palette" of typical spacing and font sizes, for basically the same purposes as for colors. A good example of a dimens file:

```xml
<resources>

    <!-- font sizes -->
    <dimen name="font_larger">22sp</dimen>
    <dimen name="font_large">18sp</dimen>
    <dimen name="font_normal">15sp</dimen>
    <dimen name="font_small">12sp</dimen>

    <!-- typical spacing between two views -->
    <dimen name="spacing_huge">40dp</dimen>
    <dimen name="spacing_large">24dp</dimen>
    <dimen name="spacing_normal">14dp</dimen>
    <dimen name="spacing_small">10dp</dimen>
    <dimen name="spacing_tiny">4dp</dimen>

    <!-- typical sizes of views -->
    <dimen name="button_height_tall">60dp</dimen>
    <dimen name="button_height_normal">40dp</dimen>
    <dimen name="button_height_short">32dp</dimen>

</resources>
```

You should use the `spacing_****` dimensions for layouting, in margins and paddings, instead of hard-coded values, much like strings are normally treated. This will give a consistent look-and-feel, while making it easier to organize and change styles and layouts.

**strings.xml**

Name your strings with keys that resemble namespaces, and don't be afraid of repeating a value for two or more keys. Languages are complex, so namespaces are necessary to bring context and break ambiguity.

**Bad**
```xml
<string name="network_error">Network error</string>
<string name="call_failed">Call failed</string>
<string name="map_failed">Map loading failed</string>
```

**Good**
```xml
<string name="error.message.network">Network error</string>
<string name="error.message.call">Call failed</string>
<string name="error.message.map">Map loading failed</string>
```

Don't write string values in all uppercase. Stick to normal text conventions (e.g., capitalize first character). If you need to display the string in all caps, then do that using for instance the attribute [`textAllCaps`](http://developer.android.com/reference/android/widget/TextView.html#attr_android:textAllCaps) on a TextView.

**Bad**
```xml
<string name="error.message.call">CALL FAILED</string>
```

**Good**
```xml
<string name="error.message.call">Call failed</string>
```

**Avoid a deep hierarchy of views.** Sometimes you might be tempted to just add yet another LinearLayout, to be able to accomplish an arrangement of views. This kind of situation may occur:

```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    >

    <RelativeLayout
        ...
        >

        <LinearLayout
            ...
            >

            <LinearLayout
                ...
                >

                <LinearLayout
                    ...
                    >
                </LinearLayout>

            </LinearLayout>

        </LinearLayout>

    </RelativeLayout>

</LinearLayout>
```

Even if you don't witness this explicitly in a layout file, it might end up happening if you are inflating (in Java) views into other views.

A couple of problems may occur. You might experience performance problems, because there are is a complex UI tree that the processor needs to handle. Another more serious issue is a possibility of [StackOverflowError](http://stackoverflow.com/questions/2762924/java-lang-stackoverflow-error-suspected-too-many-views).

Therefore, try to keep your views hierarchy as flat as possible: learn how to use [RelativeLayout](https://developer.android.com/guide/topics/ui/layout/relative.html), how to [optimize your layouts](http://developer.android.com/training/improving-layouts/optimizing-layout.html) and to use the [`<merge>` tag](http://stackoverflow.com/questions/8834898/what-is-the-purpose-of-androids-merge-tag-in-xml-layouts).

**Beware of problems related to WebViews.** When you must display a web page, for instance for a news article, avoid doing client-side processing to clean the HTML, rather ask for a "*pure*" HTML from the backend programmers. [WebViews can also leak memory](http://stackoverflow.com/questions/3130654/memory-leak-in-webview) when they keep a reference to their Activity, instead of being bound to the ApplicationContext. Avoid using a WebView for simple texts or buttons, prefer TextViews or Buttons.


### Test frameworks

Android SDK's testing framework is still infant, specially regarding UI tests. Android Gradle currently implements a test task called [`connectedAndroidTest`](http://tools.android.com/tech-docs/new-build-system/user-guide#TOC-Testing) which runs JUnit tests that you created, using an [extension of JUnit with helpers for Android](http://developer.android.com/reference/android/test/package-summary.html). This means you will need to run tests connected to a device, or an emulator. Follow the official guide [[1]](http://developer.android.com/tools/testing/testing_android.html) [[2]](http://developer.android.com/tools/testing/activity_test.html) for testing.

**Use [Robolectric](http://robolectric.org/) only for unit tests, not for views.** It is a test framework seeking to provide tests "disconnected from device" for the sake of development speed, suitable specially for unit tests on models and view models. However, testing under Robolectric is inaccurate and incomplete regarding UI tests. You will have problems testing UI elements related to animations, dialogs, etc, and this will be complicated by the fact that you are "walking in the dark" (testing without seeing the screen being controlled).

**[Robotium](https://code.google.com/p/robotium/) makes writing UI tests easy.** You do not need Robotium for running connected tests for UI cases, but it will probably be beneficial to you because of its many helpers to get and analyse views, and control the screen. Test cases will look as simple as:

```java
solo.sendKey(Solo.MENU);
solo.clickOnText("More"); // searches for the first occurence of "More" and clicks on it
solo.clickOnText("Preferences");
solo.clickOnText("Edit File Extensions");
Assert.assertTrue(solo.searchText("rtf"));
```

### Emulators

If you are developing Android apps as a profession, buy a license for the [Genymotion emulator](http://www.genymotion.com/). Genymotion emulators run at a faster frames/sec rate than typical AVD emulators. They have tools for demoing your app, emulating network connection quality, GPS positions, etc. They are also ideal for connected tests. You have access to many (not all) different devices, so the cost of a Genymotion license is actually much cheaper than buying multiple real devices.

Caveats are: Genymotion emulators don't ship all Google services such as Google Play Store and Maps. You might also need to test Samsung-specific APIs, so it's necessary to have a real Samsung device.

### Proguard configuration

[ProGuard](http://proguard.sourceforge.net/) is normally used on Android projects to shrink and obfuscate the packaged code.

Whether you are using ProGuard or not depends on your project configuration. Usually you would configure gradle to use ProGuard when building a release apk.

```groovy
buildTypes {
    debug {
        minifyEnabled false
    }
    release {
        signingConfig signingConfigs.release
        minifyEnabled true
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
}
```

In order to determine which code has to be preserved and which code can be discarded or obfuscated, you have to specify one or more entry points to your code. These entry points are typically classes with main methods, applets, midlets, activities, etc.
Android framework uses a default configuration which can be found from `SDK_HOME/tools/proguard/proguard-android.txt`. Using the above configuration, custom project-specific ProGuard rules, as defined in `my-project/app/proguard-rules.pro`, will be appended to the default configuration.

A common problem related to ProGuard is to see the application crashing on startup with `ClassNotFoundException` or `NoSuchFieldException` or similar, even though the build command (i.e. `assembleRelease`) succeeded without warnings.
This means one out of two things:

1. ProGuard has removed the class, enum, method, field or annotation, considering it's not required.
2. ProGuard has obfuscated (renamed) the class, enum or field name, but it's being used indirectly by its original name, i.e. through Java reflection.

Check `app/build/outputs/proguard/release/usage.txt` to see if the object in question has been removed.
Check `app/build/outputs/proguard/release/mapping.txt` to see if the object in question has been obfuscated.

In order to prevent ProGuard from *stripping away* needed classes or class members, add a `keep` options to your ProGuard config:
```
-keep class com.futurice.project.MyClass { *; }
```

To prevent ProGuard from *obfuscating* classes or class members, add a `keepnames`:
```
-keepnames class com.futurice.project.MyClass { *; }
```

Check [this template's ProGuard config](https://github.com/futurice/android-best-practices/blob/master/templates/rx-architecture/app/proguard-rules.pro) for some examples.
Read more at [Proguard](http://proguard.sourceforge.net/#manual/examples.html) for examples.

**Early on in your project, make a release build** to check whether ProGuard rules are correctly keeping whatever is important. Also whenever you include new libraries, make a release build and test the apk on a device. Don't wait until your app is finally version "1.0" to make a release build, you might get several unpleasant surprises and a short time to fix them.

**Tip.** Save the `mapping.txt` file for every release that you publish to your users. By retaining a copy of the `mapping.txt` file for each release build, you ensure that you can debug a problem if a user encounters a bug and submits an obfuscated stack trace.

**DexGuard**. If you need hard-core tools for optimizing, and specially obfuscating release code, consider [DexGuard](http://www.saikoa.com/dexguard), a commercial software made by the same team that built ProGuard. It can also easily split Dex files to solve the 65k methods limitation.

### Thanks to

Antti Lammi, Joni Karppinen, Peter Tackage, Timo Tuominen, Vera Izrailit, Vihtori Mäntylä, Mark Voit, Andre Medeiros, Paul Houghton and other Futurice developers for sharing their knowledge on Android development.

### License

[Futurice Oy](http://www.futurice.com)
Creative Commons Attribution 4.0 International (CC BY 4.0)
