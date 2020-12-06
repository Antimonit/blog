---
layout: post
title:  "From Gradle scripts to buildSrc and back to precompiled scripts"
date:   2020-12-06 17:25:49 +0900
tags:   programming
---

Had you ever worked on a project with many feature modules, you might have found yourself duplicating nearly an identical setup in most of the features.
A typical `gradle.build` of one such feature could look like this:

```groovy
plugins {
    id("com.android.library")
    id("kotlin-android")
    id("kotlin-android-extensions")
}

android {
    compileSdkVersion 29

    defaultConfig {
        minSdkVersion 21
        targetSdkVersion 29
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
        consumerProguardFiles "consumer-rules.pro"
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile("proguard-android-optimize.txt"), "proguard-rules.pro"
        }
    }  

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}

dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib"
    implementation "androidx.core:core-ktx:1.3.2"
    implementation "androidx.appcompat:appcompat:1.2.0"

    testImplementation "junit:junit:4.13.1"
    androidTestImplementation "androidx.test.ext:junit:1.1.2"
    androidTestImplementation "androidx.test.espresso:espresso-core:3.3.0"
}
```
{: data-filename="/feature/build.gradle" }

Occasionally you may apply an extra plugin or define different dependencies, but for the most part, the configuration of the `android` block is duplicated across multiple features.

Things get worse once you start defining custom build types and build flavors which should be unified across all modules
or when you try to configure a project-wide plugin such as Jacoco which requires different configuration based on whether the module has either `com.android.application` or `com.android.library` plugin applied or is an Android-free module.

Duplicating any kind of code is the easiest way to make codebase harder to maintain and to inadvertently introduce bugs when the copies start to diverge.
The same reasoning applies to build scripts and utilizing `buildSrc` is one way to solve the problem.

## buildSrc

The `buildSrc` directory is automatically recognized and compiled by Gradle before any other part of the project.
It is a good place to define and maintain imperative build logic, such as custom tasks or plugins.

The compiled code from the `buildSrc` is put in the classpath of the root build script, making it available to any other module in the project.
We can, for example, define a simple `Config` class and, after syncing the project, use it in our Gradle scripts.

```java
public class Config {
    static final int COMPILE_SDK = 29;
    static final int TARGET_SDK = 29;
    static final int MIN_SDK = 21;
}
```
{: data-filename="/buildSrc/src/main/java/Config.java"}

```groovy
android {
    compileSdkVersion Config.COMPILE_SDK

    defaultConfig {
        minSdkVersion Config.MIN_SDK
        targetSdkVersion Config.TARGET_SDK
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }
    ...
}
...
```
{: data-filename="/feature/build.gradle" }

### buildSrc structure

The `buildSrc` directory is recognized as a module and follows the same structure as any other module.

- Main source code is located in `/buildSrc/src/main/<language>/`
- Test source code is located in `/buildSrc/src/test/<language>/`
- Gradle script located at `/buildSrc/build.gradle`

It has `java` and `groovy` plugins already applied and, as such, we can use either of those languages to write our code.
In order to use other languages, such as `kotlin` or perhaps `scala`, we have to apply the plugin ourselves.

```groovy
plugins {
    id("org.jetbrains.kotlin.jvm") version "1.4.10"
}

repositories {
    jcenter()
}

dependencies {
    implementation("org.jetbrains.kotlin:kotlin-stdlib")
}
```
{: data-filename="/buildSrc/build.gradle" }

```kotlin
object Config {
    const val COMPILE_SDK = 29
    const val TARGET_SDK = 29
    const val MIN_SDK = 21
}
```
{: data-filename="/buildSrc/src/main/kotlin/Config.kt" }

### Custom task

Of course, we are not limited to defining static constants.
Let's encapsulate some behavior in a Gradle task. For the sake of simplicity, let's create a primitive task that prints current JVM memory stats to stdout.

```kotlin
open class ReportJvmMemoryTask : DefaultTask() {
    @TaskAction
    fun run() {
        val runtime = Runtime.getRuntime()
        println("Free JVM memory: ${runtime.freeMemory() shr 20}MB.")
        println("Total JVM memory: ${runtime.totalMemory() shr 20}MB.")
        println("Max JVM memory: ${runtime.maxMemory() shr 20}MB.")
    }
}
```
{: data-filename="/buildSrc/src/main/kotlin/tasks/ReportJvmMemoryTask.kt" }

We have created a task, but on its own, it has no effect. We must also register it in the Gradle script of some of our modules.

```groovy
...
task reportJvmMemory(type: ReportJvmMemoryTask)
build.finalizedBy(reportJvmMemory)
```
{: data-filename="/feature/build.gradle" }

The task will now run automatically after the `:feature:build` task.
We can also run it explicitly whenever we want using `gradlew :feature:reportJvmMemory`

### Custom plugin

Custom tasks can be useful but they won't help us organize all the duplicated configuration in our modules.
Let's take it a level further by defining a custom Gradle plugin that encapsulates common android library configuration.

