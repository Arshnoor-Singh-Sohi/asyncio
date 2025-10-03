# Async Task Scheduler - A Hands-On Asyncio Tutorial

## Project Overview

This project demonstrates concurrent task execution using Python's asyncio library. The application simulates fetching data from multiple sources simultaneously, processing results, and coordinating multiple asynchronous operations. Instead of waiting for each task to complete before starting the next one, asyncio allows us to keep multiple operations "in flight" at the same time, dramatically improving performance for I/O-bound workloads.

**Why asyncio?** When your program spends most of its time waiting for external resources like network responses, database queries, or file operations, asyncio enables you to handle many operations concurrently within a single thread. This is perfect for web scraping, API clients, web servers, and any application that needs to coordinate multiple I/O operations efficiently.

## Introduction to Asyncio

Before diving into the code, let's understand the core concepts that make asyncio powerful.

### What is Asynchronous Programming?

Traditional synchronous code executes line by line. When you make a network request, your program stops and waits for the response before continuing. Asynchronous programming allows your program to start a task and then move on to other work while waiting for that task to complete.

Think of it like a restaurant. In a synchronous restaurant, a waiter takes one order, goes to the kitchen, waits for the food to be prepared, serves it, and only then takes the next order. In an asynchronous restaurant, the waiter takes multiple orders without waiting for each meal to be prepared, and brings out food as it becomes ready.

### Core Asyncio Concepts

**Coroutines**: Functions defined with `async def` are coroutines. They're special functions that can be paused and resumed, allowing other code to run during waiting periods. When you call a coroutine, it returns a coroutine object that needs to be awaited.

```python
async def fetch_data():
    # This is a coroutine
    return "data"
```

**The Event Loop**: This is the heart of asyncio. The event loop manages and executes all asynchronous tasks, deciding which coroutine to run next. Think of it as a traffic controller that keeps all your async operations moving efficiently.

**await**: This keyword tells Python "this operation might take a while, so let other tasks run while we wait." You can only use `await` inside an `async def` function, and you can only await things that are "awaitable" like coroutines, tasks, or futures.

**Tasks**: A task wraps a coroutine and schedules it to run on the event loop. Creating a task allows the coroutine to run concurrently with other tasks.

**Concurrency vs Parallelism**: Asyncio provides concurrency, not parallelism. Concurrency means multiple tasks are making progress during overlapping time periods by switching between them. Parallelism means multiple tasks are literally running at the same instant on multiple CPU cores. Asyncio is single-threaded but can handle thousands of concurrent operations by switching between them during waiting periods.

## Setup Instructions

### Requirements

This project requires Python 3.7 or higher, as asyncio has evolved significantly and modern syntax was introduced in Python 3.7.

### Installation

First, clone this repository or download the source files:

```bash
git clone https://github.com/yourusername/asyncio-tutorial.git
cd asyncio-tutorial
```

Install the required dependencies:

```bash
pip install -r requirements.txt
```

For this tutorial project, we primarily use standard library modules, but you might want aiohttp for real HTTP requests:

```bash
pip install aiohttp
```

### Virtual Environment (Recommended)

Create a virtual environment to keep dependencies isolated:

```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

## Step-by-Step Code Walkthrough

Let's break down how this asyncio application works, starting from the foundation and building up to the complete system.

### The Basic Coroutine

The most fundamental building block is an async function that simulates fetching data:

```python
async def fetch_data(source_id: int, delay: float) -> dict:
    """
    Simulates fetching data from an external source.
    
    In a real application, this might be an HTTP request or database query.
    We use asyncio.sleep() to simulate the I/O wait time without blocking
    other tasks from running.
    """
    print(f"Starting fetch from source {source_id}")
    await asyncio.sleep(delay)  # Simulate network delay
    print(f"Completed fetch from source {source_id}")
    return {"source": source_id, "data": f"Data from {source_id}"}
