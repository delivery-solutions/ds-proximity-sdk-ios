# Official Proximity SDK for iOS

<p align="center" width="100%">
    <img width="33%" src="https://images.squarespace-cdn.com/content/v1/6073efa8ab5c5b78808fe878/1618336613080-9PJH9HELYSVDIBCPF7OU/logo.png?format=1500w">
</p>

&nbsp;
Delivery Solutions Proximity Tracking SDK for iOS

## Links
 - [Contact Us](https://www.deliverysolutions.co/contact-us)

## Features

- Efficient Vehicle/Order tracking
- Optimised battery usage
- Geofencing capabilities and alerts
- Integrates with DS services to provide an end to end solution for Curbside, and In-Store Pickups


## Installation
DSProximity is available through [CocoaPods](https://cocoapods.org). To install
it, simply add the following line to your Podfile:
```ruby
pod 'DSProximity'
```
Open your favorite Terminal and run these commands.

```sh
cd ds-proximity 
pod install
```
**Tested with `pod --version`: `1.8.4`**: <br/>
[CocoaPods](https://cocoapods.org) is a dependency manager for Cocoa projects. You can install it with the following command:

**You should open the {Project}.xcworkspace instead of the {Project}.xcodeproj after you installed anything from CocoaPods.**

## Setup Guides
> Supported Swift 5, iOS 14+
> [How to get YOUR_LICENSE_KEY](#faq)

## Usage
Clean and Re-build application.

### Location permissions

---

DSProximity uses CoreLocation, CoreMotion for its use. The app should ask for these required permissions.

 Add in the info.plist
> Privacy - Location Always and When In Use Usage Description 
> Privacy - Location When In Use Usage Description

or
```
<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
    <string>/* Message of your choice */</string>
<key>NSLocationWhenInUseUsageDescription</key>
    <string>/* Message of your choice */</string>

```

### Background Modes

DSProximity iOS SDK uses Background modes to notify the user when app is running in background

---
> Right click on info.plist open as view source , add the below xml to allow inventa sdk to use background modes

```
    <key>UIBackgroundModes</key>
    <array>
        <string>location</string>
    </array>
```

### Importing the DSProximity SDK:

```
import DSProximity
```

In AppDelegate didFinishLaunchingWithOptions function , add the following syntax as show below

```
    // Properties initialiazer
     let locationManager = CLLocationManager()

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
       
        var orgID = " YOUR_PROXIMITY_SDK_ORG_ID"
        var deviceID = " YOUR_DEVICE_ID"
        var isTestMode = TRUE/FALSE
        
         DSProximityService.instance.configure(mode: isTestMode, orgId: orgID,
            deviceId: deviceID) { result in
            switch result {
            case .success(let response):
                print("SDK Initialized: \(response)")
            case .failure(let error):
                print("SDK - Failed to initialize: \(error.localizedDescription)")
            }
        }
        
        UIApplication.shared.setMinimumBackgroundFetchInterval(
            UIApplication.backgroundFetchIntervalMinimum)
        
        locationManager.requestAlwaysAuthorization()
        locationManager.allowsBackgroundLocationUpdates = true
        locationManager.pausesLocationUpdatesAutomatically = false
        locationManager.desiredAccuracy = kCLLocationAccuracyBest
        
        return true
    }
    
    * Substitute your ORGANIZATION_ID, DEVICE_ID and STAGING/PRODUCTION_URL key in the value attribute.
  
```

### Start Tracking

```
 @IBAction func onTapStartTracking(_ sender: UIButton) {
        guard let orderId = order.text, !orderId.isEmpty else {
            view.makeToast("Please enter the order id to start tracking.")
            return
        }
        
        //Implementation
        DSProximityService.instance.startTracking(with: orderId) { [weak self] result in
            DispatchQueue.main.async {
                switch result {
                case .success(let store):
                    print("\(orderId) Order Tracking started.")
                    DSProximityService.instance.proximityDelegate = self
                case .failure(let error):
                    print("\(orderId) Order Tracking failed. Please check your orderID")
                    print("\(error.localizedDescription)")
                }
            }
        }
    }
```

### Updating the ETA (Manually)
Manual ETA Updating should be used for following -
- User has not provided access to location and wants to manually
- User even during the automatic flow, wants to manually notify the ETA (changes flow to manual)
An ISO 8601 timestamp is provided as the parameter indicating the new ETA.
```
    @IBAction func onTapEtaUpdate(_ sender: Any) {
        DSProximityService.instance.onETAUpdate(with: "<ORDER-ID>", and: "2021-10-06T11:13:10.038Z")
    }
```

### Arriving to the store
As in the order lifecycle, store can be informed of arrival by using markArrived function.
markArrived could be called on click of a button such as "I've Arrived"
Please note, tracking will continue even though the person has arrived.
```
     @IBAction func onTapArrived(_ sender: UIButton) {
        DSProximityService.instance.onArrived(with: "<ORDER-ID>")
    }
```

### Completing an Order
As in the order lifecycle, store can be informed of the completion of order by using markCompleted function.
Generally, this is to be called when the person has collected the order and wants to notify the store that to the store and stop location tracking. This should be called only once.
```
     @IBAction func onTapCompleted(_ sender: UIButton) {
        DSProximityService.instance.onCompleted(with: "<ORDER-ID>")
        DSProximityService.instance.proximityDelegate = nil
    }
```

### Track Order State
In some cases you would want to check the state of tracking i.e. whether the tracking is already going on or not. You can call the below method to get the tracking state:
```
    @IBAction func onTrackingDriverState(_ sender: UIBarButtonItem) {
        let trackingState = DSProximityService.instance.getTrackingState()
        print("isTracking: \(trackingState)")
    }
```

### Stop Tracking
By default the SDK stops tracking only when the order is marked as completed by calling markAsCompleted function. However, there might be cases when the user would want to stop the tracking manually. Call the below function to stop tracking:
```
 DSProximityService.instance.stopTracking()
 DSProximityService.instance.proximityDelegate = nil
```

### Set Vehicle Details
The following method is used to accept the vehicle details and the contact information of the user whose location is being tracked.
```
    DSProximityService.instance.updateVehicleDetails(
        vehicleMaker: "INPUT_VEHICLE_MAKE",
        vehicleType: "INPUT_VEHICLE_TYPE",
        vehicleNumber: "INPUT_VEHICLE_NUMBER",
        description: "INPUT_VEHICLE_DESCRIPTION",
        parkingSlot: "INPUT_VEHICLE_SLOT",
        isPickupBySomeoneElse: false/true,
        userName: "YOUR_USERNAME",
        contactNumber: "INPUT_CONTACT_NUMBER",
        orderId: "ORDER_ID"
    )
```

### Events
Helper Delegate methods are called when event has been triggered
```
extension ViewController: DSProximityServiceProtocol {
    func permissionDenied() {
        view.makeToast("Location Permission Denied")
    }
    
    func permissionAccepted() {
        view.makeToast("Location Permission allowed")
    }
    
    func updatedOrder() {
        view.makeToast("Order Updated successfully")
    }
    
    func updatedFailed() {
        view.makeToast("Order update failed. Please try again")
    }
    
    func onStarted() {
        view.makeToast("Started event updated successfully")
    }
    
    func onArrived() {
        view.makeToast("Arrived event updated successfully")
    }
    
    func onETAUpdate() {
        view.makeToast("ETA Updated successfully")
    }
    
    func onCompleted() {
        view.makeToast("Order completed successfully")
        startTracking.isEnabled = true
        startTracking.setTitle("Start Tracking", for: .normal)
    }
    func locationUpdate() {
        view.makeToast("Location updated successfully")
    }
}
```
## FAQ

#### How can I get the licence key / accessToken?

Please raise a ticket with [DS support](https://www.deliverysolutions.co/contact-us) with details about the usage and the team shall provide the required keys.
## License

Copyright (c) 2022 by [Delivery Solutions](https://www.deliverysolutions.co). All Rights Reserved. Usage of this library implies agreement to abide by the license terms. Please refer to our [Terms of Service](https://www.deliverysolutions.co/terms-of-service).
