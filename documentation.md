# Batch Job in Dynamics

Tutorial of how to create and schedule a Batch Job in Dynamics 

For more in-depth technical information on SysOperation framework service classes
[see this article](https://supremexpp.wordpress.com/2018/07/25/sysoperation-framework-part-1/)

## Batch Refresh Service in Dynamics

### Problem Summary

In Dynamics, one may want a service to run in the background that can periodically execute some logic or provide
some functionality, such as checking the status of a Payment journal in the background.
This kind of functionality isn't necessarily something that should be executed by the user manually.

### Approach

Implement a Batch job using the _SysOperation Framework_ in Dynamics

Not only will the _Refresh_ service look at the current Journal, but we will extend the functionality to look at all _open_ 
Journals.
Roughly the funtionality of the service running in background will be as follows:
```
for every open Journal to update in our Dynamics instance account
    for every Payment to update in this open Journal
        call the GetACHPaymentStatus method to update status of the Payment
```

Currently the _Refresh_ only works on **APPROVED** transactions, but future direction is to extend the batch service to also give updates
and specific error messages with transactions with an **EXCEPTION**

## Dynamics Frameworks for Batch classes

### SysOperation (Intended Approach)

Newer framework in Dynamics that is used for batch operations in X++.

- Contract classes can be set up with versions on each contract
- Framework is more scalable and maintainable as potential changes and updates happen to any contract class to fit any 
business or developer requirements
- Once we call the method to get ACH Payment status, we can define in the Contract how we want our results to look like, as data comes from  X++ to C# (Connector)
  
### Runbase (Obsolete Approach)

Older framework that uses unstructured `Container`s in X++, which is a similar concept to structs in C

- Not maintainable because the composite datatype `Container` that the user defines may be subject to future changes or updates, 
so convoluted `if` or `switch` statements must be done to maintain different versions 


### Limitations of Batch Refresh Service & SysOperation Framework

In the current state, only the **EXCEPTION** status updates are triggered via the WebHook, which are not guaranteed. 
On the other hand, **APPROVED** status updates are done via _Refresh_

Limitations with the SysOperation framework approach is that there is larger amount of boilerplate code and configuration needed
for setting up Contract classes in X++.

According to Ian Jensen (Dynamics SME), the theoretical max frequency we can update statuses is once per second, but less frequent would be more ideal <br/>
In addition, if the batch recurrence is too high, the average runtime of the status update code may exceed the batch execution frequency, which would cause the processing queue to grow infinitely.

## Disclaimer

Currently a number of these approaches have yet to be validated from a technical perspective. 
