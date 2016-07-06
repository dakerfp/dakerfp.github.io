+++
title = "Singletons using std::weak_ptr"
date = "2016-07-07"
tags = ["c++11", "stl", "c++", "design patterns", "smart pointer"]
+++

A common issue is how a resource and initialize it only when needed.
A common C++ pattern to solve this is using the Resource Acquisition Is Initialization aka RAII.
That's what smart pointers do when created and get out of scope.
But what if these resources must be unique?

The first thing that comes in mind is a singleton, but the plain singleton
pattern, which has a static method or function returning a raw pointer usually creates the resource only once.
It is hardly released, or must explicitly released. The solution is to use smart pointers somehow.

The following code is the solution I came out with to solve it:

```
#include <memory>

class Resource {
private:
    // A good API should forbid invalid usage.
    Resource() {
        // initialize
    }

public:
    ~Resource() {
        // release
    }

    static std::shared_ptr<Resource> instance() {
        static std::weak_ptr<Resource> _instance;
        if (auto ptr = _instance.lock()) { // .lock() returns a shared_ptr and increments the refcount
            return ptr;
        }
        // Does not support std::make_shared<Resource> because of
        // the Resource private constructor.
        auto ptr = std::shared_ptr<Resource>(new Resource());
        _instance = ptr;
        return ptr;
    }
};
```

The most interesting aspect is the static weak_ptr which registers the resource,
but does not prevents its release, when all returned shared_ptr release the resource.
