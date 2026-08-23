+++
date = '2026-08-23T14:12:31+05:30'
draft = false
title = 'Dispatch Queues'
tags = ['swift', 'ios']
categories = ['general']
summary = 'Grand Central Dispatch explained'
+++


Dispatch Queues are objects which manages execution of tasks serially or concurrently in main or background thread in the app. It allow developers to write code which takes advantage of mutli-core hardware without the overhead of manually management of available threads and resources.


### How Dispatch Queues works
Dispatch uses queue based approach. The tasks submitted to the queue are executed in FIFO(First In First Out) order. 

* The tasks are submtited to the queue
* The tasks are picked up in FIFO order and executed by a pool of thread 
* The thread allocation for the task is done by the system the developers need not need to manage the threads and resources manually 

### Serial and Concurrent execution

**Serial** - Serial execution means only one task is executed in given time. The next task will be executed only when current tasks completes its execution. In case of Dispatch Queues the next task in the queue will be picked and executed by the thread pool only when currently executing task is completed.

**Concurrent** - Concurrent execution means two or more tasks being executed simultaneously. The tasks can be executed paralelly or by context-switching between them based on available system resources. In case of Dispatch Queue multiple tasks will be picked up in FIFO and executed concurrently or parallely. 

### Synchronous and Asychronous execution 
Synchoronous - A synchronous task will block the current thread in which the task is called. The thread will continue its execution only when the dispatched synchronous task completes its execution. You should not call synchronous task on main thread because it will cause dead lock blocking the main thread.

Asynchronous - An asynchronous task will not block the current thread in which the task is called. The task will be dispatched and executed in a seperate thread so that the current calling thread can contniue its execution. When the dispatched task completes its execution it can trigger a callback. 

### Types of Dispatch Queues

* Main Queue
* Global Concurrent Queues
* Custom Queues and Target Queue

## Main Queue 
* Main Queue executes the dispatched task in main thread 
* It is serial in nature 
* It can execute the dispatched task either synchronously or asynchronously but synchronous is not recommended as it will cause dead lock.
* UIKit runs on main thread, all the UI related opearations should be done on main thread.
* It is most commonly used when updating the UI after fetching a data from a web service asynchronously

```swift
  DispatchQueue.main.async {
    //UI update
    self.titleLabel.text = "Test"  
  }
```

## Global Concurrent Queues
* These are system provided queeus which executes the tasks concurrently.
* The tasks submitted to these queues are picked up and executed based on quality of service. 
* The type of QOS(Quality Of Service) determines the execution priority and resource allocation for the task.

### QOS(Quality Of Service) types and order 

* User interactive - It says the system to run this task ASAP. The task can be executed in main thread or background thread. It is used for animations, immediate UI updates etc. It has highest priority 

* User initiaed - This task is triggered by the user, and the user is actively waiting for the result. For example, the user taps a PDF file and waits for it to load. It tells the system to execute this task with higher priority than .utility/.background work — but still lower priority than .userInteractive
  
* Utility - It is used for tasks which are may or may not visible to user. Eg Downloading a file in background.
  
* Background - It is used for long running task not visible to user. For example cleaning up cache. It has lowest priority. 

### Custom Queues 
In addition to built-in queues we can also create our own custom Dispatch Queues by creating an object of type DispatchQueue
* The custom queues can be serial or concurrent in execution. By default it is serial in nature.
* Even though it is a custom queue under the hood the tasks submittd to custom queues are funneled to a target queue.

Its initializer of DispatchQueue takes the following parameters
  
```swift
    convenience init(
    label: String,
    qos: DispatchQoS = .unspecified,
    attributes: DispatchQueue.Attributes = [],
    autoreleaseFrequency: DispatchQueue.AutoreleaseFrequency = .inherit,
    target: DispatchQueue? = nil
)
```

