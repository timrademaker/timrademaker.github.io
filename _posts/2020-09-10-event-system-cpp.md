---
layout: post
title:  "Event System (C++)"
date:   2020-09-10 12:00:00 +0200
tag: Templates
categories: C++
---
If you want to have two objects communicate with each other without coupling them, the [observer design pattern](https://gameprogrammingpatterns.com/observer.html) might just do what you want.

Making the subject something that is owned by a class (rather than inherited by it) is simple, but I wanted the observer to not be inherited by the class that observes an object either. Giving the observing class a (non-virtual) function that the subject can call would be a possibility, but how can this be done?

## Function Pointers
Let’s say we want to store a pointer to a function with the signature void `FunctionName(bool)` in a variable named `funcPtr`. We can do this as follows: `void(*funcPtr)(bool) = &FunctionName`. Calling the function can then be done with `funcPtr(true)`. I’ll keep remarks about the readability of the variable declaration to myself.

Storing a member function pointer using the same method is not possible due to a type mismatch. Apparently, the type I’m trying to store is `void(ClassName::*)(bool)`.

This complicates things a bit. We don’t know the name of classes we (or someone else) will make in the future, so does that mean we can’t store member function pointers? Luckily, `std::function` can help with this. The type we are trying to store then simply becomes `std::function<void(bool)>`. This can be used to store both free- and member functions, so that’s a relief.

Of course, we don’t just want to limit the user to a single parameter of type `bool`. We’ll get to a way of dealing with that in a bit, but it can be done using templates (as you probably guessed).

## Binding Member Functions
You can’t simply use `std::function<void(bool)> funcPtr = &ClassName::Function` to store a member function pointer. Instead, you can use `std::bind`, which generates a forwarding call wrapper around the function.

`std::bind` can be used for [partial function application](https://en.wikipedia.org/wiki/Partial_application), but you can also opt to have all parameters as placeholders.

## Implementation Choices
### Member Functions
We need at least two functions in the event class: `Subscribe`, to subscribe a delegate to an event, and `Broadcast`, to broadcast the event to the subscribed delegates.

There’s more functions that would be nice to have in an event class (like having an `Unsubscribe` function to unsubscribe delegates), but `Subscribe` and `Broadcast` are the bare minimum.

### Number of Arguments
Something we still need to decide is how many arguments the bound functions have to take. Two options:

#### As Many As Needed
There’s no way of knowing what info someone would want to send out with an event. Therefore, not limiting the user would make sense.

We’ll have to know the number and type of parameters when the event is declared, so that we know the type this event should store. [Parameter packs](https://en.cppreference.com/w/cpp/language/parameter_pack) make this possible:

```cpp
template<typename... EventArgs>
class EventBase
{
    using DelegateType = std::function<void(EventArgs...)>;
public:
    void Subscribe(DelegateType a_Delegate);
    void Broadcast(EventArgs... a_Args);
};
```

This method has some disadvantages.<br>
1. It’s not unlikely for two events to have the same parameter types. That doesn’t mean we want to be able to bind a delegate for one event to a completely different event, yet we can’t prevent that. We would have to assume the user knows what they’re doing, and doesn’t make this mistake. If possible, I’d like to avoid making that assumption.
2. Let’s say we have an event that sends out an int, a float and a bool. That’s cool and all, but what exactly do they represent? You might know at the time you declare the event, but after that you might have to check a class that subscribes to the event. Assuming you can find one.
3. Binding a member function that takes 6 (placeholder) arguments won’t look great. This can be dealt with using a macro, so this is not the biggest issue (unless you utterly despise macros).

#### A Single Argument
```cpp
template<typename EventInfoType>
class EventBase
{
    using DelegateType = std::function<void(EventInfoType)>;
public:
    void Subscribe(DelegateType a_Delegate);
    void Broadcast(EventInfoType a_Event);
};
```

This solves the disadvantages the previous method has.
1. You can’t bind void func1(SomeEvent) to an event that takes OtherEvent as argument.
2. By using an event struct or class, we can name our variables. This makes finding out what they represent much easier, assuming they are named properly.
3. You’ll only have to use a single placeholder in your function binds.

Something else you might consider an advantage is that you can add more variables to an event without having to change the signature of all delegates (though you’ll still have to make these delegates use the new variable). 

### Ease-of-Use
#### Event Declaration
Declaring a new event is not difficult; a simple `using SomeEvent = EventBase<SomeEventInfo>` will suffice. Of course, you can also avoid using altogether, but then you’ll have `EventBase<SomeEventInfo> m_SomeEvent` in your class instead of `SomeEvent m_SomeEvent` (which comes down to preference, I guess).

If that’s not clear enough, you can create a macro for this: `#define DECLARE_EVENT(EventName, EventInfoType) using EventName = EventBase<EventInfoType>`. Unreal Engine does something like this as well, though that doesn’t automatically mean it’s good.

### Subscribing to an Event
Before you can pass a member function to a function taking a std::function, the function needs to be bound using std::bind. So, if you want to subscribe to an event, you would use something like

```cpp
someEvent.Subscribe(std::bind(&SomeClass::EventCallback, someClassInstance, std::placeholders::_1))`
```

That doesn’t look too nice, but a macro might be able to solve this:

```cpp
#define EVENT_BIND(Function, Object) std::bind(Function, Object, std::placeholders::_1)
```

Then, subscribing would be done like

```cpp
someEvent.Subscribe(EVENT_BIND(&SomeClass::EventCallback, someClassInstance))
```

This might save us some typing, but doesn’t look too great either.

There’s another, different, way of doing this. Earlier, I mentioned a member function with the signature `void(ClassName::*)(bool)`. We don’t know the name of the class a function belongs to in advance, but that’s exactly what templates can help us with. Change `Subscribe`'s signature to this:

```cpp
template<typename UserClass>
void Subscribe(void(UserClass::* a_Callback)(EventInfo))
```

And suddenly, we can pass in a member function without binding it first. Calling this function is done like `someEvent.Subscribe<ClassName>(&ClassName::EventCallback)`.<br>
This leaves one issue, namely that we have no object to call this member function on. Adding a function parameter for that deals with this issue, and also removes the need to explicitly specify a type for `UserClass` as this can be deducted from the parameter.

Now, the function can be bound internally. Additionally, callbacks can now (internally) be stored together with their owning object. As a result, unsubscribing all callbacks from an object is now possible.
