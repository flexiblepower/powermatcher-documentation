The `FlexiblePowerContext` object will be provided by the runtime to each `Agent` through its `setContext()` method. This object can then be used to interact with the environment. Note that this environment could be simulated, such that the timer and scheduling run faster (or possible slower) than real-time.

This object has several functionalities

# Time interaction
- `currentTime()` returns the current system time as a `Date` object (in UTC).
- `currentTimeMillis()` returns the current system time as a long value, this represents the number of milliseconds since 1st January 1970 00:00:00. This is similar to the `System.currentTimeMillis()` method (but will return different results is simulated environments).

# Scheduling
The different scheduling methods are used to perform jobs in the future on a seperate thread pool that is provided by the PowerMatcher runtime. The methods are similar the the ones you will find the the java `ScheduledExecutorService`, but without direct control over the thread pool. Also here the `Measurable<Duration>` type is used to define the duration.
  - `submit(Runnable)`: Submits a Runnable task for execution and returns a Future representing that task.
  - `submit(Runnable, result)`: Submits a Runnable task for execution and returns a Future representing that task.
  - `submit(Callable)`: Submits a value-returning task for execution and returns a Future representing the pending results of the task.
  - `schedule(Callable, delay)`: Creates and executes a ScheduledFuture that becomes enabled after the given delay.
  - `schedule(Runnable, delay)`: Creates and executes a one-shot action that becomes enabled after the given delay.
  - `scheduleAtFixedRate(Runnable, initialDelay, period)`: Creates and executes a periodic action that becomes enabled first after the given initial delay, and subsequently with the given period; that is executions will commence after initialDelay then initialDelay+period, then initialDelay + 2 * period, and so on.
  - `scheduleWithFixedDelay(Runnable, initialDelay, delay)`: Creates and executes a periodic action that becomes enabled first after the given initial delay, and subsequently with the given delay between the termination of one execution and the commencement of the next.