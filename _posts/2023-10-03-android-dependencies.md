---
title: Android - Library, Dependencies and Gradle.
date: 2023-10-03
categories: [Android, Fundamentals]
tags: [java, android]
---

## About

The gradle build systems allows use to attach and include external binaries or other library modules to our build as dependencies. These dependencies can be located on our PC or from a remote repo like github, and any transit repositories that they require will be automatically included.

To include a dependency into our build, you must specify the dependency inside our `build.gradle` file.



> **What are dependencies**\
> Dependencies or libraries are basically precompiled source codes. Have you noticed that when we write our custom classes inside the java folder, we dont need to do anything special at all, however, when we want to use other libraries, often we do not receive them in a .java file, instead it is often in a `.jar` file.\
> \
> This is basically the java byte code. In this section we will learn how to add these compiled binaries into our project, and use them.
{: .prompt-tip}


Now, when selecting a gradle language, you have the option of either kotlin or grovvy. This guide will show both usecases. Be sure to watch out for the language highlight in the code snippets and infer the synthax differences.


<br><br>

---
## build.gradle(Module: )

Essentially these are the dependancies that we are referring to, they are declared within the `build.gradle.kts` (or `build.gradle` for gradle) files: 

```kotlin
dependencies {  
    implementation("androidx.appcompat:appcompat:1.6.1")  
    implementation("com.google.android.material:material:1.8.0")  
    implementation("androidx.constraintlayout:constraintlayout:2.1.4")  
    testImplementation("junit:junit:4.13.2")  
    androidTestImplementation("androidx.test.ext:junit:1.1.5")  
    androidTestImplementation("androidx.test.espresso:espresso-core:3.5.1")  
}
```

<br>

### Library module dependency

Here's how dependencies work. We declare the library inside the `build.gradle`.kts,
```kotlin
dependencies {
    // Local Library Module Dependency
    implementation project(":mylibrary")
}
```

Then we must also declare them inside the `settings.gradle.kts`, under "include"
```kotlin
pluginManagement {  
    repositories {  
        google()  
        mavenCentral()  
        gradlePluginPortal()  
    }  
}  
dependencyResolutionManagement {  
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)  
    repositories {  
        google()  
        mavenCentral()  
    }  
}  
  
rootProject.name = "Stone Camera"  
include(":app")
include(:"mylibrary")  // Here
```

> **Important**\
> When you build your application, the build system compiles the library module and packages that resulting compile will be included in the app.
{: .prompt-tip}

<br>

### Local binary dependency
For local binaries (downloaded `.jar` files), Gradle declares dependencies on jar files inside your project's libs folder.

```gradle
dependencies {
    // Local Library Module Dependency
    implementation project(":mylibrary")
    
    // Local binary dependency
    implementation fileTree(dir: "libs", includes: ['*.jar'])
    
    // Alternatively, to specify specific files...
    implement files("libs/myFile.jar", "libs/myOtherfile.jar")
}
```

<br>

### Remote binary dependency
This will allow gradle to attempt to fetch the files over the internet.
```gradle
dependencies {
    // Local Library Module Dependency
    implementation project(":mylibrary")
    
    // Local binary dependency
    implementation fileTree(dir: "libs", includes: ['*.jar'])
    
    // Alternatively, to specify specific files...
    implement files("libs/myFile.jar", "libs/myOtherfile.jar")
    
    // Remote binary dependencies
    implementation "com.example.android:app-magic:12.3"
}
```


The way it works is to ask gradle to look for the file named "app-magic", version 12.3 from the group/package "com.example.android". A group dependency like this requires that we decalre the approapriate remote repositories where Gradle should look for the library. If the library doesnt already exist locally, they will be pulled from the remote side  when the build requires it. 


> **Groovy or kotlin**\
> A lot of times our gradle build files could be written in groovy or kotlin synthax. This maybe confusing. However its important to take node the general context works like this:\
> `type "argument"` this is where a function was redacted; `implements "com.somewhere"`\
> `type functionName("arguments")`\
> `type functionName(kwarg: "argument")`
{: .prompt-tip}

<br><br>

---
## Practical Demonstration

Using the glide library which is an image loading and caching library for android focused on smooth scrolling, as our example...

