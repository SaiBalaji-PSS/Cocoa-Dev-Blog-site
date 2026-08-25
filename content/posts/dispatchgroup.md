+++
date = '2026-08-25T21:38:40+05:30'
draft = false
title = 'Dispatchgroup'
tags = ['swift', 'ios']
categories = ['general']
summary = 'Dispatch group explained'
+++

#Dispatch Group

Dispatch group is an object which is used to monitor a group of task as a single unit. It is useful when we want to perform multiple asynchronous tasks concurrently and get notified when all of them are completed.

Most common approach is making multiple API calls in parallel and get a notificiation when all the API calls have been completed.

Dispatch Group object has following methods 

```enter()``` - manually notifies the group that a task has started. You call it before kicking off the async work, which increments the group's internal count.

```leave()``` - you call this manually, inside the completion handler, to signal that a task has finished. It decrements the group's internal count.

```wait()``` - Waits synchronously for the previously submitted work to finish. This will block the execution of program untill all the tasks in dispatch group is completed. We should not use it in main thread because the main thread will be blocked till all the tasks in dispatch group is completed.

```notify(queue: DispatchQueue, work: DispatchWorkItem)``` - It represents a block which will be called when all the task in dispatch group has been completed. It is asynchronous and will not block the current thread. This is prefered over wait()


Disptach Group Maintains an internal counter value. When enter() is called the counter gets incremented, when leave() is called the counter gets decremented. When the counter value becomes 0 the notify will be called.

###Note 
* If the number of enter() exceeds the number of leave() then notify() may never fire. 

* If number of leave() execeeds the number of enter() then it will crash




## Example

Here we make two API calls in parallel and wait for both of them to complete execution.

* The begining of the API call the task is added to dispatch group by calling enter(). When the API call is completed we call leave() to indicate the completion of the task in task group. 
* When the number of times enter() called matches with the number of times leave() called then all the tasks in Dispatch group  has  completed its execution. Which will call notifiy block. 

```swift
func fetchPosts(onCompletion:@escaping(Data?,Error?)->(Void)){
    URLSession.shared.dataTask(with: URLRequest(url: URL(string: "https://jsonplaceholder.typicode.com/comments")!)) { data , response , error in
        onCompletion(data,error)
        
    }.resume()
}

func fetchComments(onCompletion:@escaping(Data?,Error?)->(Void)){
    URLSession.shared.dataTask(with: URLRequest(url: URL(string: "https://jsonplaceholder.typicode.com/posts")!)) { data , response , error in
        onCompletion(data,error)
        
    }.resume()
}

func getPostsAndComments(){
    var dispatchGroup = DispatchGroup()
    dispatchGroup.enter()//Enter call
    fetchPosts { data , error  in
       
        if let error{
            print(error)
          
        }
        if let data{
            print(data)
           
        }
        dispatchGroup.leave()//Leave call
    }
    dispatchGroup.enter()//Enter call
    fetchComments { data , error  in
        
        if let error{
            print(error)
            
        }
        if let data{
            print(data)
           
        }
        dispatchGroup.leave()//Leave call
    }
    dispatchGroup.notify(queue: .main) {
        print("BOTH OF THE API CALLS COMPLETED")
        //Update the UI
    }
}
getPostsAndComments()

```
