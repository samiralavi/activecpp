[![GCC Build & Test](https://github.com/samiralavi/activecpp/actions/workflows/gcc.yml/badge.svg)](https://github.com/samiralavi/activecpp/actions/workflows/gcc.yml)
[![Clang Build & Test](https://github.com/samiralavi/activecpp/actions/workflows/clang.yml/badge.svg)](https://github.com/samiralavi/activecpp/actions/workflows/clang.yml)
[![Documentation](https://github.com/samiralavi/activecpp/actions/workflows/docs.yaml/badge.svg)](https://github.com/samiralavi/activecpp/actions/workflows/docs.yaml)

# Active Objects for Modern C++

## ActorThread: Active object pattern in C++

Implementation of the
[Active Object pattern](https://samiralavi.github.io/activecpp/main/active-object-pattern/) 
wrapping a standard C++ thread.

### Simple
* Whole implementation contained in **a single header file!**
* Inherit from a template and you are done. See the tiny example bellow.

### Main features
* Exchange messages of any type (does not requires them to derive from a common base class)
* Messages are asynchronously delivered in the same order they were sent
* Allows to invoke callbacks on clients of unknown type (useful for libraries)
* Callbacks on the active object *auto-store themselves* with no boilerplate code
* Timers ability with *client-driven handlers* (no need for handler&harr;object resolving maps)

### Performance
* Internal lock-free MPSC messages queue
* Extensive internal use of move semantics supporting delivery of non-copiable objects 
* Several million msg/sec between each two threads (both Linux and Windows) in ordinary hardware

### Robustness
* The wrapped thread lifecycle overlaps and is driven by the object existence
* The object is kept alive by smart pointers (whoever has a reference can safely send messages)
* No internal strong references (only the final users determine the destruction/end)
* Nonetheless, callbacks onto already deleted active objects do not crash the application

### Minimum compiler required (C++17 and newer)
* Mininum gcc version supported is 8.
* Works with clang 5 and Visual Studio 2017.
* Clean, standard C++17 (no conditional code, same implementation for all platforms)

### Example

```cpp
#include <samiralavi/activecpp.h>

#include <string>
#include <iostream>
#include <sstream>

using namespace samiralavi;

struct Message { std::string description; };
struct OtherMessage { double beautifulness; };

class Consumer : public ActorThread<Consumer>
{
    friend ActorThread<Consumer>;
    void onMessage(Message& p)
    {
        std::cout << "thread " << std::this_thread::get_id()
                  << " receiving " << p.description << std::endl;
    }
    void onMessage(OtherMessage& p)
    {
        std::cout << "thread " << std::this_thread::get_id()
                  << " receiving " << p.beautifulness << std::endl;
    }
};

class Producer : public ActorThread<Producer>
{
    friend ActorThread<Producer>;
    std::shared_ptr<Consumer> consumer;
    void onMessage(std::shared_ptr<Consumer>& c)
    {
        consumer = c;
        timerStart(true, std::chrono::milliseconds(250), TimerCycle::Periodic);
        timerStart(3.14, std::chrono::milliseconds(333), TimerCycle::Periodic);
    }
    void onTimer(const bool&)
    {
        std::ostringstream report;
        report << "test from thread " << std::this_thread::get_id();
        consumer->send(Message { report.str() });
    }
    void onTimer(const double& value)
    {
        consumer->send(OtherMessage { value });
    }
};

class Application : public ActorThread<Application>
{
    friend ActorThread<Application>;
    void onStart()
    {
        auto consumer = Consumer::create(); // spawn new threads
        auto producer = Producer::create();
        producer->send(consumer);
        std::this_thread::sleep_for(std::chrono::seconds(3));
        stop();
    }
};

int main()
{
    return Application::run(); // re-use existing thread
}
```
Despite received by reference, *a copy* of the original object is delivered to the destination thread.
Alternatively, the carried object can be moved, which is the way to transfer non-copiable objects like a unique_ptr.
Copying is highly efficient with pointers, but note that several threads must not concurrently access a unsafe pointed object.

See the [*examples*](examples/) folder for more elaborated examples, including a library and its client using the callbacks mechanism.
