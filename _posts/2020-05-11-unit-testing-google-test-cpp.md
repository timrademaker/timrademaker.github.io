---
layout: post
title:  "Unit Testing – Google Test (C++)"
date:   2020-05-11 12:00:00 +0200
tag: Visual Studio
categories: C++ "Unit Testing"
---
Microsoft's unit testing framework for C++ is not the only testing framework Visual Studio supports. The "Desktop Development With C++" workload also comes with [Google Test](https://github.com/google/googletest/).

## Default Unit Test File
Creating a new project from the "Google Test" template gives us this file:

```cpp
#include "pch.h"
TEST(TestCaseName, TestName) {
    EXPECT_EQ(1, 1);
    EXPECT_TRUE(true);
}
```

We start off with a single, passing test. Great!

To create your own test methods, use `TEST(TestCaseName, TestName)`. The test case name is used to group tests in the test explorer, which makes it a bit like the `TEST_CLASS` from Microsoft's CppUnitTest framework.

## Asserts
Just like Microsoft's unit testing framework, Google Test comes with asserts. One of the first differences you'll notice is that these are macros instead of static functions.<br>
Another difference is that the default file uses `EXPECT_*`, rather than `ASSERT_*`. The difference between these is that `EXPECT_*` doesn't abort a test case upon failure, whereas `ASSERT_*` does. Therefore, `EXPECT_*` is preferred for general use, unless it doesn't make sense to continue after an assertion fails (for example, when later assertions rely on a value or pointer that has been tested as incorrect).<br>
Most (if not all) of these `ASSERT_*` macros have an `EXPECT_*` counterpart.

Running this test:

```cpp
TEST(TestCaseName, TestName)
{
    ASSERT_EQ(1, 2);
    ASSERT_EQ(2, 3);
}
```

Would give the following output:

```
Expected equality of these values:
    1
    2
```

Replacing `ASSERT` with `EXPECT`, the same test case gives:

```
#1 - Expected equality of these values:
    1
    2
#2 - Expected equality of these values:
    2
    3
```

To provide custom failure messages, use the `<<` operator:

```cpp
EXPECT_EQ(a, b) 
    << "a (" << a << ") and b (" << b << ") are not equal!";
```

### True Check
Of course, Google Test has an assert that verifies that a condition is true:

```cpp
TEST(TestExamples, IsTrue)
{
    EXPECT_TRUE(true);
    EXPECT_TRUE(1 == 1);
    EXPECT_FALSE(false);
    EXPECT_FALSE(1 == 2);
}
```

### Equality Check
Microsoft's framework only has one function for equality checks, whereas Google Test has multiple macros for them. Some basic equality checks:

```cpp
TEST(TestExamples, AreEqual)
{
    EXPECT_FLOAT_EQ(1.0f, 1.0f);                // Without tolerance
    EXPECT_NEAR(1.0f, 1.1f, 0.2f);              // With tolerance
    EXPECT_STREQ("Text", "Text");               // Compare text
    EXPECT_STRCASEEQ("Text", "TEXT");           // Ignore case
    
    EXPECT_NE(1.0f, 2.0f);
    EXPECT_FALSE(std::abs(1.0f - 2.0f) < 0.1f);
    EXPECT_NE(true, false);
    EXPECT_STRNE("Text", "Different text");
}
```

`EXPECT_EQ` has two more versions: `EXPECT_DOUBLE_EQ` and `EXPECT_FLOAT_EQ`. These two can be used to avoid problems caused by rounding.<br>
The string comparisons have a version with `CASE` in them, which makes the assert ignore case.

Some (in)equality assertions that Microsoft's framework does not offer:

```cpp
TEST(TestExamples, AdditionalEqualityChecks)
{
    EXPECT_LT(1, 2);        // Less than (<)
    EXPECT_LE(1, 2);        // Less than or equal to (<=)
    EXPECT_LE(1, 1);        // Less than or equal to (<=)
    EXPECT_GT(2, 1);        // Greater than (>)
    EXPECT_GE(2, 1);        // Greater than or equal to (>=)
    EXPECT_GE(1, 1);        // Greater than or equal to (>=)
}
```

