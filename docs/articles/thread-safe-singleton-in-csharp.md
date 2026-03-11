---
layout: page
title: How to Write a Thread-Safe Singleton in C# 13
description: A back to basics look at the Singleton pattern in C# 13, covering thread safety, lazy initialisation and the approaches I'd recommend today.
date: 2026-03-11T12:00:00
---

This is a bit of a back to basics post. I've been doing a lot of infrastructure and DevOps work over the last few years and recently had a conversation where the Singleton pattern came up. I realised I wanted to refresh my knowledge on the thread-safe implementations, especially with some of the newer C# 13 features. This article is mainly a reference for myself, but hopefully someone else finds it useful too.

## Why Thread Safety Matters

The Singleton pattern is meant to ensure a class only ever has one instance. The problem is that if two threads simultaneously check whether the instance exists and both find it `null`, you end up with two instances. That completely defeats the point of the pattern.

I've seen this cause some really subtle bugs in production, so it's worth getting right.

## Approach 1: Lazy&lt;T&gt;

This is my preferred approach these days. `Lazy<T>` handles thread safety and lazy initialisation out of the box, so there's very little you can get wrong;

```csharp
public sealed class Singleton
{
    private static readonly Lazy<Singleton> _instance = new(() => new Singleton());

    public static Singleton Instance => _instance.Value;

    private Singleton()
    {
        // Initialisation logic here
    }
}
```

`Lazy<T>` is thread-safe by default (`LazyThreadSafetyMode.ExecutionAndPublication`). The instance is created the first time `Instance` is accessed, and all concurrent callers will get the same instance. Simple as that.

## Approach 2: Static Field Initialisation

If you don't need lazy creation, this is the simplest possible implementation. C# guarantees that static field initialisers run only once per application domain, in a thread-safe manner;

```csharp
public sealed class Singleton
{
    private static readonly Singleton _instance = new();

    public static Singleton Instance => _instance;

    private Singleton()
    {
        // Initialisation logic here
    }
}
```

The trade-off is that the instance is created when the type is first referenced, not necessarily when `Instance` is first accessed. If the class has other static members, the instance may be created earlier than you'd like. In practice, I've found this is rarely a problem.

## Approach 3: Double-Check Locking

I'm including this for completeness as it's the classic approach you'll see in a lot of older code. I wouldn't recommend writing new code this way when `Lazy<T>` exists, but it's good to understand how it works;

```csharp
public sealed class Singleton
{
    private static volatile Singleton? _instance;
    private static readonly Lock _lock = new();

    public static Singleton Instance
    {
        get
        {
            if (_instance is null)
            {
                lock (_lock)
                {
                    _instance ??= new Singleton();
                }
            }

            return _instance;
        }
    }

    private Singleton()
    {
        // Initialisation logic here
    }
}
```

A few things to note here;

- The `??=` is the null-coalescing assignment operator. It only assigns the right-hand side if the left-hand side is `null`. So `_instance ??= new Singleton()` is shorthand for `if (_instance is null) { _instance = new Singleton(); }`. This is the key part of the double-check pattern. If another thread already created the instance while we were waiting for the lock, `_instance` won't be `null` and we just skip the assignment. Without this check inside the lock, both threads would create a new instance and you'd end up with two.
- C# 13 introduces the `Lock` type (`System.Threading.Lock`) as a replacement for locking on arbitrary `object` instances. It's purpose-built for synchronisation and provides better performance and clarity than the old `lock (new object())` approach you might have seen.
- The `volatile` keyword ensures that reads and writes to `_instance` are not reordered by the compiler or CPU, which is essential for the double-check pattern to work correctly. Getting this wrong can lead to some very confusing bugs.

## Which Approach Should You Use?

| Approach | Thread-Safe | Lazy | Complexity |
| --- | --- | --- | --- |
| `Lazy<T>` | Yes | Yes | Low |
| Static Field | Yes | No* | Lowest |
| Double-Check Locking | Yes | Yes | High |

\* The static field approach creates the instance when the type is first loaded, which may be before it is needed.

My recommendation is to use `Lazy<T>` unless you have a specific reason not to. It's clear, concise and hard to mess up.

## Bonus: Making It Testable

One thing I've found over the years is that singletons can make unit testing really painful because they carry state across tests. A common way around this is to extract an interface and use dependency injection;

```csharp
public interface ISingletonService
{
    string GetValue();
}

public sealed class SingletonService : ISingletonService
{
    private static readonly Lazy<SingletonService> _instance = new(() => new SingletonService());

    public static SingletonService Instance => _instance.Value;

    private SingletonService() { }

    public string GetValue() => "Hello from Singleton";
}

// In your DI container (e.g. ASP.NET Core)
builder.Services.AddSingleton<ISingletonService>(SingletonService.Instance);
```

This way your consuming code depends on `ISingletonService` and can be tested with a mock, while the production code still uses the singleton instance. I've found this works well in practice.

## Summary

So in summary;

- Use `Lazy<T>` for a thread-safe, lazy singleton. It's the simplest and most robust approach in modern C#.
- Use static field initialisation if you don't need lazy creation.
- Avoid double-check locking unless you are maintaining legacy code.
- Use the new `Lock` type in C# 13 instead of locking on `object` when you do need explicit locking.
- Consider dependency injection to keep your code testable.
