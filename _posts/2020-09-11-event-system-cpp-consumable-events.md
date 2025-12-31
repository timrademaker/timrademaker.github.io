---
layout: post
title: "Event System (C++) – Consumable Events"
tag: Templates
categories: C++
excerpt: A possibility for an event system is to make events consumable. That is, a certain delegate can tell the event system to stop sending out the event it just received as other delegates shouldn’t be handling it any more.
---
This article is a continuation of [my previous article]({% post_url 2020-09-10-event-system-cpp %}), where I wrote about event systems. Reading it it not required to understand this article, but it might explain some things I mention.

## Consumable Events
A possibility for an event system is to make events consumable. That is, a certain delegate can tell the event system to stop sending out the event it just received as other delegates shouldn't be handling it any more. A use case for this could be an input system.<br>
I'm not talking about this as (an attempt at) optimisation, though this might be possible as well.

This requires some considerations.
1. The event broadcasting order needs to be guaranteed. The order not being guaranteed opens up the possibility of the event being stopped by the wrong delegate.
2. Delegates need a way of telling the event that it was consumed. One way of doing this is having all delegates return a boolean (rather than void). However, what if you don't want an event to be consumable?

For 1, using a sequential container is part of the solution. Giving the user control over the order itself might be a bit more difficult.<br>
Something that could be done here is a bit of a compromise: Let users choose a priority level for their events. When an event is consumed, it doesn't reach delegates with a lower priority level, while still reaching delegates at the same level.

For 2, you could use a boolean that indicates if an event is consumable or not. If it is, check the return value of the delegates. If it's not, don't check it.<br>
All delegates would have to return a boolean like this, and all delegate function signatures would be somewhat similar (`bool Delegate(EventInfo)`). This doesn't clearly show the user whether events can be consumed or not.<br>
You could add the return type (`bool` or `void`) as template parameter, but there's a problem with this: you can't check the return value of a function returning `void` in an `if`-statement. So, you'd need to implement the `Broadcast`-function in two different ways.

### Partial Specialization
[Partial specialization](https://en.cppreference.com/w/cpp/language/partial_specialization) allows you to customise the implementation of a struct or class for specific template arguments. That said, partial specialization only works for classes and structs. Not functions, as those have to be explicitly (or fully) specialized. You can read why this is the case in [this article by Herb Sutter](http://www.gotw.ca/publications/mill17.htm).

As this means we can't use partial specialization to solve this, do we have to write a second class with the exact same code (except for a single return value that's not checked in one of the two)? Luckily, there's another (possible) solution: `std::enable_if`.

### std::enable_if
[`std::enable_if`](https://en.cppreference.com/w/cpp/types/enable_if) can be used to leverage the concept of [SFINAE](https://en.cppreference.com/w/cpp/language/sfinae) in order to conditionally remove functions. If the condition passed to `std::enable_if` is `false`, the function declaration it is used in is ill-formed and discarded. Using this allows us to have one function implementation for a non-consumable event, and one for a consumable event without implementing the entire class twice.

`std::enable_if` can't be used with a template type specified at class-level, but has to use a function-level template type ([explained here](https://stackoverflow.com/a/6972771)). To deal with this, it's possible to use a kind of hacky solution:

```cpp
template<typename EventInfoType, typename ReturnType>
class EventBase
{
public:
    template<typename T = ReturnType>
    void Broadcast(EventInfoType a_Event);
    // Rest of the class
};
```

Actually using `std::enable_if` here:

```cpp
template<typename EventInfoType, typename ReturnType>
class EventBase
{
public:
    template<typename T = ReturnType>
    typename std::enable_if<std::is_void<T>::value>::type Broadcast(EventInfoType a_Event);
    template<typename T = ReturnType>
    typename std::enable_if<!std::is_void<T>::value>::type Broadcast(EventInfoType a_Event);
    // Rest of the class
};
```

This results in a single `Broadcast` function being defined per event, as ReturnType can't be void and not void at the same time. Now, two variations of the `Broadcast` function can be implemented without implementing the entire class twice.

Note that I am assuming that any type other than `void` is bool-convertible. You'll probably want to limit `ReturnType` to `void` and `bool` anyway, and possible ways of doing that include using a `static_assert` or a templated `constexpr bool`, which would be the same as the check done in the first `enable_if` template argument.

```cpp
// In a function
static_assert(std::is_same<T, void>::value);
// Outside of the class
template<typename T>
constexpr bool is_void_v = std::is_same<T, void>::value;
```

The `enable_if` can also be placed in the template parameter list, if you prefer. Using a `constexpr` bool like `is_void_v`:

```cpp
template<typename EventInfoType, typename ReturnType>
class EventBase
{
public:
    template<typename T = ReturnType, class = std::enable_if<is_void_v<T>>::type>
    void Broadcast(EventInfoType a_Event);
    template<typename T = ReturnType, class = std::enable_if<!is_void_v<T>>::type>
    void Broadcast(EventInfoType a_Event);
    // Rest of the class
};
```

When doing this, the function definition would look something like this:

```cpp
template<typename T, class _Enabled>
    void Broadcast(EventInfoType a_Event)
{
    // Implementation
}
```

### If Constexpr
When using C++17 or higher, `if constexpr` is a thing. You can have a single public `Broadcast` function, while having two (private) implementations:

```cpp
template<typename EventInfoType, typename ReturnType>
class EventBase
{
public:
    void Broadcast(EventInfoType a_Event)
    {
        if constexpr (std::is_void<ReturnType>::value)
        {
            BroadcastRegularEvent(a_Event);
        }
        else
        {
            BroadcastConsumableEvent(a_Event);
        }
    }
private:
    void BroadcastConsumableEvent(EventInfoType a_Event);
    void BroadcastRegularEvent(EventInfoType a_Event);
    // Rest of the class
};
```

### Tag Dispatching
If `if constexpr` is not an option, you can also use [tag dispatching](https://www.fluentcpp.com/2018/04/27/tag-dispatching/). A "tag" is a type that has no behaviour and no data.<br>
Tag dispatching requires an extra argument in the internal functions, so it's simply an application of function overloading.

```cpp
template<typename EventInfoType, typename ReturnType>
class EventBase
{
public:
    void Broadcast(EventInfoType a_Event)
    {
        Broadcast_Impl(a_Event, std::bool_constant<!std::is_void<ReturnType>::value>());
    }
private:
    // Consumable event
    void Broadcast_Impl(EventInfoType a_Event, std::true_type);
    // Non-consumable event
    void Broadcast_Impl(EventInfoType a_Event, std::false_type);
    // Rest of the class
};
```

In this case, only two tags are needed. When there's more than two options, you can use empty structs or classes instead.

## Conclusion
When working on an event system, you might just want to choose between having all events be consumable or having none be. The latter is probably easier, as it takes away the need for delegate ordering and prevents events from "disappearing" (i.e. being consumed by a delegate that shouldn't consume it). It also results in there only being one return type for delegates, removing needless complication for the user (who would have to find out what return type is expected).
