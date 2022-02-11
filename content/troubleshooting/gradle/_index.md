+++
title = "Gradle"
chapter = true
weight = 1
pre = "<b>1.1. </b>"
+++

# Gradle

When building Android applications, Gluon Substrate will under the hood invoke
gradle to execute the android-specific tasks. This can lead to the issue
reported at [the Substrate issue tracker](https://github.com/gluonhq/substrate/issues/1118).

If you have a global `gradle.properties` file, properties defined in that
file have precedence over project-specific properties.
Hence, you want to avoid setting the general `org.gradle.jvmargs` property
in that global propery file.

Have a look at [https://github.com/gluonhq/substrate/issues/1118](https://github.com/gluonhq/substrate/issues/1118) for more details.

