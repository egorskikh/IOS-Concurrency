# IOS-Concurrency

## Notes

```swift

// Class level variable

let queue = DispatchQueue(label: "ru.egorskikh.worker")

// Somewhere in your function

queue.async {

  // Call slow non-UI methods here

  DispatchQueue.main.async {

    // Update the UI here
    
  }  

}

```
