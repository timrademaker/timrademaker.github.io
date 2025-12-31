---
layout: post
title:  "Unit Testing – Google Mock"
tag: Visual Studio
categories: C++ "Unit Testing"
---
Another part of [Google Test's GitHub repository](https://github.com/google/googletest) is Google Mock. This is Google's framework for writing and using C++ mock classes.

## What _is_ a Mock Class?
A mock class implements the same interface as a real object, but doesn't actually do anything that a regular object would. Instead, mock objects let you specify how they will be used and what they should do at runtime. Which methods are going to be called? In which order will they be called? How often will a method be called? What arguments will they receive? What will they return?

Mock objects are not the same as fake objects. Fake objects do have a working implementation, but this implementation usually contains shortcuts that makes them unsuitable for production. This can be to make an operation less expensive, but there's could be other reasons as well.<br>
An example of a fake object would be an in-memory file system. Instead of actually reading from a file, it just returns what you specify in advance. Writing to a “file” would actually be writing to memory.<br>
Another example of a fake object would be a login page that doesn't actually do any authentication.

A mock object is "[pre-programmed with expectations, which form a specification of the calls they are expected to receive.](https://github.com/google/googletest/blob/master/googlemock/docs/for_dummies.md)"

## What Are We Testing?
A mock class doesn't actually help in testing the class that is being mocked. Instead, it allows you to check the interaction between a regular class and the mocked class.

So, when testing a login page, you might want to see how often authentication is attempted. In this case, you wouldn't use a fake login page that doesn't even attempt to authenticate. Instead, you'd have a mock authentication system in place. This could then track how often a certain method is called.

## Making a Mock Class
To make a mock class, the `MOCK_METHOD` macro is used. To this macro, you pass the return type, function name, argument list, and optionally the function's qualifiers (const, override, noexcept).<br>
It would look something like this:

```cpp
class MockClass
{
public:
    MOCK_METHOD(ReturnType, MethodName, (Args...), (Specs...);
};
```

All mocked methods must be public. Yes, derived classes are able to change the access level of a virtual function from a base class.

Google Mock's documentation has an example of how to mock a class (in this case a “Turtle”, using a LOGO-like API for drawing). Given the following class:

```cpp
class ITurtle
{
public:
    ITurtle() = default;
    virtual ~ITurtle() = default;
    virtual void PenUp() = 0;
    virtual void PenDown() = 0;
    virtual void Forward(int a_Distance) = 0;
    virtual void Turn(int a_Degrees) = 0;
    virtual void GoTo(int a_X, int a_Y) = 0;
    virtual int GetX() const = 0;
    virtual int GetY() const = 0;
};
```

Mocking its functions would make it look like:

```cpp
#include "gmock/gmock.h"
class MockTurtle : public ITurtle
{
public:
    MOCK_METHOD(void, PenUp, (), (override));
    MOCK_METHOD(void, PenDown, (), (override));
    MOCK_METHOD(void, Forward, (int a_Distance), (override));
    MOCK_METHOD(void, Turn, (int a_Degrees), (override));
    MOCK_METHOD(void, GoTo, (int a_X, int a_Y), (override));
    MOCK_METHOD(int, GetX, (), (const, override));
    MOCK_METHOD(int, GetY, (), (const, override));
};
```

## Testing With a Mock Class
Tests follow the same steps every time: First, you create a mock object. Then, you set expectations for the mock object. Finally, you execute some actions.

Let's say we have an implementation of a painter class. Testing it with the mock turtle could look like this:

```cpp
using ::testing::AtLeast;
using ::testing::_;
TEST(PainterTest, CanDrawSomething)
{
    // Create mock object
    MockTurtle turtle;
    // Set expectations
    EXPECT_CALL(turtle, PenDown())
        .Times(AtLeast(1));
    EXPECT_CALL(turtle, Forward(_))
        .Times(AtLeast(1));
    EXPECT_CALL(turtle, Turn(_))
        .Times(AtLeast(1));
    EXPECT_CALL(turtle, PenUp())
        .Times(AtLeast(1));
    // Execute actions
    Painter p(turtle);
    EXPECT_TRUE(p.DrawCircle(0, 0, 10));
}
```

### How Often Is a Method Called?
To set expectations on a method being called, the macro `EXPECT_CALL(object, function)` is used. The `.Times()`-function sets an expectation on how often a function is called. In the example above, `AtLeast(1)` is used for all expected calls. This isn't our only option.

```cpp
AnyNumber()    // Called any number of times
AtLeast(n)     // Call expected at least n times
AtMost(n)      // Call expected at most n times
Between(m, n)  // Call expected between m and n times (inclusive)
Exactly(n)     // Call is expected exactly n times. "Exactly" can be omitted
```

If you want to explicitly disallow a call to a function, use `Times(0)`.

### What Arguments Are Expected?
The `EXPECT_CALL` macro doesn't just take the name of the function. It also takes the _arguments_ the function is expected to be called with. In the example, the value doesn't matter. Instead of a value, `_` is used. Again, there's more options.

```cpp
_               // Any value (of the correct type)
Eq(value)       // argument == value
Ge(value)       // argument >= value
Gt(value)       // argument > value
Le(value)       // argument <= value
Lt(value)       // argument < value
Ne(value)       // argument != value
```

It doesn't even end there. There's string matchers, floating point matchers, and container matchers as well.

### Expected Return Values
Mock objects can return values. By default, mock classes return nothing, `false` or `0`. If we do need a mock object to return something, we can specify this using `WillOnce(action)` and `WillRepeatedly(action)` in combination with `Return(n)`. 

```cpp
TEST(PainterTest, ExpectReturn)
{
    // Create mock object
    MockTurtle turtle;
     
    // Set expectations
    // GetY will be called once, and will return 100
    EXPECT_CALL(turtle, GetY())
        .Times(1)
        .WillOnce(Return(100));         // Return 100 once
    // GetX
    EXPECT_CALL(turtle, GetX())
        .Times(5)
        .WillOnce(Return(100))          // Return 100 once
        .WillOnce(Return(200))          // Return 100 once
        .WillRepeatedly(Return(300));   // Return 300 for the remaining calls
    // Execute actions
    ...
}
```

### Sequence of Function Calls
In some cases, function calls could be expected to be made in a certain order. To test this, the class `InSequence` can be used.

```cpp
TEST(PainterTest, SequenceExample)
{
    // Create mock objects
    MockTurtle turtle;
    MockTurtle turtle2;
    // Set expectations
    InSequence seq;
    EXPECT_CALL(turtle, GetX())
        .Times(1);
    EXPECT_CALL(turtle2, GetX())
        .Times(1);
    EXPECT_CALL(turtle, GetY())
        .Times(1);
     
    // Execute actions
    turtle.GetX();
    turtle2.GetX();
    turtle.GetY();
}
```

From the moment the `InSequence` object is created onward (until it goes out of scope), the set expectations are expected in the sequence in which they are set. If a function call to another function that is not in the sequence is made, this won't fail the test (but will be marked as uninteresting call).

Maybe you want multiple sequences. This is also possible, using the `Sequence` class.

```cpp
TEST(PainterTest, MultipleSequences)
{
    // Create mock objects
    MockTurtle turtle;
    MockTurtle turtle2;
    // Set expectations
    Sequence seq;
    Sequence seq2;
    EXPECT_CALL(turtle, GetX())
        .Times(1)
        .InSequence(seq, seq2);
    EXPECT_CALL(turtle, GetY())
        .Times(1)
        .InSequence(seq);
    EXPECT_CALL(turtle2, GetX())
        .Times(1)
        .InSequence(seq2);
    EXPECT_CALL(turtle, GoTo(_, _))
        .Times(1)
        .InSequence(seq);
    // Execute actions
    // Both sequences expect this call first
    turtle.GetX();
    // Sequence 1
    turtle.GetY();
    turtle.GoTo(0, 0);
    // Sequence 2
    turtle2.GetX();
}
```

In this example, two sequences are made. Both sequences expect a call to `GetX`, `seq` then expects calls to `GetY` and `GoTo`, and `seq2` expects a call to `GetX` again. The calls are made to two turtles.

The order in which the sequences are finished does not matter. Calling the methods in the order `GetY`, `GetX` and GoTo would still pass the test as the function calls don't interfere.<br>
In diagram form:

{% include image.html url="/assets/2020/05/turtle-multiplesequences-flow.png" description="Flowchart of two branching sequences" %}

It is possible to make the two sequence branches reconnect at some point, by adding an expectation to both sequences again. This complicated the test, so avoid this whenever possible.

### Sticky Expectations
By default, expectations are “sticky”. This means they stay indefinitely. In some cases, this might unexpectedly cause a test to fail:

```cpp
TEST(PainterTest, StickyExpectations)
{
    // Create mock object
    MockTurtle turtle;
    // Set expectations
    EXPECT_CALL(turtle, GoTo(_, _))
        .Times(AnyNumber());
    EXPECT_CALL(turtle, GoTo(0, 0))
        .Times(2);
    // Execute actions
    turtle.GoTo(0, 0);
    turtle.GoTo(0, 0);
    turtle.GoTo(0, 0);       // Will cause the test to fail
}
```

Even though `GoTo` is also expected with any argument, calling it with `(0, 0)` a third time causes the test to fail. This is because `(0, 0)` specifically is expected only twice. The expectation for this is still around to make sure it is only called twice.

When expectations are matched, the one that was added last is used. So in this case, swapping the positions of the expectations would cause the test to fail as well: The expectation for `(0, 0)` is never matched, as any call with those arguments also matches `(_, _)`.

To solve this issue, an expectation can be retired using `RetiresOnSaturation()`. Sequences automatically retire their expectations, so you don't have to explicitly do this in those.

### Default Actions
It is also possible to set a default action, without setting expectations. This can be done by using the macro `ON_CALL` and the function `WillByDefault(action)`.

```cpp
TEST(PainterTest, WillByDefaultExample)
{
    // Create mock object
    MockTurtle turtle;
    // Set (no) expectations
    ON_CALL(turtle, GetX())
        .WillByDefault(Return(100));
    // Make an uninteresting call
    ASSERT_EQ(turtle.GetX(), 100);
}
```

In this example, no expectations on how often a function will be called are set. Instead, a default action is set. By default, a specific function call to a specific object will return a specified value (100). This can be (temporarily) overridden by setting expectations for this specific function call:

```cpp
EXPECT_CALL(turtle, GetX())
    .Times(AtLeast(1))
    .WillOnce(Return(1));
```

The first time this function is called, 1 will be returned. Every call after this will return 100, as this is the default value that was set.

`Return` isn't the only action Google Mock has. The rest can be found [on GitHub](https://github.com/google/googletest/blob/master/googlemock/docs/cheat_sheet.md#actions-actionlist).

## Types of Mock Objects
Google Mock currently supports three types of mock objects: Nice, Naggy and Strict. Creating these is done using `NiceMock<MockClass>`, `NaggyMock<MockClass>` and `StrictMock<MockClass>` respectively. By default, mock objects are naggy.

Nice objects ignore all uninteresting calls. If a call is not expected, it won't result in a warning.<br>
Naggy objects log a warning for all uninteresting calls.<br>
Strict objects simply fail the test if any uninteresting call is made.

## Mocking a Non-Virtual Class
If you want to test interaction with a class containing both virtual and non-virtual functions, things get a bit more complicated (assuming you don't just want to test interaction with the virtual functions).

Possible solutions to this problem are:
1. Only test interaction with virtual functions
2. Make all functions virtual
3. Use the adapter design pattern, and mock the adapter
4. Create a mock class that implements the same functions as the original class

Option 1 and 2 are not recommendable. Only testing interaction with some functions would reduce test coverage, and might even make testing useless. Making all functions virtual would remove what not making a function virtual can signal: "This function should not be overridden by anyone".

Option 3 would mean changing a class to use an adapter instead of the actual object. This is an option in some situations, but isn't always the answer.

Option 4 is the least destructive. To make this work with existing code, the class you want to test would have to be templatized.

For example, we might have a `GameObject` class:

```cpp
class GameObject
{
public:
    GameObject() = default;
    virtual ~GameObject() = default;
    virtual void OnBeginPlay() {}
    virtual void OnUpdate(float a_DeltaTime) {}
    virtual void OnDestroy() {}
    void MoveBy(float a_X, float a_Y, float a_Z)
    {
        m_Position[0] += a_X;
        m_Position[1] += a_Y;
        m_Position[2] += a_Z;
    }
private:
    float m_Position[3] = { 0.0f, 0.0f, 0.0f };
};
```

To manage our `GameObject`s, we have a `GameObjectManager` class:

```cpp
class GameObjectManager
{
public:
    GameObjectManager() = default;
    ~GameObjectManager() = default;
    void AddObject(std::shared_ptr<GameObject> a_Object)
    {
        m_Objects.push_back(a_Object);
    }
    void UpdateObjects(float a_DeltaTime)
    {
        for (auto& obj : m_Objects)
        {
            obj->OnUpdate(a_DeltaTime);
        }
    }
    void MoveObjects(float a_X, float a_Y, float a_Z)
    {
        for (auto& obj : m_Objects)
        {
            obj->MoveBy(a_X, a_Y, a_Z);
        }
    }
private:
    std::vector<std::shared_ptr<GameObject>> m_Objects;
};
```

We want to test interaction between these two classes. To do so, a `MockGameObject` class would have to be created.<br>
Let's say we want to verify that `GameObjectManager::UpdateObjects` calls `GameObject::OnUpdate`. We could create a mock game object and override this function.<br>
We also want to test if `GameObjectManager::MoveObjects` actually calls `GameObject::MoveBy`. `GameObject::MoveBy` is non-virtual, so we can't override it in order to make a mock object.

We create a new class, `MockGameObject`. This class doesn't derive from `GameObject`, even though it mocks that class.<br>
We only need to implement functions that will be interacted with by `GameObjectManager` in our tests. In this case, that's only `OnUpdate` and `MoveBy`.

```cpp
class MockGameObject
{
public:
    MOCK_METHOD(void, OnUpdate, (float a_DeltaTime));
    MOCK_METHOD(void, MoveBy, (float a_X, float a_Y, float a_Z));
};
```

We change `GameObjectManager` a bit to make it work with `MockGameObject`:

```cpp
template<class GameObjectBase>
class GameObjectManager
{
public:
    ...
    void AddObject(std::shared_ptr<GameObjectBase> a_Object)
    ...
private:
    std::vector<std::shared_ptr<GameObjectBase>> m_Objects;
};
```

If this is something you set up early in a project, you won't have any problems when adding a mocked version of `GameObject`. If using Google Mock is a late addition, you don't necessarily have to replace all instances of `GameObjectManager` with `GameObjectManager<GameObject>`. Instead, you could rename the `GameObjectManager` class to something else (like `GameObjectManagerClass`, if you're as "good" at naming classes as I am). Then, add using `GameObjectManager = GameObjectManagerClass<GameObject>;` below the class declaration.

After doing this, testing is the same as it is with any other mock class.

## Conclusion
Google Mock is an interesting library. Its functionality goes a lot further than what I usually did to test class interaction (which was simply using a variable to track how often a function was called). Once you get an idea of how Google Mock works, using it is actually quite easy. There's a lot of depth to this library as well, so I'd definitely recommend giving it a go if you want to test class interaction.

## Example Project
For an example project using Google Mock, [check my GitHub](https://github.com/timrademaker/UnitTesting).
