# Oral-B SDK

## Abstract

The Oral-B SDK provides authorized developers with access to Oral-B Bluetooth enabled toothbrushes and/or the Oral-B cloud.

Therefore the `OBTSDK` provides interfaces to:

  - **discover** Bluetooth enabled Oral-B toothbrushes (SmartSeries 6000 and higher) via Bluetooth Low Energy (BLE)
  - **connect** to Oral-B toothbrushes
  - receive **updates** on state changes of the connected toothbrush, such as brushing time, device state, ...
  - receive **brush sessions** from a user's **Oral-B Cloud** account

Moreover the `OBTSDK` will **automatically synchronize brush sessions** recorded with a connected toothbrush with the Oral-B Cloud service.

Given that a user granted access, you can pull all brush sessions of a user.

The Oral-B toothbrush is modeled into `OBTBrush`. Brush sessions are encapsulated in the `OBTBrushSession` model.


## Oral-B Developer Portal / How To Become An Authorized Third-Party Developer

To become a authorized third-party developer you have to [sign up](https://developer.oralb.com/) for a Oral-B third-party developer account. Registered third-party developers can then request an APP-ID and APP-KEY for his own applications.


## Add Permissions for Bluetooth and Internet To Your Applications Manifest
If your application wants to make use of the Bluetooth library and connect to Bluetooth enabled Oral-B toothbrushes, you have to add the BLUETOOTH and INTERNET permissions to the manifest:

```xml
    <uses-permission android:name="android.permission.BLUETOOTH"/>

	<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>

	<uses-permission android:name="android.permission.INTERNET"/>
	
	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
```
## Authentication

The Oral-B SDK differentiates between two tiers auf authentication: **Developer authentication** and **User authentication**.

If you just want to interact with the toothbrush, the Developer authentication is sufficient. However, if you want to access a users brush sessions you have to ask for permissions first.

**Please note**
> If you are building an app which is used to regularly record the users sessions the user would expect that you allow him to connect your app to the Cloud. Otherwise the sessions recorded while using your app might not be synced to the users cloud account.


### Developer Authentication

**Add APP-ID And APP-KEY To Your Applications Manifest**

To authorize your application, you have to add the APP-ID and APP-Key to the meta-data of your application in your manifest:

```xml
	<application android:label="@string/app_name" ...>
	
	  	...
	  	
	    <meta-data android:name="com.obt.sdk.ApplicationId" android:value="@string/obt_sdk_app_id"/>
	    
		<meta-data android:name="com.obt.sdk.ApplicationKey" android:value="@string/obt_sdk_app_key"/>
		
	    ...
	    
	</application>
```

Afterwards you have to invoke `OBTSDK.initialize(Context context)` to be able to use the SDK.


### User Authentication

To allow a user to (automatically) sync sessions recorded with your app to their Oral-B Cloud account and to be able to fetch all sessions recorded by a user, you have to ask the user to grant permissions to your application. This will be done using OAuth 2 and is handled by the SDK.

You have to invoke `OBTSDK.authorizeApplication(OBTAuthorizationListener listener)` to do so.


## Add Connection Activity To Your Applications Manifest
If your application wants to use the Bluetooth library and connect to Bluetooth enabled Oral-B Toothbrushes and/or wants to use the Oral-B Cloud, you have to add following activity to your application in your manifest:

```xml
	<activity android:name="com.obt.OBTActivity"
	
          android:orientation="portrait"
          
          android:theme="@android:style/Theme.Translucent.NoTitleBar"
          
          android:label="@string/app_name" />
```


## Quick Start

### Installation

**Manually with Android Studio and Gradle**

Create a repository from a basic directy using 'flatDir' in build.gradle in project root:

```groovy
    repositories {
        mavenCentral()
        flatDir {
            dirs 'libs'
        }
    }
```

Then reference the .aar as dependency in build.gradle of your module:

```groovy
    dependencies {
      compile(name:'libraryfilename', ext:'aar')
    }
```

### Initialization of Oral-B SDK

```java
	public class DemoApplication extends Application {

    	@Override
    	public void onCreate() {
    		super.onCreate();
    		try {
    		    //Call to initialize the OBTSDK
    			OBTSDK.initialize(this);
    		} catch (PackageManager.NameNotFoundException e) {
    			e.printStackTrace();
    		}
    	}
    }
```


### Start searching Oral-B toothbrushes and connect to one

To connect to a toothbrush, you have to follow these steps:

1. Init the SDK with your APP ID and APP KEY.
2. Add yourself as a listener to the OBTSDK
3. Listen to `onNearbyBrushesFoundOrUpdated(List<OBTBrush> nearbyBrushes)` and connect to a brush using `OBTSDK.connectToothbrush(OBTBrush brush)`
4. Listen to any of the relevant callbacks of the `OBTBrushListenerAdapter` or `OBTBrushListener`, e.g. `onBrushingTimeChanged(int brushingTime)` in case you want to display the brush time

```java
    private OBTBrushAdapter obtBrushAdapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.activity_bluetooth);

        // Creating a ListView to show nearby Oral-B Toothbrushes
        ListView nearbyOralBToothbrushList = (ListView) findViewById(R.id.found_brushes_list);

        // Instantiate a custom adapter to hold the Oral-B Toothbrushes and set it to the list
        obtBrushAdapter = new OBTBrushAdapter();
        nearbyOralBToothbrushList.setAdapter(obtBrushAdapter);

        // Set OnItemClickListener to the ListView
        nearbyOralBToothbrushList.setOnItemClickListener(new AdapterView.OnItemClickListener() {
                @Override
                public void onItemClick(AdapterView<?> adapterView, View view, int position, long id) {
                    // Connect to chosen Oral-B Toothbrush
                    OBTSDK.connectToothbrush(obtBrushAdapter.getItem(position));
                    }
                });
    }

    @Override
    public void onResume() {
        super.onResume();
        // Set this activity as OBTBrushListener
        OBTSDK.setOBTBrushListener(this);
    }

    @Override
    protected void onPause() {
        super.onPause();
        // Remove the OBTBrushListener
        OBTSDK.setOBTBrushListener(null);
    }

    @Override
    public void onNearbyBrushesFoundOrUpdated(List<OBTBrush> nearbyBrushes) {
        // Clear the custom adapter and set found Oral-B Toothbrushes as items
        obtBrushAdapter.clear();
        obtBrushAdapter.addAll(nearbyBrushes);
        obtBrushAdapter.notifyDataSetChanged();
    }

    @Override
    public void onBrushingTimeChanged(int brushingTime) {
        // Show brushing time in ui
    }
```

### Fetch a user's brush sessions

To retrieve brush sessions from a user, you have to follow these steps:

1. Init the SDK with your APP ID and APP KEY.
2. Ask the user to authorize your application using `OBTSDK.authorizeApplication(OBTAuthorizationListener listener)` _(user will be redirected to grant permissions)_
3. On successful authorization you can invoke `OBTSDK.fetchUserSessions(long from, OBTCloudListener callback)`



### Oral-B SDK Interface-Methods
The Oral-B SDK provides following functionality:


```java
	public static void initialize(@NonNull Context context) throws PackageManager.NameNotFoundException
```

This method will initialize the OBT SDK with your APP-ID and APP-KEY. Make sure to have added these values to your manifest!


```java
	public static void authorizeApplication()
```

This method needs to be called to let the user authorize your application to access the Oral-B Cloud


```java
	public static boolean isBluetoothAvailableAndEnabled()
```

This method checks whether Bluetooth Low Energy is available and enabled


```java
	public static void startScanning()
```

This method starts scanning for Bluetooth enabled Oral-B Toothbrushes


```java
	public static void stopScanning()
```

This method stops scanning for Bluetooth enabled Oral-B Toothbrushes


```java
	public static void connectToothbrush(OBTBrush brush)
```

This method connects to a specific Bluetooth enabled Oral-B Toothbrush


```java
	public static void disconnectToothbrush() 
```

This method disconnects a currently connected Bluetooth enabled Oral-B Toothbrush


```java
	public static boolean isConnected()
```

This method checks if there is any Bluetooth enabled Oral-B Toothbrush currently connected


```java
	public static boolean isScanning()
```

This method checks if the OBT SDK is currently scanning for Bluetooth enabled Oral-B Toothbrushes


```java
	public static OBTBrush getConnectedToothbrush()
```

This method retrieves currently connected Bluetooth enabled Oral-B Toothbrush


```java
	public static OBTBrush getPreferableToothbrush()
```

This method retrieves currently preferable Bluetooth enabled Oral-B Toothbrush


```java
	public static List<OBTBrush> getNearbyToothbrushes()
```

This method retrieves a list of all Bluetooth enabled Oral-B Toothbrushes nearby


## OBTBrushListener

This interface will receive callbacks from Oral-B Toothbrushes. It provides following methods:

```java
    void onNearbyBrushesFoundOrUpated(List<OBTBrush> nearbyBrushes);
```

This method will be called whenever the list of nearby Oral-B Toothbrushes changes


```java
    void onBluetoothError();
```

This method will be called when a Bluetooth error occurred


```java
    void onBrushDisconnected();
```

This method will be called when Oral-B Toothbrush disconnects


```java
    void onBrushConnected();
```

This method will be called when connection to a Oral-B Toothbrush was successful


```java
    void onBrushingTimeChanged(long brushingTime);
```

This method will be called when the reported brushing time of the connected Oral-B Toothbrush changes


```java
    void onBrushingModeChanged(int brushingMode);
```

This method will be called when the current brushing mode of the connected Oral-B Toothbrush changes


```java
    void onDeviceStateChanged(int deviceState);
```

This method will be called when the current device state of the connected Oral-B Toothbrush changes


```java
    void onRSSIChanged(int rssi);
```

This method will be called when the RSSI of the connected Oral-B Toothbrush changes


```java
    void onBatteryLevelChanged(float batteryLevel);
```

This method will be called when the battery level of the connected Oral-B Toothbrush changes


```java
    void onSectorChanged(int sector);
```

This method will be called when the reported sector of the connected Oral-B Toothbrush changes


```java
    void onPressureChanged(boolean isHighPressure);
```

This method will be called when the high pressure value of the connected Oral-B Toothbrush changes


```java
    void onAvailableBrushModesChanged(List<Integer> availableModes);
```

This method will be called when the available brush modes of the connected Oral-B Toothbrush changes


## OBTCloudListener

This interface will receive callbacks from Oral-B Cloud. It provides following methods:

```java
    void onSessionsLoaded(List<OBTSession> obtSessions);
```

This method will be called when sessions of the user were retrieved from the Oral-B Cloud


```java
    void onFailure(int code, String message);
```

This method will be called when there was an error during communication with the Oral-B Cloud


## OBTAuthorizationListener

This interface will receive callbacks from Oral-B Cloud authorization process. It provides following methods:

```java
    void onAuthorizationSuccess();
```

This method will be called when application was successful authorized by the user


```java
    void onAuthorizationFailed();
```

This method will be called when the authorization has failed


```java
    void onAuthorizationCanceled();
```

This method will be called when the user has canceled the authorization process