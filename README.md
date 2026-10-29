# 7. Develop an Android Application Implementing Started and Bound Services

[← Back to Index](README.md)

## Subject: Services, Notifications and Broadcast Receivers

## Project Name
`ServiceDemo`

## Aim
To develop an Android application that demonstrates **Started Services** and **Bound Services** in Android.

## Introduction

An Android Service is a component that can perform operations without providing a user interface.

There are two service types demonstrated in this application:

- **Started Service:** A service started using `startService()`. It continues running until it is stopped or otherwise terminated by the system.
- **Bound Service:** A service that allows other components, such as activities, to connect to it, interact with it, and receive results. It generally remains bound while clients are connected.

This application uses buttons to start and stop a Started Service and connect to a Bound Service.

## 1. File Name: `activity_main.xml`

**File Path:** `app/src/main/res/layout/activity_main.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center"
    android:padding="20dp">

    <Button
        android:id="@+id/btnStart"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Start Service" />

    <Button
        android:id="@+id/btnStop"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Stop Service"
        android:layout_marginTop="15dp" />

    <Button
        android:id="@+id/btnBound"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Bind Service"
        android:layout_marginTop="15dp" />

</LinearLayout>
```

## 2. File Name: `MyStartedService.java`

**File Path:** `app/src/main/java/com/example/servicedemo/MyStartedService.java`

Create a Java class named `MyStartedService` inside your app's Java package.

```java
package com.example.servicedemo;

import android.app.Service;
import android.content.Intent;
import android.os.IBinder;
import android.widget.Toast;

public class MyStartedService extends Service {

    @Override
    public int onStartCommand(
            Intent intent,
            int flags,
            int startId) {

        Toast.makeText(
                this,
                "Started Service Running",
                Toast.LENGTH_SHORT
        ).show();

        return START_STICKY;
    }

    @Override
    public void onDestroy() {
        super.onDestroy();

        Toast.makeText(
                this,
                "Started Service Stopped",
                Toast.LENGTH_SHORT
        ).show();
    }

    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }
}
```

## 3. File Name: `MyBoundService.java`

**File Path:** `app/src/main/java/com/example/servicedemo/MyBoundService.java`

Create another Java class named `MyBoundService`.

```java
package com.example.servicedemo;

import android.app.Service;
import android.content.Intent;
import android.os.Binder;
import android.os.IBinder;

public class MyBoundService extends Service {

    private final IBinder binder = new LocalBinder();

    public class LocalBinder extends Binder {

        public MyBoundService getService() {
            return MyBoundService.this;
        }
    }

    @Override
    public IBinder onBind(Intent intent) {
        return binder;
    }

    public String getMessage() {
        return "Bound Service Connected";
    }
}
```

## 4. File Name: `MainActivity.java`

**File Path:** `app/src/main/java/com/example/servicedemo/MainActivity.java`

```java
package com.example.servicedemo;

import android.content.ComponentName;
import android.content.Intent;
import android.content.ServiceConnection;
import android.os.Bundle;
import android.os.IBinder;
import android.widget.Button;
import android.widget.Toast;

import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {

    Button btnStart, btnStop, btnBound;

    MyBoundService myBoundService;
    boolean isBound = false;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        btnStart = findViewById(R.id.btnStart);
        btnStop = findViewById(R.id.btnStop);
        btnBound = findViewById(R.id.btnBound);

        // Start Service
        btnStart.setOnClickListener(v -> {
            Intent intent = new Intent(
                    MainActivity.this,
                    MyStartedService.class
            );

            startService(intent);
        });

        // Stop Service
        btnStop.setOnClickListener(v -> {
            Intent intent = new Intent(
                    MainActivity.this,
                    MyStartedService.class
            );

            stopService(intent);
        });

        // Bind Service
        btnBound.setOnClickListener(v -> {
            if (!isBound) {
                Intent intent = new Intent(
                        MainActivity.this,
                        MyBoundService.class
                );

                bindService(
                        intent,
                        serviceConnection,
                        BIND_AUTO_CREATE
                );
            } else {
                Toast.makeText(
                        MainActivity.this,
                        myBoundService.getMessage(),
                        Toast.LENGTH_SHORT
                ).show();
            }
        });
    }

    // Service Connection
    private final ServiceConnection serviceConnection =
            new ServiceConnection() {

        @Override
        public void onServiceConnected(
                ComponentName name,
                IBinder service) {

            MyBoundService.LocalBinder binder =
                    (MyBoundService.LocalBinder) service;

            myBoundService = binder.getService();
            isBound = true;

            Toast.makeText(
                    MainActivity.this,
                    myBoundService.getMessage(),
                    Toast.LENGTH_SHORT
            ).show();
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            isBound = false;
            myBoundService = null;
        }
    };

    @Override
    protected void onDestroy() {
        if (isBound) {
            unbindService(serviceConnection);
            isBound = false;
            myBoundService = null;
        }

        super.onDestroy();
    }
}
```

## 5. File Name: `AndroidManifest.xml`

**File Path:** `app/src/main/AndroidManifest.xml`

Declare both services inside the existing `<application>` element:

```xml
<service
    android:name=".MyStartedService"
    android:exported="false" />

<service
    android:name=".MyBoundService"
    android:exported="false" />
```

**Important:** Add these declarations inside your existing `<application>` element, not outside it. Do not duplicate service declarations that Android Studio has already created. Keep your existing activity and launcher configuration unchanged.

## 6. Explanation

### Started Service

- The **Start Service** button calls `startService()` to start `MyStartedService`.
- The service's `onStartCommand()` method displays the Toast message `Started Service Running`.
- The **Stop Service** button calls `stopService()` to stop the started service.
- The `onDestroy()` method displays `Started Service Stopped` when the service is destroyed.
- `START_STICKY` requests that Android recreate the service after certain process terminations when possible. It does not guarantee that the service will restart.

### Bound Service

- The **Bind Service** button calls `bindService()` with `BIND_AUTO_CREATE`.
- Android creates the service if necessary and connects the activity to it.
- `LocalBinder` provides access to the `MyBoundService` instance.
- The `getMessage()` method returns `Bound Service Connected`.
- The activity unbinds from the service in `onDestroy()` to release the connection.

## 7. Expected Output

1. The application displays three buttons: **Start Service**, **Stop Service**, and **Bind Service**.
2. Clicking **Start Service** displays `Started Service Running`.
3. Clicking **Stop Service** stops the started service and displays `Started Service Stopped` when its destruction callback runs.
4. Clicking **Bind Service** connects to the Bound Service and displays `Bound Service Connected`.
5. If the Bound Service button is clicked again while connected, the same connection message is displayed again.

## 8. Result

Thus, an Android application demonstrating Started Services and Bound Services was developed successfully.

## 9. Important Notes

- Ensure that the package name `com.example.servicedemo` matches your project's package name.
- These services demonstrate basic service lifecycle callbacks and binding; they do not perform long-running background work.
- On modern Android versions, background-service restrictions apply. A service that must perform ongoing user-visible work may require a foreground service and a notification.
- The Started Service and Bound Service in this example are separate components.

## 10. Upload the Project to GitHub

1. Open your Android Studio project folder.
2. Create or update the `README.md` file with this content.
3. Save the file.
4. Open your GitHub repository.
5. Commit and push your changes.

### Git Commands

```bash
git add README.md
git commit -m "Add Started and Bound Services practical"
git push
```