First of all, we have to add appropriate dependencies to our `buildSrc` module since we are going to use API provided by Android (such as `BaseExtension`) and Kotlin (such as `KotlinJvmOptions`) plugins.

```groovy
plugins {
    id("org.jetbrains.kotlin.jvm") version "1.4.10"
}

repositories {
    jcenter()
    google()
}

dependencies {
    implementation("com.android.tools.build:gradle:4.1.0")
    implementation("org.jetbrains.kotlin:kotlin-stdlib")
    implementation("org.jetbrains.kotlin:kotlin-gradle-plugin")
}
```
{: data-filename="/buildSrc/build.gradle" }

Now, similarly to the `ReportJvmMemoryTask`, we can define a custom plugin by implementing the `Plugin` interface and move the configuration from the Gradle script there.

```kotlin
class BaseAndroidLibrary : Plugin<Project> {

    override fun apply(target: Project) {
        target.plugins.apply("com.android.library")
        target.plugins.apply("kotlin-android")
        target.plugins.apply("kotlin-android-extensions")

        target.extensions.configure(BaseExtension::class.java) { android ->
            android.compileSdkVersion(Config.COMPILE_SDK)

            android.defaultConfig {
                it.minSdkVersion(Config.MIN_SDK)
                it.targetSdkVersion(Config.TARGET_SDK)
                it.versionCode = 1
                it.versionName = "1.0"

                it.multiDexEnabled = true
                it.testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
                it.consumerProguardFiles("consumer-rules.pro")
            }

            android.compileOptions {
                it.sourceCompatibility = JavaVersion.VERSION_1_8
                it.targetCompatibility = JavaVersion.VERSION_1_8
            }

            android.buildTypes {
                it.named("release") { release ->
                    release.isMinifyEnabled = false
                    release.proguardFiles(android.getDefaultProguardFile("proguard-android-optimize.txt"), "proguard-rules.pro")
                }
            }
        }
    }
}
```
{: data-filename="/buildSrc/src/main/kotlin/BaseAndroidLibrary.kt" }

Notice we still access the `Config` class defined within the `buildSrc` module just like we did before.
Since the plugin is written in Kotlin, we can utilize all of its language features, including extension functions, sealed classes, objects, ... whatever you please.

Finally, we can apply the plugin to all of our feature modules and remove the `android` block altogether, since all common configuration is now encapsulated in our plugin.
The only thing left is the list of dependencies which varies from module to module.

```groovy
apply plugin: BaseAndroidLibrary

dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib"
    implementation "androidx.core:core-ktx:1.3.2"
    implementation "androidx.appcompat:appcompat:1.2.0"

    testImplementation "junit:junit:4.13.1"
    androidTestImplementation "androidx.test.ext:junit:1.1.2"
    androidTestImplementation "androidx.test.espresso:espresso-core:3.3.0"
}
```
{: data-filename="/feature/build.gradle" }

## kotlin-dsl and precompiled scripts

Unfortunately, by extracting the logic from a script into an encapsulated plugin, we have sacrificed a bit of legibility
and, in many places, we have to use special code constructs to achieve the same thing as in the original script.
To alleviate the situation a little, we can create custom extensions to bring back the original concise syntax.

```kotlin
fun Project.android(configure: LibraryExtension.() -> Unit) {
    extensions.configure(LibraryExtension::class.java, configure)
}

fun BaseExtension.kotlinOptions(configure: KotlinJvmOptions.() -> Unit) {
    (this as ExtensionAware).extensions.configure(KotlinJvmOptions::class.java, configure)
}
```

But that does not solve the issue that we have to discover such special code constructs ourselves.
It is simply not obvious that `android` block in Android library modules is equivalent to `extensions.configure(LibraryExtension::class.java)` and discovering this fact is not trivial either.

**That is where the `kotlin-dsl` plugin fills in the gap between type-safety of Kotlin and wildly dynamic world of Groovy.**

To set this feature up, we must apply `kotlin-dsl` and `precompiled-script-plugins` plugins to our `buildSrc` module.
It's not strictly necessary to make things work, but it will be simpler if we use Kotlin for our Gradle build scripts as well.
Let's change the `build.gradle` to `build.gradle.kts` and update its contents.

```kotlin
plugins {
    `kotlin-dsl`
    `kotlin-dsl-precompiled-script-plugins`
}

repositories {
    jcenter()
    google()
}

dependencies {
    implementation("com.android.tools.build:gradle:4.1.0")
    implementation("org.jetbrains.kotlin:kotlin-stdlib:1.4.10")
    implementation("org.jetbrains.kotlin:kotlin-gradle-plugin:1.4.10")
}
```
{: data-filename="/buildSrc/build.gradle.kts" }

Then, take our `BaseAndroidLibrary.kt` and convert it to a Kotlin Gradle script named `base-android-library.gradle.kts`.
We can get rid of the `Plugin<Project>` boilerplate and
any of the extensions we have created to make the syntax smoother.

