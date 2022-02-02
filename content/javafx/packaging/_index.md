+++
title = "Packaging"
chapter = true
weight = 1
pre = "<b>1.1. </b>"
+++

# Packaging

Using [jpackage](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jpackage.html),
a Java/FX application can easily be transformed into a package/installer in a platform-specific format.
The approach discussed in this section results in a package that comes with a bundled 
Java Virtual Machine (JVM) and a set of Java Modules that are needed to run the application.
This is different from the approach where the `gluonfx` plugin is used to create a native
image that does not contain a JVM or the required modules, as those are compiled ahead of
time (AOT) into the executable.

## Linking

First, find the modules used in the application. This can be done using `jdeps` command:
```
detected_modules=`$JAVA_HOME/bin/jdeps \
  -q \
  --multi-release ${JAVA_VERSION} \
  --ignore-missing-deps \
  --print-module-deps \
  --class-path "target/installer/input/libs/*" \
    target/classes/${MAIN_CLASS}.class`
echo $detected_modules
```

Apart from those detected modules, any additional modules can be specified manually:
```
manual_modules=jdk.crypto.ec,jdk.localedata
```

Next, a runtime containing these modules can be created by using the `jlink` command:
```
$JAVA_HOME/bin/jlink \
  --strip-native-commands \
  --no-header-files \
  --no-man-pages  \
  --compress=2  \
  --strip-debug \
  --add-modules "${detected_modules},${manual_modules}" \
  --include-locales=en \
  --output target/java-runtime
```

## Packaging

The following command creates a packaged application (or installer bundle) for the platform:

```
$JAVA_HOME/bin/jpackage \
--dest target/installer \
--input target/installer/input/libs \
--name ${APP_NAME} \
--main-class ${MAIN_CLASS} \
--main-jar ${MAIN_JAR} \
--java-options -Xmx2048m \
--runtime-image target/java-runtime \
--icon ${ICON_PATH} \
--app-version ${APP_VERSION} \
--vendor "Gluon" \
--copyright "Copyright © 2021 Gluon" \
--type ${INSTALLER_TYPE}
```

If you want to test the bundle without creating an installer,
you can specify the `type` to be a `app-image` as shown below:

```
$JAVA_HOME/bin/jpackage \
--dest target/installer \
--input target/installer/input/libs \
--name ${APP_NAME} \
--main-class ${MAIN_CLASS} \
--main-jar ${MAIN_JAR} \
--java-options -Xmx2048m \
--runtime-image target/java-runtime \
--icon ${ICON_PATH} \
--app-version ${APP_VERSION} \
--vendor "Gluon" \
--copyright "Copyright © 2021 Gluon" \
--type app-image
```

`jpackage` comes with an array of options, many of which are platform specific.
Use the `--help` command to print a list of valid options for the current platform:

```
jpackage --help
```