```

The key insight here is `await asyncio.sleep(delay)`. During this sleep, the event loop is free to run other tasks. If this were regular `time.sleep()`, it would block the entire program.

### Creating and Managing Tasks

To run multiple fetch operations concurrently, we need to create tasks:

```python
async def fetch_all_concurrent(sources: list[int]) -> list[dict]:
    """
    Launches all fetch operations concurrently and waits for all to complete.
    
    Using asyncio.gather() is the most common pattern for running multiple
    coroutines concurrently and collecting all their results.
    """
    tasks = [fetch_data(source_id, random.uniform(1, 3)) for source_id in sources]
    results = await asyncio.gather(*tasks)
    return results
```

The `asyncio.gather()` function takes multiple awaitables and runs them concurrently, returning their results in the same order once all complete. The asterisk `*tasks` unpacks the list into separate arguments.

### Processing Results as They Complete

Sometimes you want to process results as soon as they're available rather than waiting for everything to finish:

```python
async def fetch_and_process_as_completed(sources: list[int]):
    """
    Process results as soon as each fetch completes, rather than waiting
    for all to finish.
    
    This is useful when you want to show progress or start downstream
    processing as soon as possible.
    """
    tasks = [fetch_data(source_id, random.uniform(1, 3)) for source_id in sources]
    
    for coro in asyncio.as_completed(tasks):
        result = await coro
        print(f"Received and processing: {result}")
        await process_result(result)
```

The `asyncio.as_completed()` function yields coroutines in the order they complete, not the order they were created.

### Error Handling in Async Code

Handling errors in async code follows similar patterns to synchronous code, but with some special considerations:

```python
async def fetch_with_retry(source_id: int, max_retries: int = 3) -> dict:
    """
    Attempts to fetch data with automatic retry logic.
    
    Error handling in async code works the same as sync code, but you need
    to be aware that errors can occur at await points.
    """
    for attempt in range(max_retries):
        try:
            return await fetch_data(source_id, random.uniform(1, 2))
        except Exception as e:
            if attempt == max_retries - 1:
                raise
            print(f"Retry {attempt + 1} for source {source_id}")
            await asyncio.sleep(1)  # Wait before retrying
```

### Coordinating Multiple Async Operations

The main coordinator function ties everything together:

```python
async def main():
    """
    The main entry point that coordinates all async operations.
    
    This function demonstrates different patterns for running async code:
    - Sequential operations (one after another)
    - Concurrent operations (all at once)
    - Processing as completed
    """
    print("Starting async task scheduler...\n")
    
    # Example 1: Run tasks concurrently
    sources = [1, 2, 3, 4, 5]
    print("=== Concurrent Fetch ===")
    start_time = asyncio.get_event_loop().time()
    results = await fetch_all_concurrent(sources)
    elapsed = asyncio.get_event_loop().time() - start_time
    print(f"Fetched {len(results)} items in {elapsed:.2f} seconds\n")
    
    # Example 2: Process as completed
    print("=== Process as Completed ===")
    await fetch_and_process_as_completed(sources)
```

### The Entry Point

Python provides `asyncio.run()` as the standard way to run async code:

```python
if __name__ == "__main__":
    asyncio.run(main())
```

This function creates an event loop, runs the coroutine until it completes, and then closes the event loop. It handles all the boilerplate for you and is the recommended entry point for asyncio programs.

## Tutorial Usage Examples

### Basic Execution

Run the main script to see asyncio in action:

```bash
python async_scheduler.py
```

**Expected output:**
```
Starting async task scheduler...

=== Concurrent Fetch ===
Starting fetch from source 1
Starting fetch from source 2
Starting fetch from source 3
Starting fetch from source 4
Starting fetch from source 5
Completed fetch from source 3
Completed fetch from source 1
Completed fetch from source 5
Completed fetch from source 2
Completed fetch from source 4
Fetched 5 items in 2.87 seconds

