# Getting started with the Combine framework in Swift
https://www.avanderlee.com/swift/combine/

## The Foundation framework and Combine

- A URLSessionTask Publisher that publishes the data response or request error
- Operators for easy JSON decoding
- **A Publisher for a specific Notification.Name that publishes the notification**

```swift 
import Combine

extension Notification.Name {
    static let newBlogPost = Notification.Name("new_blog_post")
}

struct BlogPost {
    let title: String
    let url: URL
}

// NotificationCenter.Publisher: A publisher that emits elements when broadcasting notifications.
let blogPostPublisher = NotificationCenter.Publisher(center: .default, name: .newBlogPost, object: nil)
    .map { (notification) -> String? in
        // The text property of the label requires to receive a String? value while the stream publishes a Notification.
        return (notification.object as? BlogPost)?.title ?? ""
    }

let lastPostLabel = UILabel()

// Subscribers.Assign: A simple subscriber that assigns received elements to a property indicated by a key path.
let lastPostLabelSubscriber = Subscribers.Assign(object: lastPostLabel, keyPath: \.text)

blogPostPublisher.subscribe(lastPostLabelSubscriber)

let blogPost = BlogPost(title: "Getting started with the Combine framework in Swift", url: URL(string: "https://www.avanderlee.com/swift/combine/")!)
NotificationCenter.default.post(name: .newBlogPost, object: blogPost)
print("Last post is: \(lastPostLabel.text!)")
// Last post is: Getting started with the Combine framework in Swift
```

## The rules of a subscription

- A subscriber can only have one subscription
- Zero or more values can be published
- **At most** one completion will be called
  - Subscriptions can come with completion, but not always. 
  - The abobe Notification example is such a Publisher which will never complete. 
  - The URLSessionTask Publisher will complete either with the data response or the request error. Fact is that whenever an error is thrown on a stream, the subscription is dismissed. Even if the stream allows multiple values to pass through.

