---
layout: post
title: "Limiting threads count for GreenRobot EventBus"
date: 2014-03-06 22:38:54 -0500
comments: true
categories: 
    - development
    - greenrobot
    - android
---

In one of my projects I used [EventBus] library. The library is pretty cool and I would recommend everybody to use it.

But I found one small issue with this library - in case you send too many events very fast (i.e. more than 1000 per second) you may face issue with application crash due to inability to create new thread.

The problem is lying in the used `ExecutorService`:

``` java

package de.greenrobot.event;
// imports
public final class EventBus {
    static ExecutorService executorService = Executors.newCachedThreadPool();

    // class content skipped 
}

```

As you can see, it calls `newCachedThreadPool` method to create thread pool.

From [javadoc][ExecutorsService] for this method:

> Creates a thread pool that creates new threads as needed, but will reuse 
> previously constructed threads when they are available.

This is where we hit the wall - if we send too many requests and each request is slow (i.e. saving into database or network request) then we will try to create too many threads and app will crash.

Thankfully, `executorService` is `static` but not `final`. And this will allow us to solve the issue easily. 

Let's create our configurer class in package `de.greenrobot.event` so we will be able to access `executorService` property which is package-visible and override it with different thread pool:

``` java EventBusExecutorServiceConfigurer

package de.greenrobot.event;

import java.util.concurrent.Executors;

public class EventBusExecutorServiceConfigurer {

    public void configureEventBus() {
        ExecutorService executorServiceToUse;
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
            executorServiceToUse = (ExecutorService) AsyncTask.THREAD_POOL_EXECUTOR;
        } else {
            executorServiceToUse = Executors.newFixedThreadPool(
                Runtime.getRuntime().availableProcessors());
        }
        EventBus.executorService = executorServiceToUse;
    }
}

```

Android 3.0+ has special [THREAD_POOL_EXECUTOR] which is supposed to be used by asynchronous tasks, so we use it on newer devices. On older, pre-Honeycomb devices we create our own thread pool.

And then in Android application we configure `executorService`:

``` java 

public class CustomApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        new EventBusExecutorServiceConfigurer().configureEventBus();

        // other initialization
    }
}

```

[EventBus]: https://github.com/greenrobot/EventBus
[ExecutorsService]: http://developer.android.com/reference/java/util/concurrent/Executors.html#newCachedThreadPool()
[THREAD_POOL_EXECUTOR]: http://developer.android.com/reference/android/os/AsyncTask.html#THREAD_POOL_EXECUTOR