=== Process as Completed ===
Starting fetch from source 1
...
```

Notice how all five fetches start before any of them complete. This is concurrency in action. If this were synchronous code, each fetch would need to complete before the next one started.

### Comparing Performance

Create a comparison script to see the performance difference:

```bash
python compare_sync_async.py
```

This will show you that five operations that each take two seconds complete in about two seconds with asyncio, versus ten seconds synchronously.

### Interactive Exploration

Try modifying the delays in the code to see how asyncio handles different timing scenarios. Change the `delay` parameter to see how tasks complete in different orders.

## Asyncio Concepts in Action

Let's map the code back to the concepts we learned.

### async/await in Practice

Every function that needs to perform async operations is marked with `async def`. Within these functions, we use `await` before any operation that might take time, allowing the event loop to run other tasks during the wait.

```python
# This is a coroutine that can be paused
async def fetch_data(source_id: int, delay: float):
    # The await keyword pauses this coroutine during the sleep
    await asyncio.sleep(delay)
    return data
```

### Task Creation and Management

When we want operations to run concurrently, we create tasks explicitly or use gathering functions:

```python
# Creates tasks that start running immediately
tasks = [fetch_data(i, 2) for i in range(5)]
results = await asyncio.gather(*tasks)
```

The moment you pass coroutines to `gather()`, they're scheduled on the event loop and start executing concurrently.

### The Event Loop at Work

Although we don't interact with the event loop directly in modern Python, it's constantly working behind the scenes. When you call `asyncio.run(main())`, Python creates an event loop, submits your main coroutine to it, and the loop manages all task scheduling.

### Cooperative Multitasking

Asyncio uses cooperative multitasking, which means tasks must explicitly yield control with `await`. This is different from preemptive multitasking in threading where the OS can interrupt any thread at any time. The advantage is that you have more control and avoid many race conditions, but the downside is that a task that doesn't await will block everything else.

## Project Structure

Here's how the tutorial project is organized:

```
asyncio-tutorial/
├── README.md                    # This comprehensive tutorial
├── requirements.txt             # Python dependencies
├── async_scheduler.py           # Main application demonstrating asyncio
├── compare_sync_async.py        # Performance comparison script
├── examples/
│   ├── basic_coroutine.py      # Simplest async/await example
│   ├── gathering_tasks.py      # Using asyncio.gather()
│   ├── as_completed.py         # Processing results as they arrive
│   └── error_handling.py       # Exception handling patterns
├── advanced/
│   ├── cancellation.py         # Task cancellation
│   ├── timeouts.py             # Handling timeouts
│   └── context_managers.py     # Async context managers
└── tests/
    └── test_async_scheduler.py # Async unit tests
```

Each file in the examples directory is a standalone script that demonstrates one specific concept. Start with `basic_coroutine.py` and work your way through them in order.

## Exercises for Learners

Ready to practice? Try these exercises to deepen your understanding.

### Exercise 1: Add a New Data Source

Add a new coroutine called `fetch_weather_data()` that simulates fetching weather information. Make it take longer than the other fetches and integrate it into the main concurrent fetch operation.

**Hint:** Follow the pattern of `fetch_data()` but customize the return value to include weather-specific information.

### Exercise 2: Implement Timeout Handling

Modify `fetch_data()` to include a timeout. If a fetch takes longer than three seconds, it should raise a `TimeoutError`.

**Hint:** Use `asyncio.wait_for()` to wrap your async operation with a timeout.

### Exercise 3: Create a Task Queue

Build a simple task queue that processes jobs with a limited number of concurrent workers (for example, only three tasks running at once even if you have ten jobs).

**Hint:** Look into `asyncio.Semaphore()` for controlling concurrency limits.

### Exercise 4: Add Progress Reporting

Modify the concurrent fetch to report progress as a percentage as tasks complete.

**Hint:** Use `asyncio.as_completed()` and track how many tasks have finished.

### Exercise 5: Handle Partial Failures

Update `fetch_all_concurrent()` to handle cases where some fetches fail but others succeed. The function should return successful results and a list of failures rather than raising an exception.

**Hint:** Pass `return_exceptions=True` to `asyncio.gather()` and filter the results.

## Advanced Notes

Once you're comfortable with the basics, these advanced topics will help you write more robust async code.

### Task Cancellation

Tasks can be cancelled if they're no longer needed, which is important for resource management:

```python
task = asyncio.create_task(long_running_operation())
# Later, if we need to cancel it:
task.cancel()
try:
    await task
