The reason as well as the goal why I am developing this framework, sortof,  is to enable a wide range of developers (from junior to senior) develop clean, testable, well tought of code ( as I bring utility of some design patterns closer) while also bringing some ease to refactoring legacy codes. I also intend to introduce, in the process, another angle to implementing dependency injection. Hopefully, with this, you can get started in no time writing clean code.

All suggestions are welcomed.

You can request a feature to be added, or contribute (have some tests to back it up :) )


Thank you.

Sam.



(This documentation will be updated soon to the lattest version of neatchain. feel free to check out the updated tests here :  https://github.com/NeatChain/dev/blob/master/NeatChain.Tests/When_a_chain_is_created.cs and here: https://github.com/NeatChain/dev/blob/master/NeatChain.Tests/when_Number1Handler_is_run_outside_the_usual_pileline.cs )

NeatChain




Get it via nuget

Install-Package NeatChain

===



Neatly set up a chain of objects that are strategically executed


You are provided with a neat opinionated pattern of input argument validation


You are provided with the capability to only execute classes when its their responsibility to do so



```cs

using System;
using System.Collections.Generic;
using System.Linq;
using Microsoft.VisualStudio.TestTools.UnitTesting;

using NeatChainFx.Tests.TestHandlers;

namespace NeatChainFx.Tests
{

```
You can make things as explicit as this
```cs

    [TestClass]
    public class When_a_chain_is_created
    {
        /// <summary>
        ///     You can make things as explicit as this
        /// </summary>
        [TestMethod]
        public void it_should_execute_only_the_member_of_the_chain_that_can_handle_the_argument_passed1()
        {
            var testArg = new List<int> {1, 2, 3};
            var chainSetUp = NeatChain.ThatAcceptsArgumentType<int>.ToBeHandledBy
                .AtMostOneOfTheseHandlers(
                    new Number1Handler(),
                    new Number2Handler(),
                    new Number3Handler());
            testArg.ForEach(arg =>
            {
                List<int> result;
                Assert.IsTrue(chainSetUp.ExecutionChainSucceeded(out result, arg));
                Assert.IsTrue(result.Count == 1);
                Assert.AreEqual(arg*100, result.First());
            });
        }
```
You can make things as simple as this
```cs

        /// <summary>
        ///     You can make things as simple as this
        /// </summary>
        [TestMethod]
        public void it_should_execute_only_the_member_of_the_chain_that_can_handle_the_argument_passed2()
        {
            var testArg = new List<int> {1, 2, 3};
            var  chainSetUp = NeatChain.SetUp(
                ExecutionStrategy.OnlyTheFirsHandlerFoundWhoHasTheResponsibilityIsExecuted,
                new Number1Handler(),
                new Number2Handler(),
                new Number3Handler());
            testArg.ForEach(arg =>
            {
                List<int> result;
                Assert.IsTrue(chainSetUp.ExecutionChainSucceeded(out result, arg));
                Assert.IsTrue(result.Count == 1);
                Assert.AreEqual(arg*100, result.First());
            });
        }
```
 You can make things as easy as this
```cs

        /// <summary>
        ///     You can make things as easy as this
        /// </summary>
        [TestMethod]
        public void it_should_execute_only_the_member_of_the_chain_that_can_handle_the_argument_passed3()
        {
            var  chainSetUp = NeatChain.SetUp(new Number1Handler());

            List<int> result;
            Assert.IsTrue(chainSetUp.ExecutionChainSucceeded(out result, 1));
            Assert.IsTrue(result.Count == 1);
            Assert.AreEqual(100, result.First());

            //though a simple case , there are benefits, such as
            //1. defensive A: you are provided with a neat opinionated pattern of input argument validation
            //2. defensive B: you are provided with the capability to determine if it is the responsibility of the execute method
            //    to process the argument, if not, argument will not even be validated and the execute method will not be called.
            //3. method execution still passes through the same pipeline as chain execution, thus same neat exception handling
        }
```
You can make things as 'one liner' as this
```cs

        /// <summary>
        ///     You can make things as 'one liner' as this
        /// </summary>
        [TestMethod]
        public void it_should_execute_only_the_member_of_the_chain_that_can_handle_the_argument_passed4()
        {
            var result = NeatChain.SetUpWithArgument(1, new Number1Handler()).Execute<int>();

            Assert.IsTrue(result.Count == 1);
            Assert.AreEqual(100, result.First());
        }

        /// <summary>
        ///     Or you can make things as complicated as this
        /// </summary>
        [TestMethod]
        public void it_should_execute_only_the_member_of_the_chain_that_can_handle_the_argument_passed5()
        {
            const string exceptionMessageCreatedInTheConverter = "exceptionMessageCreatedInTheConverter";
            var exceptionWasRaised = false;
            var converterWasCalled = false;
            var testArg = new List<int> {1, 2, 3};
            var  chainSetUp = NeatChain.SetUp(
                ExecutionStrategy.OnlyTheFirsHandlerFoundWhoHasTheResponsibilityIsExecuted,
                new Number1Handler(),
                new Number2Handler(),
                new Number3Handler());
            testArg.ForEach(arg =>
            {
                List<int> result;
                Assert.IsTrue(chainSetUp.ExecutionChainSucceeded(out result, arg, (message, e) =>
                {
                    // if an exception occures, here will be used as the catch block
                    exceptionWasRaised = true;
                    Assert.AreEqual(exceptionMessageCreatedInTheConverter, e.Message);
                }, objectsReturned =>
                {
                    converterWasCalled = true;
                    // the assumption is that each handler may not return the type you want,
                    // e.g a web service call that returns json or xml
                    // usally dyanmic data is returned from each handler, this provides
                    // the opportunity to provide a custom converter to the expected type
                    // else a direct cast will be used
                    var tmpResult = new List<int>();
                    objectsReturned.ForEach(x => tmpResult.Add((int) x));

                    Assert.IsTrue(tmpResult.Count == 1);
                    Assert.AreEqual(arg*100, tmpResult.First());
                    //lets throw an exception here to simulate something going wrong in a given handler
                    throw new Exception(exceptionMessageCreatedInTheConverter);
                    return tmpResult;
                }));
                Assert.IsTrue(result.Count == 0);
                Assert.IsTrue(exceptionWasRaised);
                Assert.IsTrue(converterWasCalled);
            });
        }
        
 ```

```cs
       
    }
}

```
```cs

using System;
using System.Collections.Generic;

namespace NeatChainFx.Tests.TestHandlers
{
    public class Number1Handler : NetChainHandler<int>
    {
        protected override List<Action<int, int>> SetValidations(ChainCondition chainCondition, List<Action<int, int>> validations)
        {
            validations.Add((arg, index) => chainCondition.Requires(arg).IsNotNull());
            validations.Add((arg, index) => chainCondition.Requires(arg).IsAn<int>());
            return validations;
        }


        protected override bool HasResponsibilityToExecute(int arg, List<int> args)
        {
            return (arg == 1);
        }

        protected override List<dynamic> Execute(int arg, List<int> args)
        {
            return new List<dynamic> {arg*100};
        }
    }
}
```

```cs

using System;
using System.Collections.Generic;

namespace NeatChainFx.Tests.TestHandlers
{
    public class Number2Handler : NetChainHandler<int>
    {
        protected override List<Action<int, int>> SetValidations(ChainCondition chainCondition, List<Action<int, int>> validations)
        {
            validations.Add((arg, index) => chainCondition.Requires(arg).IsNotNull());
            validations.Add((arg, index) => chainCondition.Requires(arg).IsAn<int>());
            return validations;
        }

        protected override bool HasResponsibilityToExecute(int arg, List<int> args)
        {
            return (arg == 2);
        }

        protected override List<dynamic> Execute(int arg, List<int> args)
        {
            return new List<dynamic> {arg*100};
        }
    }
}
```

```cs

using System;
using System.Collections.Generic;

namespace NeatChainFx.Tests.TestHandlers
{
    public class Number3Handler : NetChainHandler<int>
    {
        protected override List<Action<int, int>> SetValidations(ChainCondition chainCondition, List<Action<int, int>> validations)
        {
            validations.Add((arg, index) => chainCondition.Requires(arg).IsNotNull());
            validations.Add((arg, index) => chainCondition.Requires(arg).IsAn<int>());
            return validations;
        }

        protected override bool HasResponsibilityToExecute(int arg, List<int> args)
        {
            return (arg == 3);
        }

        protected override List<dynamic> Execute(int arg, List<int> args)
        {
            return new List<dynamic> { arg * 100 };
        }
    }
}



```


```cs
using System.Collections.Generic;
using System.Linq;
using Microsoft.VisualStudio.TestTools.UnitTesting;
using NeatChainFx.Tests.TestHandlers;

namespace NeatChainFx.Tests
{
```

```cs

    [TestClass]
    public class when_Number1Handler_is_run_outside_the_usual_pileline
    {
        [TestMethod]
        public void it_should_only_be_able_to_handle_arguments_that_has_the_value_of_one()
        {
            var inputIsValid = true;
            var number1Handler = new Number1Handler();
            number1Handler.ValidateInputArguments(1, (message, e) => { inputIsValid = false; });
            Assert.IsTrue(inputIsValid);
            Assert.IsTrue(number1Handler.HasResponsibilityToExecute(1));
            Assert.AreEqual(100, number1Handler.Execute(1).FirstOrDefault());
        }
```

```cs

        [TestMethod]
        public void it_should_not_not_be_responsible_to_handle_values_other_than_one()
        {
            var argList = new List<int> {2, 3, -1};

            argList.ForEach(arg =>
            {
                var inputIsValid = true;
                var number1Handler = new Number1Handler();
                number1Handler.ValidateInputArguments(arg, (message, e) => { inputIsValid = false; });
                //input is valid quite alright
                Assert.IsTrue(inputIsValid);
                //but its not its own responsibility to handle it
                Assert.IsFalse(number1Handler.HasResponsibilityToExecute(arg));
                //even though theoratically it can still process it, but in the pipeline, this will not be alloed to happen
                Assert.AreEqual(100*arg, number1Handler.Execute(arg).FirstOrDefault());
            });
        }
        
 ```

```cs
       
    }
}
```



