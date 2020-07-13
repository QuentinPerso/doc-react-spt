# React-Native SPTProximityKit plugin 

- [1. Change Log](#changelog)
- [2. Installation](#installation)
- [3. Project configuration](#projectConfiguration)
- [4. SDK integration](#integration)
- [5. Requesting location authorization](#location)
- [6. Consent management](#cmp)

If you have any problem regarding SDK integration or need more informations email us at [support@singlespot.com](mailto:support@singlespot.com).

<a id="changelog"></a>
## 1. Change Log

* **v1.0.0** :
    - Brand new React Native plugin  

<a id="installation"></a>
## 2. SDK Instalation

From your project folder: 

#### yarn

```bash
$ yarn add git+https://USERNAME:PASSWORD@sdk.singlespot.com/react-native/SPTProximityKit-Plugin.git -- save
```

#### npm

```bash
$ npm install git+https://USERNAME:PASSWORD@sdk.singlespot.com/react-native/SPTProximityKit-Plugin.git -- save
```

⚠️ Replace USERNAME and PASSWORD by your own values found in Singlespot dashboard ⚠️

<a id="projectConfiguration"></a>
## 3. Project configuration

### 3.2 iOS configuration

#### 3.2.1 In the iOS project add to your podfile : 

```
pod 'SPTProximityKit', :git => 'https://USERNAME:PASSWORD@sdk.singlespot.com/cocoapods/SPTProximityKit.git'
```
⚠️ Replace USERNAME and PASSWORD by your own values found in Singlespot dashboard ⚠️

#### 3.2.2 Run `$ pod install` from iOS folder : 

```bash
$ cd ios 
$ pod install
```

#### 3.2.3 Add NSLocation...Description to info.plist file

To authorize background location, you must add `NSLocationWhenInUseUsageDescription`, `NSLocationAlwaysUsageDescription` & `NSLocationAlwaysAndWhenInUseUsageDescription` to your `info.plist` file

<p align="center">
    <img src="https://s3-eu-west-1.amazonaws.com/spt-static-assets/images/sdk/ios-info-plist.png"/>
</p>

The `value` field contains your custom text under Apple standart message in the Apple localisation authorization standart alert.

<p align="center">
<img src="https://s3-eu-west-1.amazonaws.com/spt-static-assets/images/sdk/ios-alert-box.png" width="280px" height="248px"/></p>




### 3.2 Android configuration


In your android folder's `build.gradle` (project level) add :

```
allprojects {
    repositories {

        <existing repo...>

        // ADD THIS =>
        maven {
            url "https://sdk.singlespot.com/artifactory/SPTProximityKit"
            credentials {
                username = USERNAME
                password = PASSWORD
            }
        }
    }
}
```
⚠️ Replace USERNAME and PASSWORD by your own values found in Singlespot dashboard ⚠️


And in your `AndroidManifest.xml` add your `API_KEY` and `API_SECRET` inside '<application \>' as follow : 

```xml
<meta-data
    android:name="com.sptproximitykit.ApiKey"
    android:value="YOUR_API_KEY" />

<meta-data
    android:name="com.sptproximitykit.ApiSecret"
    android:value="YOUR_API_SECRET" />
```
⚠️ Replace YOUR_API_KEY and YOUR_API_SECRET by your own values found in Singlespot dashboard ⚠️


## 4. SDK integration

### 4.1. initialisation

Add the following code in `App.js` (ATTENTION: please do not forget to replace YOUR\_API\_KEY and YOUR\_API\_SECRET by your own values):

```js
import SPTProximityKitPlugin from 'react-native-spt-proximity-kit';

componentDidMount() {

    var locationAskMode = 'sptLocationRequestModeServerBased';
    var cmpMode ='sptCmpModeAtLaunch';

    SPTProximityKitPlugin.setLocationMode(locationAskMode);
    SPTProximityKitPlugin.setCMPMode(cmpMode);
    SPTProximityKitPlugin.initWithApiKey('YOUR_API_KEY', 'YOUR_API_SECRET');

    this.updateConsents();
} 

```
⚠️ Replace YOUR_API_KEY and YOUR_API_SECRET by your own values found in Singlespot dashboard ⚠️

`locationRequestmode` sets the location requesting mode and may take the following values (see [5. Requesting location authorizationt](#location)):
* `'sptLocationRequestModeNeverAsk'`: default mode, use this mode if you wish to handle the location authorization request by yourself. (⚠️ SPTProximityKit ***needs*** location `Always` !)
* `'sptLocationRequestModeServerBased'`: our server manages the whole configuration for you and automatically requests location authorizations to the user (see [5.1 Location Mode `ServerBased`](#locationserverbased)).
* `'sptLocationRequestModeServerOnDemand'`: use this mode if you wish to indicate to the SDK when your application should make the location authorization request (see [5.1 Location Mode `OnDemand `](#locationondemand)).

`cmpMode` sets the CMP displaying mode and may take the following values (see [3. User consent management](#cmp)):
* `'sptCmpModeNotCMP'`: default mode, not using the SDK as a CMP. A valid CMP is still **mandatory** for any app collecting location data (see [3.5. Integrating with a 3rd party CMP](#3rdpartycmp)).
* `'sptCmpModeAtLaunch'`: mode to display Singlespot CMP on the very first launch of your app and retrying according to the dashboard configuration (see [3.1.1 CMP mode `atLaunch`](#cmpatlaunch)).
* `'sptCmpModeOnDemand'`: mode to display Singlespot CMP and retrying according to the dashboard configuration (see [3.1.2 CMP mode `onDemand `](#cmpondemand)).

Various callbacks can be set if actions are needed during CMP lifecycle . ( For details, see [3.4. Callbacks](#callbacks))

<a id="location"></a>
## 5. Requesting location authorization

`Always` location authorization is necessary for SPTProximityKit SDK. Thus, as location may not be the core of your business (but is ours), you are offered different ways of requesting location authorization, that can be remotely configured via Singlespot dashboard.

<a id="locationserverbased"></a>
### 5.1. Mode `ServerBased`

In this mode, a location request is dislayed on app entering foreground if at least of one the dashboard conditions is fullfilled (minimum number of launches or minimum number of days). This is not the best way to request location, but if there is no point in your app that seems better than launch to do it, that might by the right solution for you (you can customize this behaviour from the **Configuration** section of the **SDK** menu on the left navigation panel).

<a id="locationondemand"></a>
### 5.2. Mode `OnDemand`

With this method, you decide when to display the first location request. The potential following requests (if Always authorization is requested afterwards, or any potential retry) behave as configured in the dashboard, on app entering foreground (you can customize this behaviour from the **Configuration** section of the **SDK** menu on the left navigation panel).

Just call the following methods when appropriate : 

```js
SPTProximityKitPlugin.requestLocationAuthorizations();
```

### 5.3. Mode `NeverAsk`

Use this mode if you want to handle the location request yourself. Just make sure you request Always location!


<a id="cmp"></a>
## 6. Consent management

<a id="initialconsentrequest"></a>
### 6.1. Initial consent request
If Singlespot SDK is used as CMP (`cmpMode` to `atLaunch` or `onDemand`), a webView is provided listing:

- all the purposes required by your application, with switches;
- all the partners in the application that need a consent.

The list of purposes and vendors will be configurable from the Singlespot dashboard.

<a id="cmpatlaunch"></a>
#### 6.1.1 CMP mode `atLaunch`

If the value `kSPTCMPModeAtLaunch` is set in the SDK initialisation method for the parameter `cmpMode`, then the CMP will be displayed on the next launch of the application.

<a id="cmpondemand"></a>
#### 6.1.2 CMP mode `onDemand`

If the value `kSPTCMPModeOnDemand` is set in the SDK initialisation method for the parameter `cmpMode`, you can trigger the display of the CMP with the following method:

```js
SPTProximityKitPlugin.startCMP();
```

### 6.2. Consent management

In order to allow the users to manage their consent after the first request, you need to call the following SDK method from wherever is the most appropriate in your application:

```js
SPTProximityKitPlugin.openCMPSettings();
```

Doing so opens a 'settings' webView that will allow to switch on or off purposes and to see all the vendors.

### 6.3. Accessing consents

The Singlespot CMP follows the IAB Framework, and therefore grant access to the IAB consent string via the following method:

```js
var iabConsentString = await SPTProximityKitPlugin.iabConsentString("");
```

Consents status for a specific purpose or vendor can also be accessed via the following methods, using the id provided in the vendorlist ([https://vendorlist.consensu.org/vendorlist.json]):

```js
var iabVendorConsent  = await SPTProximityKitPlugin.isConsentGivenForIABVendor(<int vendorIABid>);
var iabPurposeConsent = await SPTProximityKitPlugin.isConsentGivenForIABPurpose(<int purposeIABid>);
```


If you want to get the consent status for a custom purpose or vendor, you can use the following methods with the corresponding key that has been provided by our team at CMP setup:

```js
var customVendorConsent  = await SPTProximityKitPlugin.isConsentGivenForCustomVendor(<String vendorIABid>);
var customPurposeConsent = await SPTProximityKitPlugin.isConsentGivenForCustomPurpose(<String purposeIABid>);
```

Custom vendors and purposes consents are also stored in the `NSUserDefaults` and `SharedPreferences` under the same keys, and can therefore be accessed by any 3rd party SDK with the right key.

Aliases have been defined to get the consent value for the two non-IAB purposes that are required by Singlespot (as mentionned in 6.5 [below](#3rdpartycmp)):

```js
var geoDataConsent = await SPTProximityKitPlugin.geoDataConsent("");
var geoMediaConsent = await SPTProximityKitPlugin.geoMediaConsent("");
```

<a id="callbacks"></a>
### 6.4. Callbacks

If you need callbacks for the various events related to the CMP, you can also set some `NativeEventEmitter` as follow:

```js
import { NativeEventEmitter} from 'react-native';


const sptProximityKitEmitter = new NativeEventEmitter(SPTProximityKitPlugin);

const consentChangedSubscription = sptProximityKitEmitter.addListener( 'EventConsentChanged', () => { ... });
const cmpPresentedSubscription = sptProximityKitEmitter.addListener( 'EventCmpPresented', () => { ... });
const cmpSettingsPresentedSubscription = sptProximityKitEmitter.addListener( 'EventCmpSettingsPresented', () => { ... });
const cmpsettingsDismissedSubscription = sptProximityKitEmitter.addListener( 'EventCmpSettingsDismissed', () => { ... });
const cmpcompletionSubscription = sptProximityKitEmitter.addListener( 'EventCmpCompletion', (completionObject) =>  { 
    const cmpDisplayed =  completionObject.cmpDisplayed;
    const error = completionObject.error;

    });
```

* `'EventConsentChanged'` is triggered each time consents are changed via the Singlespot CMP. Use this to update other partners consents if needed;
* `'EventCmpPresented'` is triggered when the CMP webView is presented on SDK initialization;
* `'EventCmpSettingsPresented'` is triggered when the CMP settings webView is presented;
* `'EventCmpSettingsDismissed'` is triggered when the CMP settings webView is dismissed;
* `'EventCmpCompletion'` is triggered when the CMP initial process is completed, whether or not a webView has been displayed ( `result` is an object containing the parameters `cmpDisplayed` and  `error`; argument `cmpDisplayed` will be `true` if a webView was presented, `false` otherwise). If there is no error, `error` will be `null`. You can resume your application normal execution at this point if needed.


<a id="3rdpartycmp"></a>
### 6.5. Integrating with a 3rd party CMP

If the value `false` is set in the SDK initialisation method, you need to use a 3rd party CMP. To be compatible with Singlespot SDK, it must get consent for the following two purposes (translation and description will be provided on demand):
- storage and access to location data for targeted advertising purposes;
- storage and access to location data for marketing studies purposes.

The SDK will read the vendor consent from the `IABConsent_ConsentString` directly, but as the above mentioned purposes are not part of the IAB framework, consent for these need to be handled differently.

You can set consent for these two purposes using the following methods:


```js
SPTProximityKitPlugin.setGeoMediaConsent(<bool>);
SPTProximityKitPlugin.setGeoDataConsent(<bool>);
```

You can also store them as `boolean` in the `NSUserDefaults`, like the `IABConsent_ConsentString`, under a key that must be provided in the **Consent** section of the Singlespot dashboard.

Finally if your CMP is not even IAB compatible you can directly set Singlespot vendor consent with:

```js
SPTProximityKitPlugin.setSinglespotConsent(<bool>);
```

A warning in Xcode console will be raised if an IAB consent string is present and thus overrode. Any subsequent setting of an IAB consent string will also override this value.


 
#### Please note

To the extent possible, please let the Singlespot SDK handle its consents checks and activate itself when sufficient consents are granted. Simply setting the consents as mentionned above will be enough to know when to be switched on or off.

If you have any question, feel free to contact us at [support@singlespot.com](mailto:support@singlespot.com).