### Instance Comparison
Google Test does not have a specialised assert to check if two references refer to the same object. You could make your own macro for this (`#define EXPECT_SAME(expected, actual) EXPECT_EQ(&expected, &actual`), or you could use a predicate assertion (which I'll cover later in this article).

### NULL Check
Like instance comparison, there is no specialised assert macro to check if a pointer is `NULL`. Instead, you could use `ASSERT_TRUE` to make sure a pointer is not a `nullptr`. `ASSERT_EQ(ptr, nullptr)` would also work. You can even pass in a smart pointer without having to use `.get()`.

```cpp
TEST(TestExamples, IsNullTest)
{
    int* nullPtr = nullptr;
    int* notNullPtr = new int;
    std::unique_ptr<int> uniqueInt = std::make_unique<int>(1);
    std::shared_ptr<int> sharedInt = nullptr;
    EXPECT_FALSE(nullPtr);
    ASSERT_TRUE(uniqueInt);
    *uniqueInt += 1;
    EXPECT_FALSE(sharedInt);
    ASSERT_TRUE(notNullPtr);
    delete notNullPtr;
}
```

Nullptr checks are one of the few cases in which you should use `ASSERT` instead of `EXPECT` (unless the pointer isn't used after the assertion).

### Exception Check
To verify that a function, functor or lambda throws an exception, `EXPECT_THROW` exists. You use it with a function call in it, followed by the expected exception:

```cpp
TEST(TestExamples, ExceptionTest)
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
    EXPECT_THROW(lambda(), std::exception);
    EXPECT_THROW(lambda(1), std::overflow_error);
}
```

Unlike with Microsoft's framework, you don't have to write a wrapping lambda to call a function with some arguments.

If you're fine with any exception being thrown, `EXPECT_ANY_EXCEPTION` can be used. `EXPECT_NO_THROW` exists as well. These both do what you'd expect based on the name.

```cpp
TEST(TestExamples, AdditionalExceptionTest)
{
    ...
    EXPECT_ANY_THROW(lambda());
    EXPECT_NO_THROW(lambda(2));
}
```

### Forced Fail
You can also force a test to fail with `FAIL()`.

## Predicate Assertions
It's possible that the available asserts are not enough. While you can always use `EXPECT_TRUE` to evaluate an expression, this is not ideal. A problem with this method is that the values of parts of the expression are not visible in the default message. Solving this by constructing your own failure message (by streaming it into `EXPECT_TRUE`) can be awkward if the expression has side-effects or is expensive to evaluate.<br>
Google Test provides three different options to solve this problem.

### Using an Existing Boolean Function
If you already have a function that returns a bool (or a type that can be implicitly converted to a bool), you can use a predicate assertion that will print the arguments. This predicate, `EXPECT_PREDN` takes `N` arguments, with `N` being a number between 1 and 5 (inclusive). This assert evaluates the arguments only once, regardless of success or failure.<br>
For example, take this test:

```cpp
bool IsEven(const int a_Number)
{
    return (a_Number % 2) == 0;
}

TEST(PredicateAssertions, AssertPredExample)
{
    const int a = 1;
    EXPECT_PRED1(IsEven, a);
}
```

The test fails, and the following output is given:

```
IsEven(a) evaluates to false, where
a evaluates to 1
```

Disadvantages of this method are that you are limited to 5 arguments (though do you really need more?), and that the arguments you pass need to be usable with the `<<` operator. This last one usually isn't too difficult, but you might not need to know the value of every variable in a struct or class.

### Returning an AssertionResult
Instead of using the predicate macros, you can write your own function that returns an `AssertionResult` instead of a `bool`. To create an `AssertionResult`, you can use `::testing::AssertionSuccess()` and `::testing::AssertionFailure()`. You can stream messages to the `AssertionResult` object using the `<<` operator.<br>
A function and test like this:

```cpp
::testing::AssertionResult IsEvenAR(const int a_Number)
{
    if ((a_Number % 2) == 0)
    {
        return ::testing::AssertionSuccess();
    }
    else
    {
        return ::testing::AssertionFailure() 
            << a_Number << " is odd";
    }
}

TEST(PredicateAssertions, AssertionResultExample)
{
    const int a = 1;
    EXPECT_TRUE(IsEvenAR(a));
}
```

Will give the following output:

```
Value of: IsEvenAR(a)
  Actual: false (1 is odd)
Expected: true
```

You can also supply a success message in case you want to use `EXPECT_FALSE` instead. Do keep in mind that this will make successful tests (using `EXPECT_TRUE`) slower.

### Using a Predicate Formatter
If the default message that `EXPECT_PRED` outputs is unsatisfactory, or some arguments do not support streaming to `ostream`, you can use `EXPECT_PRED_FORMATN` instead. This macro takes a predicate-formatter rather than a regular function or functor. A predicate-formatter has the following signature:

```cpp
::testing::AssertionResult PredicateFormattern(const char* expr1,
                                               const char* expr2,
                                               ...
                                               const char* exprn,
                                               T1 val1,
                                               T2 val2,
                                               ...
                                               Tn valn);
```

`val1` to `valn` are the values of the predicate arguments, and `expr` to `exprn` are "the corresponding expressions as they appear in the source code" (so just the names of the variables).<br>
The following test:

```cpp
::testing::AssertionResult IsEvenFormatted(const char* a_Number_Expression, const int a_Number)
{
    if ((a_Number % 2) == 0)
    {
        return ::testing::AssertionSuccess();
    }
    else
    {
        return ::testing::AssertionFailure() 
            << a_Number_Expression 
            << " (" << a_Number << ") is odd";
    }
}

TEST(PredicateAssertions, PredicateFormatterExample)
{
    const int a = 1;
    EXPECT_PRED_FORMAT1(IsEvenFormatted, a);
}
```

Gives the output:

```
a (1) is odd
```

## Logging
If you want to log additional information while running tests (without the test having to fail), you can just use something like `printf` to output to the default window. This info won't be visible in the output window in Visual Studio, and you can't see it in the test explorer either. Instead, you'll have to run the Google Test project as executable to see this output.

## Test Fixtures
To use the same data for multiple tests, test fixtures can be used. This is a bit like the test classes from Microsoft's framework.

To create a test fixture, create a class that derives from `::testing::Test`. Start of with the `protected` access specifier, and add a default constructor or override `void SetUp()` function to prepare objects for each test. If needed, add a destructor or override `void TearDown()`. Using the constructor and destructor is preferred, [but you might have a reason not to](https://github.com/google/googletest/blob/master/googletest/docs/faq.md#CtorVsSetUp). A new instance of the test fixture is created for every test, so the result is the same.

For per-fixture initialization, add a function with the following signature to the test fixture class: `static void SetUpTestSuite()`. For the cleanup, `static void TearDownTestSuite()` is used.<br>
These functions are only called once per test fixture.<br>
Note: These function signatures have been in use since Google Test version 1.10.0. Before this, the function names `SetUpTestCase` and `TearDownTestCase` were used. You might have to use these instead. If you do update Google Test at some point but don't want to replace these function names, [Google still has you covered](https://github.com/google/googletest/blob/master/googletest/include/gtest/gtest.h#L434).

A fixture could look something like this:

```cpp
class TestFixture : public ::testing::Test
{
protected:
    TestFixture()
    {
        printf("TestFixture constructor\n");
    }

    ~TestFixture()
    {
        printf("TestFixture destructor\n");
    }
    
    // Can be omitted
    static void SetUpTestSuite()
    {
        printf("Test fixture setup\n");
    }

    // Can be omitted
    static void TearDownTestSuite()
    {
        printf("Test fixture teardown\n");
    }

    void SetUp() override
    {
        printf("Test method setup\n");
    }

    void TearDown() override
    {
        printf("Test method teardown\n");
    }
};
```

Using the test fixture requires you to use `TEST_F(TestFixtureName, TestName)` instead of `TEST(TestCaseName, TestName)`.

There might be some things you want to set up for all of the tests. If so, you need to create a class that derives from `::testing::Environment`.

```cpp
class TestEnvironment : public ::testing::Environment 
{
public:
    ~TestEnvironment() override = default;

    void SetUp() override 
    {
        printf("Test environment setup\n");
    }

    void TearDown() override 
    {
        printf("Test environment teardown\n");
    }
};
```

This environment then needs to be registered. To do this, creating a `main` function is recommended:

```cpp
int main(int argc, char** argv)
{
    ::testing::InitGoogleTest(&argc, argv);
    ::testing::AddGlobalTestEnvironment(new TestEnvironment);
    return RUN_ALL_TESTS();
}
```

Google Test takes ownership of the created test environment, so don't delete them yourself.

If you add multiple environments, they are set up in the order in which they are registered. Their teardown happens in reverse order.

## Disabling a Test
To disable a test, prefix the test (case) name with `DISABLED_`. The following tests will be skipped (unless the flag `--gtest_also_run_disabled_tests` is specified):

```cpp
TEST(DISABLED_TestCaseName,TestName) { ... }
TEST(TestCaseName,DISABLED_TestName2) { ... }
TEST_F(TestFixture, DISABLED_Test) { ... }
```

## Conclusion
Google Test has more features than Microsoft's testing framework. There's even things I didn't cover in this article (like death tests).

Getting started with Google Test is pretty easy, yet there's a lot of things to learn about in this framework.

## Example Project
For an example project using Google Test, [check my GitHub](https://github.com/timrademaker/UnitTesting).
