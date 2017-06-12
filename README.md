[![Build Status - Master](https://travis-ci.com/IBM-Swift/Health.svg?token=vW8PK5GRFLpKp5azyZhD&branch=master)](https://travis-ci.com/IBM-Swift/Health)
![macOS](https://img.shields.io/badge/os-macOS-green.svg?style=flat)
![Linux](https://img.shields.io/badge/os-linux-green.svg?style=flat)

# Health
The Health package provides a basic infrastructure that Swift applications can use for reporting their overall health status.

As an application developer, you create an instance of the `Health` class and then register one or more health checks. A health check can be either a closure that conforms to the `HealthCheckClosure` typealias or a class that conforms to the `HealthCheck` protocol. Once you have your health checks in place, you can ask your `Health` instance for its status.

## Swift version
The latest version of Health works with the `3.1.1` version of the Swift binaries. You can download this version of the Swift binaries by following this [link](https://swift.org/download/#snapshots).

## Usage:
To leverage the Health package in your Swift application, you should specify a dependency for it in your `Package.swift` file:

```swift
import PackageDescription

 let package = Package(
     name: "MyAwesomeSwiftProject",

     ...

     dependencies: [
         .Package(url: "https://github.com/IBM-Swift/Health.git", majorVersion: 0),

         ...

     ])
```

And this is how you create a `Health` instance and register your health checks:

```swift

...

let health = Health()

...

// Add custom checks
health.addCheck(check: MyCheck1())
health.addCheck(check: MyCheck2())
health.addCheck(check: myClosureCheck1)
health.addCheck(check: myClosureCheck1)

...

// Get current health status
let count = health.numberOfChecks
let status: Status = health.status
let state: State = status.state
let dictionary = status.toDictionary()
let simpleDictionary = status.toSimpleDictionary()

...

```

The contents of the simple dictionary simply contains a key-value pair that lets you know whether the application is UP or DOWN:

```
["status": "UP"]
```

The contents of the dictionary contains a key-value pair that lets you know whether the application is UP or DOWN plus additional details about the health checks that failed (if available):

```
["details": ["Cloudant health check.", "A health check closure reported status as DOWN."], "status": "DOWN"]
```

Swift applications can then use either dictionary, depending on the use case, to report the overall status of the application. For instance, an endpoint on the application could be defined that queries the `Health` object to get the overall status and then send it back to a client as a JSON payload.

## Caching
When you create an instance of the `Health` class, you can pass an optional argument (named `statusExpirationTime`) to its initializer as shown next:

```swift
let health = Health(statusExpirationTime: 30000)
```

The `statusExpirationTime` parameter specifies the number of milliseconds that a given instance of `Health` should cache its status before recomputing it. For instance, if the value assigned to `statusExpirationTime` is `30000` (as shown above), then 30 seconds must elapse before the `Health` instance computes its status again by querying each health check that has been registered. Please note that the default value for the `statusExpirationTime` parameter is `30000`.

## Health and a Kitura-based application
One common use case for this Swift package is to integrate it into a Kitura-based application, as shown next:

```swift

...

// Create main objects...
router = Router()

health = Health()

// Register health checks...
health.addCheck(check: Microserv1Check())
health.addCheck(check: microserv2check)

...

// Define /health endpoint that leverages Health
router.get("/health") { request, response, next in
  let status = health.status.toDictionary()
  // let status = health.status.toSimpleDictionary()
  try response.send(json: status).end()
}

```
