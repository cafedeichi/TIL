### NotificationCenter

#### 〜iOS12

```swift
    let center = NotificationCenter.default
    let notification = Notification.Name("MyNotification")

    let observer = center.addObserver(forName: notification, object: nil, queue: nil) { _ in
        print("Notification recevied")
    }

    center.post(name: notification, object: nil)
    center.removeObserver(observer)
```

#### iOS13〜

```swift
    let center = NotificationCenter.default
    let notification = Notification.Name("MyNotification")
    
    let publisher = center.publisher(for: notification, object: nil)
        
    let subscription = publisher.sink { _ in
        print("Notification recevied")
    }
    center.post(name: notification, object: nil)
    subscription.cancel()
```

### Publiseher

```swift
import Combine

class StringSubscriber: Subscriber {
    
    func receive(subscription: Subscription) {
        print("Received Subscription")
        subscription.request(.max(3)) // backpressure
    }
    
    func receive(_ input: String) -> Subscribers.Demand {
        print("Received Value", input)
        return .unlimited   // update backpressure from 3 to unlimited
        // return .none // receive(completion:) not called if Subscribers.Demand is .none
    }
    
    func receive(completion: Subscribers.Completion<Never>) {
        print("Completed")  // 
    }
    
    
    typealias Input = String
    typealias Failure = Never
    
}

let publisher = ["A","B","C","D","E","F","G","H","I","J","K"].publisher
let subscriber = StringSubscriber()
publisher.subscribe(subscriber)
```

```swift
import Combine

enum MyError: Error {
    case subscriberError
}

class StringSubscriber: Subscriber {
    
    func receive(subscription: Subscription) {
        subscription.request(.max(2))
    }
    
    func receive(_ input: String) -> Subscribers.Demand {
        print(input)
        return .max(1)  // update back pressure
    }
    
    func receive(completion: Subscribers.Completion<MyError>) {
        print("Completion")
    }
    
    
    typealias Input = String
    typealias Failure = MyError    
}

let subscriber = StringSubscriber()

let publihser = PassthroughSubject<String, MyError>()

publihser.subscribe(subscriber)

let subscription = publihser.sink(receiveCompletion: { (completion) in
    print("Received Completion from sink")
}) { value in    
    print("Received Value from sink")
}

// Both sink(receiveCompletion:) and receive(subscription:) are called. 
publihser.send("A")
publihser.send("B")

subscription.cancel()

// receive(subscription:) is called regardless of subscription.cancel().
publihser.send("C")
publihser.send("D")

```
