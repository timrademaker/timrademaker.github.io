---
layout: post
title:  "Unit Testing - Microsoft CppUnitTest Framework (C++)"
date:   2020-05-05 12:00:00 +0200
tag: Visual Studio
categories: C++ "Unit Testing"
---
Regardless of whether you write unit tests by choice or because you are forced to, it is beneficial to have some idea about what the unit testing framework you are using (or are going to use) has to offer.

By default, Visual Studio's "Desktop Development With C++" workload comes with Microsoft's own C++ unit testing framework, Google's Testing framework, Boost.Test and CTest. Both Microsoft's and Google's testing frameworks have a project template available, Boost.Test requires [a slightly different setup](https://docs.microsoft.com/en-us/visualstudio/test/how-to-use-boost-test-for-cpp), and CTest works like it would in a CMake environment.

For now though, I'll just be looking at Microsoft's unit testing framework.

## Default Unit Test File
After creating a new project from the "Native Unit Test Project" template, you'll notice that it contains a source file:

```cpp
#include "pch.h"
#include "CppUnitTest.h"
using namespace Microsoft::VisualStudio::CppUnitTestFramework;
namespace NativeUnitTestProject
{
    TEST_CLASS(NativeUnitTestProject)
    {
    public:
         
        TEST_METHOD(TestMethod1)
        {
        }
    };
}
```

A namespace that has the same name as you gave to this project when creating it, one test class with that same name and a single (empty) test method have been generated.

To declare your own test methods, you first need to create a test class using the `TEST_CLASS(className)`-macro (or you can use the generated one). In the test class, you then use the `TEST_METHOD(methodName)`-macro to declare one or more test methods. This can't be done outside of a test class. The namespace is not required, but it serves to separate test modules.

## Asserts
To check if a part of your code has the expected outcome, the `Assert` class can be used. The functions in this class are static, and all of them share two parameters: `const wchar_t* message` and `const __LineInfo* pLineInfo`. These parameters both default to `NULL`.<br>
The `message` parameter can be used to pass a message that is displayed when a test case fails. `pLineInfo` can be useful if you are running the test cases without .pdb files as without these, the stack trace cannot retrieve method names, filenames or line numbers. The macro `LINE_INFO()` exists to automatically gather line info for you.

### True Check
The most straight-forward functions in the `Assert` class is probably `IsTrue`. As the name suggests, this function verifies that a condition is true, and fails a test if the it is actually false. Its counterpart, `IsFalse`, does the opposite.

Both of these functions take a boolean as (first) argument.

```cpp
TEST_METHOD(IsTrueTest)
{
    Assert::IsTrue(true);
    Assert::IsTrue(1 == 1);
    Assert::IsFalse(false);
    Assert::IsFalse(1 == 2);
}
```

### Equality Check
You can put every check in an `IsTrue` call, but using `AreEqual` is also an option. In addition to simply comparing if two values are equal, this function also allows you to compare floats/doubles with tolerance, and to compare `char*` and `wchar_t*` strings (with the option to ignore case).<br>
Like `IsTrue`, this function has a counterpart: `AreNotEqual`. This function has the same options as `AreEqual`.

The first two function arguments are of type `T&` (except for the specialisations). As this function is templated, it can be used for all types.

```cpp
TEST_METHOD(AreEqualTest)
{
    Assert::AreEqual(1.0f, 1.0f);               // Without tolerance
    Assert::AreEqual(1.0f, 1.1f, 0.2f);         // With tolerance
    Assert::AreEqual("Text", "Text");           // Compare text
    Assert::AreEqual("Text", "TEXT", true);     // Ignore case
    Assert::AreNotEqual(1.0f, 2.0f);
    Assert::AreNotEqual(1.0f, 2.0f, 0.1f);
    Assert::AreNotEqual(true, false);
    Assert::AreNotEqual("Text", "Different text");
}
```

### Instance Comparison
If you want to check if two references refer to the same object instance, you can use `AreSame`. Internally, this function takes the addresses of both passed references and compares them. `AreNotSame` checks if two references are not referring to the same object instance.

Like `AreEqual`, the first two function arguments are of type `T&`.

```cpp
TEST_METHOD(AreSameTest)
{
    int someInt = 0;
    int& someIntReference = someInt;
    int anotherInt = 0;
    Assert::AreSame(someInt, someIntReference);
    Assert::AreNotSame(someInt, anotherInt);
}
```

### NULL Check
To verify that a pointer is (not) `NULL`, `IsNull` and `IsNotNull` exist. These do not work directly on smart pointers, so you'll have to use `.get()` to check those.

These functions simply take a `T*` as the first argument.

```cpp
TEST_METHOD(IsNullTest)
{
    int* nullPtr = nullptr;
    int* notNullPtr = new int;
    std::unique_ptr<int> uniqueInt = std::make_unique<int>(1);
    std::shared_ptr<int> sharedInt = nullptr;
    Assert::IsNull(nullPtr);
    Assert::IsNotNull(uniqueInt.get());
    Assert::IsNull(sharedInt.get());
    Assert::IsNotNull(notNullPtr);
    delete notNullPtr;
}
```

### Exception Check
If you want to verify that a function throws an exception in certain cases, `ExpectException` can be used. This checks if the expected exception is actually thrown, and fails the test if another exception, or no exception at all, is thrown. Keep in mind that exceptions derived from the expected exception also pass this assertion. You can, of course, also use this to your advantage if you don't care about the specific type of exception that is thrown.