* ```label``` - It represents the name of the dispatch queue.
* ```qos``` - It represents the Quality Of Service.
* ```attributes``` - It can be ```concurrent``` or ```initiallyInactive```. If no value is specified then it is serial by default.
* ```autoreleaseFrequency``` - It determines the frequency in which the dispatch queue releases the objects. 
* ```target``` - It determines the queue to which the tasks in custom queues will be funneled to.

```swift
  
let customQueue = DispatchQueue(label: "CUSTOMQUEUE", qos: .userInteractive, attributes: .concurrent)
 customQueue.async{
        print("Two")
    }
 customQueue.async{
        print("Three")
    }
 customQueue.async{
        print("Four")
    }
customQueue.async{
        print("Five")
    }
```
Output
```
Two
Four
Five
Three
```
Here the queue is async in nature so the output order varies each type

### Target Queues
*Target Queues are the queues to which the task submitted to the custom queue will be funneled to. If no custom queue is specified then uderlying system queue will be used. We can also specify a custom queue to act as target queue

When a custom queue is used as a target queue it has the following properties 

* The QOS and Attribute of custom target queue overrides the QOS and Attribute of custom queue. 

#### For example 
```swift
let targetQueue = DispatchQueue(label:"TARGETQUEUE",qos:.utility)
  
let customQueue = DispatchQueue(label: "CUSTOMQUEUE", qos: .userInteractive, attributes: .concurrent,target: targetQueue)
 customQueue.async{
       
        print("Two")
    }
 customQueue.async{
    
        print("Three")
    }
 customQueue.async{
    
        print("Four")
    }
customQueue.async{
    
        print("Five")
    }
```
### Output
```
Two
Three
Four
Five
```
Here the even though the custom queue attribute is  concurrent in nature since the target queue is serial by default without any attribute the tasks will be executed in serial order one after the another. And also note that even though the QOS of custom queue is ```.userInteractive``` the QOS of target queue overrides it. So the QOS will be ```.utility``` 

### AutoRelease Frequency 
It determins the frequency in which the objects present inside the task block will be deallocated. It can take the following values

* ```inherit```- The queue inherits the auto-release ferquency of the target queue. This is the default behaviour
* ```workItem```- The objects are deallocated from the memory for current block before executing the next block
* ```never```- The objects will never be deallocated. This is used when we want manual control of the memory allocation. 


## Practice Questions

1. Predict the output order
```swift
let serialQueue = DispatchQueue(label: "com.example.serial")

serialQueue.async { print("1") }
serialQueue.async { print("2") }
serialQueue.async { print("3") }
```

 This is a custom queue which is serial in nature. <br>
 So the task will be executed on after another <br>
 Hence the output is 1,2,3 <br>


2. Predict the output order

```swift
let concurrentQueue = DispatchQueue(label: "com.example.concurrent", attributes: .concurrent)

concurrentQueue.async { print("A") }
concurrentQueue.async { print("B") }
concurrentQueue.async { print("C") }
```

 This is a custom queue which is concurrent in nature. <br>
 So the task blocks will concurrently  <br>
 Hence the output order varies each time. <br>


3. What prints first, "End" or "Async task"?

```swift
    print("Start")
    DispatchQueue.global().async {
        print("Async task")
    }
print("End")
```


 This is a custom queue which is concurrent and async in nature. <br>
 So the task blocks will asynchrnously without blocking the calling thread<br>
 Hence the output  End will be printed first before Async Task <br>



4. Will this cause dead lock

```swift
let queue = DispatchQueue(label: "com.example.serial")

queue.async {
    print("Task A starts")
    queue.sync {
        print("Task B")
    }
    print("Task A ends")
}
```

 Yes this will cause dead lock <br>
 The queue is serial in nature<br>
 Task A is async task.<br>
 Inside Task A we call another Sync Task which will block the calling thread in this case Task A. <br>
 Since the queue is serial in nature next task can be executed only when current task is completed. Since the current task(Task A) is blocked the next task will never be executed and it will cause dead lock








