

# **Bloodstream**

## **Summary**

- #### A prototype Apple Watch & iOS application that monitors heart rate and reflex time to determine if a user is intoxicated.

- #### When a user is intoxicated, their heart rate slowly dips below average, and their reflex time is considerably reduced.

- #### This project still has a long way to go, but I've made significant progress. I've detailed several pieces below.

## **Accelerometer & Gyrometer**

- #### In order to measure reflex time, I tapped into the sensor data from the Apple Watch. 

- #### The goal of the code below is to record how long it takes for the user to raise the watch and look at a notification, etc.

- #### The code below shows the basic setup of a MotionManger class; which holds the necessary functions to process real-time sensor data. I've abstracted many helper functions and left the important stuff in. 

- #### The app samples information at a frequency of 50 Hz!

```swift
class MotionManager{
    let motionManager = CMMotionManager()
    let queue = OperationQueue()
    let wristLocationIsLeft = WKInterfaceDevice.current().wristLocation == .left
    
    // The app is using 50hz data and the buffer is going to hold 1s worth of data.
    let sampleInterval = 1.0 / 50
    let rateAlongGravityBuffer = RunningBuffer(size: 50)
    
    var gravityStr; var rotationRateStr; var userAccelStr; var attitudeStr;
    var timer = Timer(); var counter = 0.0
    
    init() {...} // Initializes class variables
    func startUpdates() {...} // Starts recording data from sensors
    func stopUpdates() {...} // Stops recording data from sensors
    func processDeviceMotion() {...} // Stores recorded data and formats it nicely
    func updateMetricsDelegate() {...} // Sends recorded data back to main swift file
```

- #### The function below recieves data from the sensor and formats it nicely.

```swift
func processDeviceMotion(_ d: CMDeviceMotion) {
        gravityStr = String(format: customG, d.gravity.x,d.gravity.y, d.gravity.z)
        userAccelStr = String(format: customU, d.Accel.x, d.Accel.y, d.Accel.z)
        rotationRateStr = String(format: customR, d.rotRate.x, d.rotRate.y, d.rotRate.z)
        attitudeStr = String(format: customA, d.atti.roll, d.atti.pitch, d.atti.yaw)
}
```

- #### Lastly, the main controller of the watch recieves the data and tells us how long it took for the user to look at the watch.

- #### Right now, it detects when the user puts the watch down by his side, and immediately sends a dummy notification. It then records the amount of time it takes for the user to raise his wrist.

```swift
// Function below is called when there are significant changes in motion
func didUpdateMotion(...) {
  DispatchQueue.main.async {
    let data = gravityStr.components(separatedBy: " ")
    self.updateLabels()

    if(data[(data.firstIndex(of: "Z") ?? 0) + 5] == "-0.3"){
      sendNotification()
      if(self.wristRaised){
        self.wristRaised = false
        self.workoutManager.stopWorkout()
        self.timerTest.invalidate()
        self.ticker = false
      }
    }
    if(abc[(abc.firstIndex(of: "Z") ?? 0) + 5] == "-1.0"){
      self.wristRaised = true
      self.timerTest = Timer.scheduledTimer(
        timeInterval: 0.1, 
        target: self, 
        selector: #selector(self.UpdateTimer), 
        userInfo: nil, repeats: true
      )
    }
  }
}
// Sends a dummy notification
func sendNotification(...)
```

## **Heart Rate Data**

- #### The app currently records a breadth of heart rate data, including resting heart rate, running heart rate, and sleeping heart rate.

- #### Doing this prevents our app from alerting the user whenever their heart rate is low. The user could simply be sleeping!

- #### The code below uses WatchKit to record heart rate with minimal code. It creates a query that constantly grabs data from the HealthKit store. 

```swift
func fetchLatestHeartRateSample(completionHandler: @escaping (_ sample: HKQuantitySample) -> Void) {
  			// We want to record heart rate data
        guard let sampleType = HKObjectType.quantityType(forIdentifier:heartRate)
       	// Create data query
        let query = createQuery(...)
        healthStore.execute(query)
}
/* Query HealthKit for today's heart rate info*/
func getDayHR(...)
/* Query HealthKit for today's resting heart rate data */
func getDayRHR(...)
/* Query HealthKit for today's workout data */
func getDayWorkouts(...)
/* Query HealthKit for today's steps data */
func getDaySteps(...)
```

## **Front End**

- #### With some functionality for heart rate & reflex time built, I built a front end to display the values. 

- #### The iPhone app seamlessly syncs up with the real time data on the watch. 

- #### Click play on the videos below to watch the app in action!