The source code is [here](https://github.com/bumptech/glide). So how do we add a github repo project files into our app?

Well, if we scroll down we can see a few examples and documentations:

![](/assets/img/attachments/android/android-dependencies-glide-library.png)

There are mainly 2 ways to get this library into our project:

<br>


### Method 1: Use as local binary
The first thing we got to do is to download their `.jar` files and put in inside our project's `libs` folder and include it as a local binary:

<br>

### Method 2: Have Gradle download it from repo
As previously mentioned, we can have gradle to download this files automatically via the "Remote dependency" method.

So lets copy this code from the readme page:
```gradle
repositories {
<<<<<<< HEAD
    google()
    mavenCentral()
}

dependencies {
    implementation 'com.github.bumptech.glide:glide:4.16.0'
=======
  google()
  mavenCentral()
}

dependencies {
  implementation 'com.github.bumptech.glide:glide:4.16.0'
>>>>>>> 5710cc5df95aad49a9e1b866bd2992ca4006e972
}
```

Into our build.gradle(Project:) gradle, we will often see google and mavenCentral is already preincluded.

```gradle
// Top-level build file where you can add .......
buildscript {
    repositories {
        google()
        mavenCentral()
    }
    dependencies {
        classpath "com.android.tools.build:gradle7.0.4"
        
        // NOTE: Do not place your application dependencies here;
        // they belong in the individual build.gradle files
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

Towards our `build.gradle(Module:)` gradle's dependencies section. We will see these code that has already preincluded in a project template. 
All wee need to do is to all ours in:

```gradle
dependencies {
    
    implementation "androidx.appcompat:appcompat:1.2.0"
    implementation "con.google.android.material:material:1.3.0"
    implementation "androidx.constraintlayout:constraintlayout:2.1.2"
    testImplementation "junit:junit4.+"
    androidTestImplementation "androidx.test.ext.junit:1.1.2"
    andoridTestImplementation "androidx.test.espresso:espresso-core:3.3.0"
    
    // Our dependencies code goes here
    implementation "com.github.bumptech.glide:glide:4.12.0"
    annotationProcessor "com.github.bumptech.glide:compiler:4.12.0"
}
```

<br><br>

---
## Modern Gradle outlines & Common errors

In 2023 september, our gradle outlines will look vastly different, first of all, our `build.gradle(Project: )` will look like this:



```gradle
// Top-level build file where you can add ......
plugins {  
    id 'com.android.application' version '8.0.2' apply false  
    id 'com.android.library' version '8.0.2' apply false  
}
```

And the settings.gradle(Project settings) will look like this:
```gradle
pluginManagement {  
    repositories {  
        google()  
        mavenCentral()  
        gradlePluginPortal()  
    }  
}  
dependencyResolutionManagement {  
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)  
    repositories {  
        google()  
        mavenCentral()  
    }  
}  
rootProject.name = "ExampleGlideProject"  
include ':app'
```

So when adding an external dependencies, we will instead declare the repositories inside the project settings instead, and leave the `build.gradle(Project:)` untouched.

The dependencies can be added into the `build.gradle(Module:)` as usual.

<br>

### Dependencies not fully resolved
This happens from time to time when you have already added the dependencies, yet you still see a missing class error:

![](/assets/img/attachments/android/android-dependencies-missing-class-error.png)

You need to make sure you added the repository into your build settings! Read the documentations carefully.

![](/assets/img/attachments/android/android-studio-pdf-viewer-documentation.png)


Add these repositories into your `settings.gradle(Project settings)` file
```gradle
pluginManagement {  
    repositories {  
        google()  
        mavenCentral()  
        gradlePluginPortal()  
        jcenter()  // Add here, usually optional
    }  
}  
dependencyResolutionManagement {  
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)  
    repositories {  
        google()  
        mavenCentral()  
        jcenter()  //Must Add here
    }  
}  
rootProject.name = "ExampleGlideProject"  
include ':app'
```

<br>

### Duplicate Class errors
This also happens from time to time when we use external libraries. This conflicts usually comes from The external library using its own version of some class implementations. [Details](https://stackoverflow.com/questions/55909804/duplicate-class-android-support-v4-app-inotificationsidechannel-found-in-modules):


#### The fix:
Add these lines of code inside `gradle.properties`. Make sure you dont repeat lines already present:
```gradle
android.useAndroidX=true
android.enableJetifier=true
```


#### Why it works?:
Your project (or one of its sub-projects, or a dependency) is no longer using `android.support` libraries and instead is using `androidx` libraries, which causes conflicts if mixed with `android.support` libraries.

If you want to use `androidx`-namespaced libraries in an old project, you need to set the compile SDK to Android 9.0 (API level 28) or higher but below "API level 31", and set both of the mentioned Android Gradle plugin flags to `true`.

**`android.useAndroidX`**: When this flag is set to `true`, the Android plugin uses the appropriate AndroidX library instead of a Support Library. The flag is `false` by default if it is not specified.

**`android.enableJetifier`**: When this flag is set to `true`, the Android plugin automatically migrates existing third-party libraries to use AndroidX dependencies by rewriting their binaries. The flag is `false` by default if it is not specified.