```kotlin
plugins {
    id("com.android.library")
    id("kotlin-android")
}

android {
    compileSdkVersion(Config.COMPILE_SDK)

    defaultConfig {
        minSdkVersion(Config.MIN_SDK)
        targetSdkVersion(Config.TARGET_SDK)
        versionCode = 1
        versionName = "1.0"

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
        consumerProguardFiles("consumer-rules.pro")
    }

    buildTypes {
        named("release") {
            isMinifyEnabled = false
            proguardFiles(getDefaultProguardFile("proguard-android-optimize.txt"), "proguard-rules.pro")
        }
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_1_8
        targetCompatibility = JavaVersion.VERSION_1_8
    }

    kotlinOptions {
        jvmTarget = "1.8"
    }
}
```
{: data-filename="/buildSrc/src/main/kotlin/base-android-library.gradle.kts" }

When we sync the project, the plugin will inspect whatever plugins are applied in the `plugins` block and generate type-safe accessors for whatever configuration those plugins provide.
Here are two such samples.

```kotlin
/**
 * Configures the [android][com.android.build.gradle.LibraryExtension] extension.
 */
internal
fun org.gradle.api.Project.`android`(configure: com.android.build.gradle.LibraryExtension.() -> Unit): Unit =
    (this as org.gradle.api.plugins.ExtensionAware).extensions.configure("android", configure)

/**
 * Configures the [kotlinOptions][org.jetbrains.kotlin.gradle.dsl.KotlinJvmOptions] extension.
 */
internal
fun com.android.build.gradle.LibraryExtension.`kotlinOptions`(configure: org.jetbrains.kotlin.gradle.dsl.KotlinJvmOptions.() -> Unit): Unit =
    (this as org.gradle.api.plugins.ExtensionAware).extensions.configure("kotlinOptions", configure)
```

Finally, let's update the method of how we apply the plugin to our modules.
Instead of directly applying `BaseAndroidLibrary`, let's make use of the `plugins` block and apply the plugin by its id.
Precompiled scripts are assigned the same id as their file names.

```groovy
plugins {
    id("base-android-library")
}

dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib"
    implementation "androidx.core:core-ktx:1.3.2"
    implementation "androidx.appcompat:appcompat:1.2.0"

    testImplementation "junit:junit:4.13.1"
    androidTestImplementation "androidx.test.ext:junit:1.1.2"
    androidTestImplementation "androidx.test.espresso:espresso-core:3.3.0"
}
```
{: data-filename="/feature/build.gradle" }

We don't have to keep all precompiled plugins in the root directory; we can put them into subdirectories.
A script located in `com/example/plugins/base-android-library.gradle.kts` would be assigned an id `com.example.plugins.base-android-library`.

It is also possible to write precompiled plugin scripts in Groovy, but the necessary set up is slightly different.
I would not recommend this since sacrificing the ability to navigate in code in the IDE is a too great price to pay,
but it might be necessary in a situation where a 3rd party plugin has an obscure API that, for some reason, does not work well with Kotlin DSL.

Check [Gradle documentation](https://docs.gradle.org/current/userguide/custom_plugins.html#sec:precompiled_plugins) for more information.

## Extra

The `build.gradle` script inside of `buildSrc` has a similar role as the `buildscript` block in the root `build.gradle` script.
Whatever `implementation` dependency we would declare in the `buildSrc` module will have
the same effect on the configuration of the project as if we declared it as a `classpath` dependency inside the `buildscript` block.
The added benefit of defining them in `buildSrc` is that we can access whatever classes these dependencies contain and use them in a type-safe manner in our Kotlin scripts.

```groovy
buildscript {
    // Don't declare classpath dependencies here.
    // Instead, declare them as implementation dependency in `/buildSrc/build.gradle.kts`.
    // It has the same outcome on the configuration of the project with the benefit that
    // we can use type-safe kotlin scripts defined in `buildSrc` sources.
}

allprojects {
    repositories {
        google()
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```
{: data-filename="/build.gradle" }

## Conclusion

Extracting common build configuration from modules and encapsulating it in a custom plugin is a great way to make sure all of our modules are configured the same way.

When used together with `precompiled-scripts-plugin`, we can use the extracted configuration as is and Gradle will generate the plugin for us.

Kotlin DSL plugin allows us to write build scripts in Kotlin, which will give us more confidence and better IDE integration when writing custom tasks and plugins or when other plugins require extensive configuration.

## See also

- [Organizing Gradle Projects](https://docs.gradle.org/current/userguide/organizing_gradle_projects.html#sec:build_sources)
- [Developing Custom Gradle Plugins - Precompiled script plugins](https://docs.gradle.org/current/userguide/custom_plugins.html#sec:precompiled_plugins)
- [Precompiled Script Plugin Sample](https://docs.gradle.org/6.4/samples/sample_precompiled_script_plugin.html)
- [Gradle Kotlin DSL Primer](https://docs.gradle.org/current/userguide/kotlin_dsl.html)
