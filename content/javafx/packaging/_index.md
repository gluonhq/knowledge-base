+++
title = "Packaging"
chapter = true
weight = 1
pre = "<b>1.1. </b>"
+++

# Packaging

Using `jpackage`, a JavaFX application can easily be transformed into a native application.
The approach discussed in this section results in a native application that comes with a
bundled Java Virtual Machine (JVM) and a set of Java Modules that are needed to run the
application.
This is different from the approach where the `gluonfx` plugin is used to create a native
image that does not contain a JVM or the required modules, as those are compiled ahead of
time (AOT) into the executable.

## Linking

First, you need to detect what modules your application is using:
```
detected_modules=`$JAVA_HOME/bin/jdeps \
  -q \
  --multi-release ${JAVA_VERSION} \
  --ignore-missing-deps \
  --print-module-deps \
  --class-path "target/installer/input/libs/*" \
    target/classes/${MAIN_CLASS}.class`
```

Apart from those detected modules, you may specify additional modules, e.g.
```
manual_modules=,jdk.crypto.ec,jdk.localedata
```

Next, you can create a runtime continaining those modules:
```
$JAVA_HOME/bin/jlink \
  --strip-native-commands \
  --no-header-files \
  --no-man-pages  \
  --compress=2  \
  --strip-debug \
  --add-modules "${detected_modules}${manual_modules}" \
  --include-locales=en \
  --output target/java-runtime
```

## Packaging

The following command creates a native installer:

```
$JAVA_HOME/bin/jpackage \
--dest target/installer \
--input target/installer/input/libs \
--name ${APP_NAME} \
--main-class ${LAUNCHER_CLASS} \
--main-jar ${MAIN_JAR} \
--java-options -Xmx2048m \
--runtime-image target/java-runtime \
--icon ${ICON_PATH} \
--app-version ${APP_VERSION} \
--vendor "Gluon" \
--copyright "Copyright © 2021 Gluon" \
"$@"
```

If you want to test the bundle without creating an installer (and installing the installer),
you can specify the type of the output to be a `app-image` as shown below:

```
$JAVA_HOME/bin/jpackage \
--dest target/installer \
--input target/installer/input/libs \
--name ${APP_NAME} \
--main-class ${LAUNCHER_CLASS} \
--main-jar ${MAIN_JAR} \
--java-options -Xmx2048m \
--runtime-image target/java-runtime \
--icon ${ICON_PATH} \
--app-version ${APP_VERSION} \
--vendor "Gluon" \
--copyright "Copyright © 2021 Gluon" \
--type app-image
"$@"
```

There are many other options you can specify, check
```
jpackage --help
```
