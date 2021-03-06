Mac application bundle plugin for Gradle
========================================

Gradle plugin that can be used to create a Mac OS
[application bundle](https://en.wikipedia.org/wiki/Bundle_%28macOS%29) as part of the build. 
This plugin is based on the [Ant plugin by TheInfiniteKind](https://github.com/TheInfiniteKind/appbundler), 
but makes it available via Gradle. It also adds the ability to sign the application bundle and
package it within an installer.

Usage
-----

The plugin is available from the [Gradle plugin registry](https://plugins.gradle.org). Adding the
plugin to the build is done by adding the following to `build.gradle`:

    plugins {
        id "nl.colorize.gradle.macapplicationbundle" version "2021.3"
    }
    
The plugin can be configured using the `macApplicationBundle` block. The configuration
options correspong to the ones from the [Ant plugin](https://github.com/TheInfiniteKind/appbundler).
The following shows an example on how to define this configuration in Gradle:

    macApplicationBundle {
        name = "Example"
        identifier = "com.example"
        description = "A description for your application"
        copyright = "Copyright 2021"
        bundleVersion = "1.0"
        icon = "resources/icon.icns"
        applicationCategory = "public.app-category.developer-tools"
        mainClassName = "com.example.app.Main"
        outputDir = "${buildDir}"
        signIdentityApp = "your signing identity"
        signIdentityInstaller = "your signing identity"
        startOnFirstThread = false
    }
    
Note that, in addition to the the `bundleVersion` property, there is also the concept of build
version. This is normally the same as the bundle version, but can be manually specified for each
build by setting the `buildversion` system property. 
    
By default, the contents of the application will be based on all JAR files produces by the project,
as described by the `libsDir` property. This behavior can be replaced by setting the `contentDir`
property in the plugin's configuration. Including all application binaries, resources, and
libraries is easiest by creating a single "fat JAR" file:

    jar {
        from {
            configurations.runtimeClasspath.collect { it.isDirectory() ? it : zipTree(it) }
        }
     }
    
The plugin adds a number of tasks to the project that use this configuration:

- **createApplicationBundle**: Creates the application bundle in the specified directory.
- **signApplicationBundle**: Signs the created application bundle and packages it into an installer
  so that it can be distributed. 

Note that the tasks are *not* added to any standard tasks such as `assemble`, as Mac application
bundles can only be created when running the build on a Mac, making the tasks incompatible with
some workflows (i.e. continuous integration pipelines that tend to use Linux servers). 
    
The plugin requires the `JAVA_HOME` environment variable to be set, as it needs to add an embedded
version of Java to the application bundle.

Signing the application bundle is mandatory for distributing the application. This in turn needs
a valid Apple developer account, and corresponding certificates. You will need two certificates:
one for the application, and one for the installer. Please do not hard-code the signing identity
into the build file. It is better to define them in the Gradle properties file at
`~/.gradle/gradle.properties` and then access them from there.   

Build instructions
------------------

Building the plugin itself can only be done on Mac OS. It also requires the following:

- [Java JDK](http://java.oracle.com) 11+
- [Gradle](http://gradle.org)
- [Ant](https://ant.apache.org)

The following Gradle build tasks are available:

- `gradle clean` cleans the build directory
- `gradle assemble` creates the JAR file for distribution
- `gradle test` runs all unit tests
- `gradle coverage` runs all unit tests and reports on test coverage
- `gradle publishPlugins` publishes the plugin to the Gradle plugin portal (requires account)
  
License
-------

Copyright 2010-2021 Colorize

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

        http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
