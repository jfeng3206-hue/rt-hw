# Homework 3

- what is functional interface

  A functional interface is a special interface with exactly on abstract method, which makes it suitable for using lambda expressions.

- what is default method

  A default method is a function declared inside an interface and contains a pre-written body.

  ```
    public interface Vehicle{
      void start(); //Abstract method

      default void stop(){
        System.out.println("Stopped vehicle"); //Default method
      }
    }
  ```

- what is the difference between Predicate, Supplier, Consumer, Function? 

  - Predicate: takes an input and returns a boolean
  - Consumer: takes an input and returns nothing.
  - Function: takes an input and returns a different result.
  - Supplier: takes no input and returns a result. 

- write a piece of code to use the  Predicate, Supplier, Consumer, Function interface

  ```
    //Predicate
    List<String> names = Arrays.asList("A","B","C");
    Predicate<String> filterNames = name -> name.equals("C");
    names.stream().filter(filterNames).forEach(System.out::println);

    // Consumer
    Consumer<String> printer = name -> System.out.print(name +", ");
    names.forEach(printer);


    //Supplier
    Supplier<List<String>> nameSupplier = () ->{
        return  Arrays.asList("d","e","f");
    };
    nameSupplier.get().forEach(System.out::print);

    //Function
    Function<String, Integer> countNameLength = name -> {
        return name.length();
    };
    names.stream().map(countNameLength).forEach(System.out::println);
  ```

- what is method reference

  A shortcut to refer to an existing method without actually calling it.
  ```
    Arrays.stream(employeeList).forEach(Employee::getSalary);
  ```


- what is CompleteableFuture

  `CompletableFuture` represents a future result of a background computation. It helps write non-blocking and asynchronous codes.

- Default keyword  vs Java default scope

  - Default keyword: provide a default method implementation within an interface

  - Default scope: access level similar to public, private, and protected. Default scoped memebers are accessible only within their own packages.

- Coding: create a list of students, Student Class has name, age, score three fields. 
List<Student> list = new ArrayList<>();

  - use stream api to find all the students’ name starting with ‘A’

    ```studentList.stream().filter(student -> student.getName().startsWith("A")).forEach(System.out::print);```
  - use stream api to get the sum of all the students score
    
    ```double totalScore = studentList.stream().mapToDouble(Student::getScore).sum();```

  - use stream api to find all the students whose sore >= 60


  - use stream api to retrieve all students name
  - use stream api to count the frequency of each age
  - use steam api to count the number of boys girls (groupby, collector.toMap())

intermediate operation vs terminal operation
- Coding: given a char array, use stream api to count the frequency of each char
Steam API: map() vs flatmap();

- how to create a thread( 4 ways, write code)

  - extends Thread Class
  - Implements Runnable
  - Implements Callable
  - Threadpool (Executor Service)

- thread lifecycle, how does thread transfer from one state to another 

  new → runnable → blocked/waiting/timed_waiting/terminated

  - New: created but no execution yet. Moves to the Runnable state by calling `start()`

  - Runnable: ready to run and waiting for the cpu scheduler to allocate processing time.
    - Runnable → Running: os assigns the thread to a cpu core
    - Runnable → Blocked/Waiting: `wait()` or `sleep()`

  - Running: actively executing
   - Running → Runnable：completes its time slice, yields execution, or is preempted by the OS scheduler.
   - Running → Waiting：`wait()` or `sleep()`
   - Running → Blocked: The thread attempts to access a synchronized block or object whose lock is held by another thread
   - Running → Terminated: complete its execution or encounters an unhandled exception
  
  - Blocked/Waiting (Timed Waiting): The thread is alive but currently ineligible to use the CPU. It is either waiting for a monitor lock (Blocked), waiting for another thread to signal an event (Waiting), or waiting for a specific duration of time (Timed Waiting).
    - Blocked→Runnable: The thread acquires the monitor lock it was waiting for.
    - Wating/Timed Waiting→Runnable： The required condition is met (e.g., notify() is called) or the specified sleep time elaps

- how does thread pool work

  A thread pool manages a collection of pre-instantiated, idle threads. When tasks arrive, they are placed in a queue and assigned to available threads.

  Once a thread finishes a task, it doesn't terminate; it returns to the pool to pick up the next task, avoiding costly thread creation and deletion.

  Return Policy? ThreadFactory? Living Span of a Thread

  Check call pool → entering waiting queue → 

- what is the potential problem for the newCachedThreadPool and newFixedThreadPool and why

  - Cached Thread Pool

    Potential problem: thread and memory exhausation.

    Why: this pool creates a new thread for every incoming task if existing threads are busy. Because it technically has an unbounded maximum limit (Integer.MAX_VALUE), a sudden spike in traffic or a slow-running task can spawn thousands of threads. This overwhelming volume causes an OutOfMemoryError (OOM) or crashes the JVM due to heavy CPU context switching
  
  - Fixed Thread Pool

    Potential Problem: Memory leaks and OOM.

    Why: This pool maintains a capped number of active threads but relies on an unbounded task queue (LinkedBlockingQueue). If tasks are submitted faster than the fixed number of threads can process them, the queue will grow indefinitely. This silent queue growth continues until it devours all available system memory

- what is Future

  Future represents the result for an async computation. Future must be called with `.get()`, which blocks the main thread. 

- what is CompletableFuture

  CompletableFuture is also a tool to help write asynchrounous codes. but it is non-blocking and composable. Instead of waiting, we can attach "callbacks" using methods like `.thenApply()` or `.thenAccept()`. 

- Future vs CompletableFuture

  CompletableFuture is more powerful because it supports non-blocking callbacks, and tasks chaining (with`.allOf()` or `.anyOf()`). It also has built-in error handling and manual completion. 


- Lock vs synchronized

  - synchronized: a keyword. automatic release. non-blocking is not supported.

  - Lock: an interface/class. needs manual release using a `finally` block, support non-blocking via `tryLock()`.


- what is wait(), notify(), notifyAll(), join()

  APIs for thread notification and signaling.

  - wait(): tells the current thread to pause execution and release the object's lock. The thread enters a waiting thread and remains there until another thread calls notify or notifyAll on the same object
  - notify(): Wakes up a single thread that is currently waiting on that object's monitor. If multiple threads are waiting, the choice of which thread is awakened is arbitrary. 
  - notifyAll: asks up all threads  currently waiting on that object's monitor. 
  - join(): the thread joins after current execution finishes
  - sleep() & wait(): sleep does not release the lock, wait release the lock. 