except asyncio.CancelledError:
    print("Task was cancelled")
```

When you cancel a task, it raises `CancelledError` at the next await point. Your coroutines should handle this gracefully, cleaning up any resources they've acquired.

### Timeout Patterns

Always set timeouts for operations that interact with external systems to prevent your program from hanging indefinitely:

```python
try:
    result = await asyncio.wait_for(fetch_data(1, 5), timeout=3.0)
except asyncio.TimeoutError:
    print("Operation took too long")
```

### When to Use asyncio.run() vs Manual Event Loops

For most applications, `asyncio.run()` is the right choice. It handles event loop creation, cleanup, and proper shutdown. Only use manual event loop management when you need fine-grained control, such as in frameworks or when integrating with other async libraries.

**Modern approach (Python 3.7+):**
```python
asyncio.run(main())
```

**Manual approach (rarely needed):**
```python
loop = asyncio.get_event_loop()
try:
    loop.run_until_complete(main())
finally:
    loop.close()
```

### Asyncio with Threading

Sometimes you need to run CPU-bound operations or use synchronous libraries that don't support asyncio. Use `loop.run_in_executor()` to run blocking code in a thread pool without blocking the event loop:

```python
import concurrent.futures

async def cpu_bound_task():
    loop = asyncio.get_event_loop()
    with concurrent.futures.ThreadPoolExecutor() as pool:
        result = await loop.run_in_executor(pool, blocking_function)
    return result
```

### Debugging Async Code

Debugging async code can be tricky. Enable asyncio debug mode to catch common mistakes:

```python
import asyncio
asyncio.run(main(), debug=True)
```

This will warn you about coroutines that were never awaited, tasks that took too long, and other common issues.

### Common Pitfalls to Avoid

**Forgetting to await:** If you call an async function without `await`, you get a coroutine object that never runs. Python will warn you about unawaited coroutines.

**Blocking the event loop:** Never use blocking I/O operations like `time.sleep()` or `requests.get()` in async code. They prevent the event loop from running other tasks. Use `asyncio.sleep()` and async libraries like `aiohttp` instead.

**Creating too many tasks:** While asyncio can handle thousands of concurrent tasks, creating millions will consume too much memory. Use patterns like worker pools or semaphores to limit concurrency.

## Contributing

Contributions are welcome! If you find bugs, have suggestions for improving the tutorial, or want to add new examples, please open an issue or submit a pull request.

### Development Setup

```bash
git clone https://github.com/yourusername/asyncio-tutorial.git
cd asyncio-tutorial
python -m venv venv
source venv/bin/activate
pip install -r requirements-dev.txt
```

### Running Tests

```bash
pytest tests/
```

Please ensure all tests pass and add new tests for any new functionality.

## License

This project is licensed under the MIT License. See the LICENSE file for details.

## Additional Resources

To continue your asyncio journey, explore these resources:

- [Official asyncio documentation](https://docs.python.org/3/library/asyncio.html) - Comprehensive reference with examples
- [Real Python asyncio guide](https://realpython.com/async-io-python/) - Detailed tutorial with practical examples
- [AsyncIO cheat sheet](https://github.com/crazyguitar/pysheeet#asyncio) - Quick reference for common patterns
- [aiohttp documentation](https://docs.aiohttp.org/) - For building async HTTP clients and servers

Remember that asyncio is a tool for I/O-bound concurrency, not a silver bullet. For CPU-bound tasks, consider multiprocessing. For most applications, simple synchronous code is perfectly adequate. Use asyncio when you genuinely need to coordinate many I/O operations efficiently.

Happy async programming!
