# Home

## Active Objects for Modern C++

The [`activecpp`](https://github.com/samiralavi/activecpp) repository provides an implementation of the Active Object design pattern in modern C++ (C++17 and newer). The primary purpose of this repository is to facilitate asynchronous and concurrent programming by decoupling method invocation from method execution. This is achieved by encapsulating an object's state and behavior in a separate thread, allowing clients to interact with the object through message passing.

### Key Features:
- **Single Header Implementation**: The entire implementation is contained within a single header file, making it easy to integrate.
- **Message Passing**: Supports asynchronous delivery of messages of any type, maintaining the order in which they were sent.
- **Lock-Free Queue**: Utilizes an internal lock-free MPSC (Multiple Producer, Single Consumer) queue for efficient message handling.
- **Callbacks and Timers**: Provides mechanisms for invoking callbacks on clients of unknown types and managing timers with client-driven handlers.
- **Performance**: Designed for high performance, capable of handling several million messages per second between threads on ordinary hardware.

### Usage:
- **Inheritance**: Users create active objects by inheriting from the `ActorThread` template class.
- **Message Handling**: Implement `onMessage` methods to handle different types of messages.
- **Thread Management**: Use provided methods to manage the lifecycle of the active object, including starting, stopping, and sending messages.

### Example:
This example demonstrates the basic usage of `activecpp` to create an active object (consumer) that processes messages sent by a producer in a separate thread.

```cpp
#include <samiralavi/activecpp.h>

#include <string>
#include <iostream>
#include <thread>
#include <chrono>

using namespace samiralavi;

// Define a message structure
struct Message {
    std::string description;
};

// Consumer class inheriting from ActorThread
class Consumer : public ActorThread<Consumer> {
    friend ActorThread<Consumer>;

    // Method to handle incoming messages
    void onMessage(Message& msg) {
        std::cout << "Consumer thread " << std::this_thread::get_id()
                  << " received message: " << msg.description << std::endl;
    }
};

// Producer function to send messages to the consumer
void producer(Consumer& consumer) {
    for (int i = 0; i < 5; ++i) {
        // Create a message
        Message msg{"Message " + std::to_string(i + 1)};
        
        // Post the message to the consumer
        consumer.post(std::move(msg));
        
        // Sleep for a while to simulate work
        std::this_thread::sleep_for(std::chrono::milliseconds(500));
    }
}

int main() {
    // Create a consumer object
    Consumer consumer;

    // Start the consumer thread
    consumer.start();

    // Create a producer thread
    std::thread producerThread(producer, std::ref(consumer));

    // Wait for the producer thread to finish
    producerThread.join();

    // Stop the consumer thread
    consumer.stop();

    return 0;
}
```

### Explanation:

1. **Message Structure**:
   - A simple `Message`struct is defined to hold the message data.

2. **Consumer Class**:
   - The `Consumer` class inherits from `ActorThread<Consumer>`.
   - It implements the `onMessage` method to handle incoming `Message` objects.

3. **Producer Function**:
   - The `producer` function sends a series of messages to the consumer.
   - It uses the `post` method to send messages asynchronously.
   - The function simulates work by sleeping for 500 milliseconds between messages.

4. **Main Function**:
   - A `Consumer` object is created and started.
   - A producer thread is created to send messages to the consumer.
   - The main thread waits for the producer thread to finish.
   - Finally, the consumer thread is stopped.


### Documentation and Build:
- **Documentation**: Comprehensive documentation is available, including design patterns and usage examples.
- **Build System**: Uses CMake for building and supports unit tests and examples.

For more details, refer to the [`README.md`](https://github.com/samiralavi/activecpp) and the documentation.
