<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.aa.baggage.rampopslogmanager">

<!--    <uses-sdk android:minSdkVersion="19" />-->

    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.BLUETOOTH" />
    <uses-permission android:name="com.symbol.emdk.permission.EMDK"/>
    <uses-permission android:name="android.permission.REQUEST_IGNORE_BATTERY_OPTIMIZATIONS"/>
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <receiver android:name=".uploadworkmanager.StartLogUploadManagerReceiver">
            <intent-filter>
                <action android:name="com.aa.baggage.rampopslogmanager.uploadworkmanager.START_LOG_UPLOAD_MANAGER">
                </action>
            </intent-filter>
        </receiver>
        <receiver android:name=".uploadworkmanager.StopLogUploadManagerReceiver">
            <intent-filter>
                <action android:name="com.aa.baggage.rampopslogmanager.uploadworkmanager.STOP_LOG_UPLOAD_MANAGER">
                </action>
            </intent-filter>
        </receiver>
        <receiver android:name=".emdkmanager.ChangeBatteryOptimizationReceiver">
            <intent-filter>
                <action android:name="com.aa.baggage.rampopslogmanager.emdkmanager.CHANGE_BATTERY_OPTIMIAZATION">
                </action>
            </intent-filter>
        </receiver>
        <uses-library android:name="com.symbol.emdk"/>
    </application>

</manifest>
