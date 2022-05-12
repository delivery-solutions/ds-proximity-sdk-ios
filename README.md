# Official Proximity SDK for iOS

<p align="center" width="100%">
    <img src="https://raw.githubusercontent.com/delivery-solutions/ds-proximity-sdk-ios/master/ProximitySDK_Banner_iOS.png">
</p>

&nbsp;
Delivery Solutions Proximity Tracking SDK for iOS

## Links
 - [Contact Us](https://www.deliverysolutions.co/contact-us)
 - [How to get YOUR_LICENSE_KEY](#faq)

## Features

- Efficient Vehicle/Order tracking
- Optimised battery usage
- Geofencing capabilities
- Alerts and Customer Notifications
- Integrates with DS services to provide an end to end solution for Curbside, and In-Store Pickups


## Installation

### Manually 

- Download the framework zip from Github releases section and unzip it.
- Drag and drop the framework file to your Xcode project
- Select the main project from left pane navigator -> Click on Target -> Click on General -> Search for framework, libraries, enbedded content
- You will find the DSProximity.framwork -> Select the Embed & Sign from dropdown.
- Framework successfully added, Now clean and build the project. Follow the setup guide section from here.

### Through Cocoapod 
DSProximity is available through [CocoaPods](https://cocoapods.org). To install it, you would require to request an access to the source repository, once done, simply add the following line to your Podfile:

```ruby

source URL ['https://deliverysolutions@bitbucket.org/deliverysolutions/proximity-sdk-ios-spec.git']

target 'PROJECTNAME' do

    pod 'DSProximity'
    
end
```
Open your favorite Terminal and run these commands.

```sh
cd ds-proximity 
pod install
```
**Tested with `pod --version`: `1.11.2`**: <br/>
[CocoaPods](https://cocoapods.org) is a dependency manager for Cocoa projects. You can install it with the following command:

**You should open the {Project}.xcworkspace instead of the {Project}.xcodeproj after you installed anything from CocoaPods.**

## Setup Guides
By downloading any content you agree to abide by the terms of the license
> Supported Swift 5
> Supports iOS 14+ devices

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

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
       
        var orgID = " YOUR_PROXIMITY_SDK_ORG_ID"
        var deviceID = " YOUR_DEVICE_ID"
        var isTestMode = TRUE/FALSE [SANDBOX -> TRUE, PRODUCTION -> FALSE]
        
         DSProximityService.instance.configure(testMode: isTestMode, orgId: orgID,
            deviceId: deviceID) { result in
            switch result {
            case .success(let response):
                print("SDK Initialized: \(response)")
            case .failure(let error):
                print("SDK - Failed to initialize: \(error.localizedDescription)")
            }
        }
        
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
    
    func orderUpdated() {
        view.makeToast("Order Updated successfully")
    }
    
    func orderFailed() {
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
    func locationUpdated() {
        view.makeToast("Location updated successfully")
    }
    
    func locationfailed() {
        view.makeToast("Location failed to update")
    }
}
```
## FAQ

#### How can I get the licence key / accessToken?

Please raise a ticket with [DS support](https://www.deliverysolutions.co/contact-us) with details about the usage and the team shall provide the required keys.

## License

Copyright (c) 2022 by [Delivery Solutions](https://www.deliverysolutions.co). All Rights Reserved. Usage of this library implies agreement to abide by the license terms. Please refer to our [Terms of Service](https://www.deliverysolutions.co/terms-of-service).