This function takes a function pointer, functor or lambda as first argument. You can't pass arguments to whatever you pass in, and this assert won't work with functions or functors taking more than zero arguments (even if they have default values). To work around this, use a lambda to wrap the function call.

```cpp
TEST_METHOD(ExceptionTest)
{
    auto lambda = [](int i = 0)
    {
        if (i == 0)
        {
            throw std::exception();
        }
        else if (i == 1)
        {
            throw std::overflow_error("Overflow");
        }
    };
    Assert::ExpectException<std::exception>(lambda);
     
    auto lambdaWrapper = [=]() { lambda(1); };
    Assert::ExpectException<std::overflow_error>(lambdaWrapper);
}
```

### Forced Fail
You can also force a test case to fail by using `Assert::Fail`. This can be useful when the other asserts do not quite suit your needs, and you need to write your own. An example of this would be when you want to check for _any_ exception being thrown:

```cpp
bool caughtAny = false;
try
{
    lambda();
}
catch (...)
{
    caughtAny = true;
}
if (!caughtAny)
{
    Assert::Fail(L"No exception thrown while one was expected!");
}
```

## Logging
In addition to the `Assert` class, Microsoft's unit testing framework also comes with a logger. This class has two overloads of a single static function: `WriteMessage` taking either a `const char*` or a `const wchar_t*` as argument.

This class can be used to provide someone looking at the test output with some extra information. Unlike the messages passed to assertions, `WriteMessage` outputs a message regardless of a test passing or failing.

```cpp
Logger::WriteMessage("This message is visible in test output after running a test!");
```

To see this output, either look in the output console ("Show output from: Test") or check the additional output for a test result in the test explorer.

## Initialization and Cleanup
Maybe there's something that needs to be done before every test, or even before any of the tests run. Doing this in one of the tests is not an option, as there is no guarantee about the order in which tests will run. To deal with this, you can do module-, class- and method-level initialization using the `TEST_*_INITIALIZE(methodName)` macro.<br>
The following code:

```cpp
TEST_MODULE_INITIALIZE(ModuleInit)
{
    Logger::WriteMessage("Test module initialization");
}
TEST_MODULE_CLEANUP(ModuleCleanup)
{
    Logger::WriteMessage("Test module cleanup");
}

TEST_CLASS(TestClass)
{
    TEST_CLASS_INITIALIZE(ClassInit)
    {
        Logger::WriteMessage("Test class initialization (class 1)");
    }
    TEST_CLASS_CLEANUP(ClassCleanup)
    {
        Logger::WriteMessage("Test class cleanup (class 1)");
    }
        
    TEST_METHOD_INITIALIZE(MethodInit)
    {
        Logger::WriteMessage("Test method initialization (class 1)");
    }
    TEST_METHOD_CLEANUP(MethodCleanup)
    {
        Logger::WriteMessage("Test method cleanup (class 1)");
    }
    // [Test methods]
};
// [Second TestClass]
```
Outputs:

```
Test module initialization
Test class initialization (class 1)
Test method initialization (class 1)
Running test method 1
Test method cleanup (class 1)
Test method initialization (class 1)
Running test method 2
Test method cleanup (class 1)
Test class initialization (class 2)
Test method initialization (class 2)
Running test method 1
Test method cleanup (class 2)
Test method initialization (class 2)
Running test method 2
Test method cleanup (class 2)
Test class cleanup (class 1)
Test class cleanup (class 2)
Test module cleanup
```

## Test Attributes
It is also possible to add attributes to tests. Doing so can be useful for filtering in the test explorer:

{% include image.html url="/assets/2020/05/attribute-filter.png" description="Filtering by test attribute" %}

Adding an attribute to a method (named `TestMethod`) is done as follows:

```cpp
BEGIN_TEST_METHOD_ATTRIBUTE(TestMethod)
    TEST_METHOD_ATTRIBUTE(L"AttributeName", L"AttributeValue")
END_TEST_METHOD_ATTRIBUTE()
```

This can be done before and after the test method declaration. You can put multiple attributes between the `BEGIN` and `END` macros.

There are some predefined macros for certain attributes:

```cpp
TEST_OWNER(ownerAlias)
TEST_DESCRIPTION(description)
TEST_PRIORITY(priority)
TEST_WORKITEM(workitem)
TEST_IGNORE()
```

Test methods with the `TEST_IGNORE()`-attribute will be skipped when running tests (like you would expect from a macro with such a name).

You can also add attributes to an entire class or module:

```cpp
BEGIN_TEST_MODULE_ATTRIBUTE()
    TEST_MODULE_ATTRIBUTE(L"ModuleAttribute", L"Value")
END_TEST_MODULE_ATTRIBUTE()
BEGIN_TEST_CLASS_ATTRIBUTE()
    TEST_CLASS_ATTRIBUTE(L"ClassAttribute", L"Value")
END_TEST_CLASS_ATTRIBUTE()
```

## Conclusion
Microsoft's unit testing framework for C++ is quite easy to use. The `Assert` class' functions are likely enough to get you started, and might even be all you need. If this isn't the case, you could always write your own assertions that use `Assert::Fail()`.
Of course, there might be situations where even this doesn't quite cut it. If that's the case, you might want to look at some other unit testing framework like Google Test or Boost.Test.

## Example Project
Want to check out what I talked about in this article without setting up a project for it? A project with the examples used in this article [can be found on my GitHub](https://github.com/timrademaker/UnitTesting).
