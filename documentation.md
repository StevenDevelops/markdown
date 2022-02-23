# Batch Job in Dynamics

Tutorial of how to create and schedule a Batch Job in Dynamics 

For more in-depth technical information on SysOperation framework service classes
[see this article](https://supremexpp.wordpress.com/2018/07/25/sysoperation-framework-part-1/)


### Problem Summary

In Dynamics, one may want a service to run in the background that can periodically execute some logic or provide
some functionality, such as checking the status of a Payment journal in the background.
This kind of functionality isn't necessarily something that should be executed by the user manually.

### Intended Approach - SysOperation

Implement a Batch Job using the _SysOperation Framework_ in Dynamics, which is a newer framework in Dynamics that is used for batch operations in X++:
- Contract classes can be set up with versions on each contract for input parameters
- Framework is more scalable and maintainable as potential changes and updates happen to any contract class to fit any 
business or developer requirements

### SysOperation Code Samples

The SysOperation Framework consists of 3 major component classes, which can be used to run periodic batch functionality in the background.

</br> The **Controller** class, which is used to parse arguments, init data contract parameter values, and initiate functionality
```
[SysOperationJournaledParametersAttribute(true)]
public class SupInterestCalculatorController 
    extends SysOperationServiceController
{    
    public static void main(Args _args)
    {
        SupInterestCalculatorController controller = 
            SupInterestCalculatorController::construct();
        controller.parmArgs(_args);
        controller.startOperation();
    }

    public static SupInterestCalculatorController construct(
        SysOperationExecutionMode _executionMode = 
            SysOperationExecutionMode::Synchronous)
    {
        SupInterestCalculatorController controller = 
            new SupInterestCalculatorController();
        controller.parmExecutionMode(_executionMode);
        return controller;
    }
    
    public void new()
    {
        super(classStr(SupInterestCalculatorService),
            methodStr(SupInterestCalculatorService, 
                CalculateInterest),
            SysOperationExecutionMode::Synchronous);        
    }
}
```

</br> The **Service** class, in which runs the main functionality of the background service
```
public class SupInterestCalculatorService
{
    public void CalculateInterest(
        SupInterestCalculatorDataContract _dataContract)
    {
        AmountMST interestAmountMST = 
            _dataContract.parmBalance() 
                * _dataContract.parmInterestRate();
        
        info(strFmt("The interest is %1", interestAmountMST));
    }
}
```

</br> The **Data Contract** class, in which you can optionally specify the input parameters for the Batch Job
```
[DataContractAttribute]
public class SupInterestCalculatorDataContract
{
    protected Percent interestRate;
    protected AmountMST balance;

    [DataMemberAttribute]
    public Percent parmInterestRate(
        Percent _interestRate = interestRate)
    {
        interestRate = _interestRate;
        return interestRate;
    }

    [DataMemberAttribute]
    public AmountMST parmBalance(AmountMST _balance = balance)
    {
        balance = _balance;
        return balance;
    }
}
```

  
### Obsolete Approach - Runbase

Older framework that uses unstructured `Container`s in X++, which is a similar concept to structs in C.
- Not maintainable because the composite datatype `Container` that the user defines may be subject to future changes or updates, 
so convoluted `if` or `switch` statements must be done to maintain different versions 



### Run your Batch Job
Once we have implemented the code samples above, we can schedule, run, and debug your Batch Job with the following video
guides



## Sources

https://supremexpp.wordpress.com/2018/07/25/sysoperation-framework-part-1/ 
https://msdax.wordpress.com/tag/sysoperationjournaledparametersattribute/ 
https://www.schweda.net/blog_ax.php?bid=613&wdl=en 
https://dynamics365musings.com/sysoperation-framework-in-d365/ 
https://stoneridgesoftware.com/understanding-report-controller-classes-in-dynamics-ax-2012/ 
