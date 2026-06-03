# Homework 2
- JRE vs JDK vs JVM

  - JVM: Java Virtual Machines. An abstract machine to take Java bytecodes and translate them into machine codes for the local operating system to execute. 

  - JRE: Java Runtime Environment. Package installed to run a Java program. It includes a JVM and other essential Java libraries and codes to support execution.

  - JDK:Java Development Kit. Used to write, develop, or debug the java programs.It is a superset of JRE with the full suite of JRE and development tools like the java compiler (javac), the debugger, and the archiver (jar)

- Java 8 basic data types

  int, long, double, boolean, char, short, float, byte

- Primitive type, reference type

  - Primitive Types: 
    - 8 types (as above). 
    - stored on the stack. 
    - wrapper class. auto-boxing. 
    - has specific default value. when copying a primitive, it is to create a completely independent copy of the value.

  - Reference (non-primitive) types: 
    - objects. Everything that's not primitive. 
    - no actual data; instead they store the memory address (refernce). Default values are null. 
    - Reference stored on the stack, but actual object data resides on the heap. 

- How does JVM work
  
  JVM compiles the java code into bytecodes (`.class` files) and translates the bytecodes into machine codes for the user's local operating system.

  JVM has 3 main phases: class loader, in which it reads compiled bytecode and brings them into JVM's memory; memory area, in which JVM allocates memory to run the program; execution engine, where JVM actually runs the bytecode by translating bytecode into machine languages.

- JVM memory data model

  JVM memory is consisted of 5 sections.

    - Method Area: class-level data, method data and static variables
    - Heap: object 
    - Stack: local variables and partial results
    - PC registers: maintain execution order. memory address of the instruction currently being executed
    - Native method stacks: native methods (written in C/C++)

- How does GC work

  Garbage Collection in Java happens automatically in JVM without the need to manually release memory like in C/C++.

  Garbage Collection in Java automatically reclaims memory by identifying and removing objects that are no longer reachable from any reference. 

- young/old/perm generation

  The heap in JVM are split by object lifespan:
    - Young Generation (Eden + 2 Survivor Space):  Once the young garbage collection happens the object is promoted into the Survivor space 0 and next into the Survivor space 1. If the object is still used at this point the next garbage collection cycle will move it to the Tenured space which means that it is moved to the old generation.
    - Old Generation: holds long-lived objects and is cleaned by the slower Major/Full GC
    - Permanent Generation: stored class metadata and interned strings, but was replaced by Metaspace (in native memory) starting in Java 8.

- difference types of GC

  - Serial: single-threaded, good for small apps or single-CPU environments.
  - Parallel/Throughtput: similar to Serial but work on multiprocessor environments and multi-threaded. Default in Java 8.
  - CMS (Concurrent Mark Sweep):  low-pause collector that works concurrently with the app; deprecated/removed in newer versions.
  - G1 GC – divides heap into regions, balances throughput and pause times; more cpu cycles spent in garbage collection; default since Java 9.
  - ZGC / Shenandoah – experimental ultra-low-latency collectors (sub-millisecond pauses) designed for very large heaps.

- Java modifier scope: public, private, protected, default scope

  - public: can be access anywhere: same class, same package, subclasses, and other packages. 
  - private: can only be accessed within the same class
  - protected: can be accessed within the same package and by subclasses in other packages (via inheritance)
  - default: Accessible only within the same package. Applied when no modifier is specified; subclasses in other packages cannot access it.

- What is static scope

  Static keyword makes an member belong to its class, instead of to any instance, so it's shared across all objects and can be accessed without creating any instance. Static variables are initialized when the class is loaded. It can be applied to variables, methods, blocks, and nested classes.

- How does classloader work

    3 major phases: loading when JVM takes the binary representation of a class or interface and generates the original class or interface, linking where JVM combines the different elements and dependencies of the program together, initialization when JVM executes initialization method of the class, such as calling the constructor, executing the static block.

- Describe the difference between unchecked and checked exceptions in Java

  - Checked Exceptions: checked at compile-time, must be handled using `try-catch` block or `throws`. Typically represent recoverable conditions outside the program's control, like `IOException`, `SQLException`, or `ClassNotFoundException`. Typically some problems the developer should prepare for.  

  - Unchecked Exceptions: checked at runtime and don't require explicit handling or declaration. They extend `RuntimeException` and usually indicate programming bugs like `NullPointerException`, `ArrayIndexOutOfBoundsException`, or `IllegalArgumentException`. Typically bugs in the code or logical errors that developers should fix. 

- What is the difference between finally, final, and finalize in Java?

  - final: restrict modification. final variable cannot be reassigned; a final method cannot be overriden; a final class cannot be extended

  - finally: s a block used with try-catch that always executes whether an exception is thrown or not

  - finalize: a method from Object class called by the GC before reclaiming an object, meant for cleanup of resources—but it's unpredictable, deprecated since Java 9, and removed in newer versions; use try-with-resources or Cleaner instead.

- Define try-with resource. How can you say that it differs from an ordinary try?

  Objects declared in the try(...) parentheses and automatically closed when the block exits, even on exceptions.

  No need for a `finally` block to manually close resources, prevents resource leaks, and handles suppressed exceptions cleanly when both the try block and the close operation throw.

- Define Runtime Exception. Describe it with the help of an example.

  Checked at runtime instead of compile time. Do not require explicit exception handling. Typical examples include: `InvalidUserInputException`, `NullPointerException`, `ArrayIndexOutOfBoundsException`, or `IllegalArgumentException`.

  ```
  int x = 10/0; //Arithmetic Exception
  int[] list = new int[]{1,2};
  System.out.println(list[3]);//ArrayIndexOutOfBoundsException
  ```

- What is the difference between NoClassDefFoundError and ClassNotFoundException in Java

  ClassNotFoundException is a checked exception thrown at runtime when an application tries to load a class and the class isn't found in the classpath. (Happens during class loading)

  NoClassDefFoundError is an Error (unchecked) thrown when a class was present at compile time but missing at runtime — the JVM successfully linked your code expecting the class, but couldn't locate its .class file when actually needed.

- Why should we clean up activities such as I/O resources in the finally block?

  I/O resources will exhaust hardware resources, therefore need to be cleanued up to prevent resource leaks.
  Without the finally block, an exception in the try block would skip the cleanup code, causing resource leaks that can exhaust system handles, lock files, or drain connection pools over time.

- Describe OutofMemoryError in exception handling.

  OutOfMemoryError is an unchecked Error (not an Exception) thrown by the JVM when it cannot allocate memory because the heap is exhausted and the garbage collector can't free enough space.

  Can be fixed by adjusting JVM heap size wiht `-Xmx` or fixing the resource leak.

- What is Generics in Java? What are the advantages of using Generics?

  Generic are templates denoted by symbols like `<E>` in java to ensure data structure safety. Generic templates can have any object type when instantiated.

  It can help prevent exceptions during compile time, and improve code reusability. 


- How does Generics work in Java? What is type erasure?

  Generics in Java work at compile time, but generic type information is removed from the bytecode. This process is called type erasure.

  During erasure, type parameters are replaced with their bounds (or Object if unbounded), so `List<String>` and `List<Integer>` both become List at runtime. 

- What is the difference between List<? extends T> and List<? super T>?

  - List<? extends T>: accepts a list of `T` or any subtype of `T`. You can read item as T, but cannot add because the exact subtype is unkonwn (Producer/read-only).
  - List<? super T>: accepts a list of `T` or any supertype of `T`. You can add T or its subtypes, but reading only gives you Object. (Consumer / write-friendly)

  PECS rule: producer extends, consumer super. 

- Optional Class: the Optional class is a Java 8 feature designed to wrap return values to prevent null pointer exceptions. They highlight common APIs such as `ofNullable` to allow null values, `orElse` to provide default values, and `orElseThrow` to handle exceptions when values are missing 

## Coding

```
 //given a random character array, find the char with third highest frequence
    //input: [a, b, b, c, c, c], output: [a]
    public List<Character> findThirdHighest(char[] chars){
        if(chars==null || chars.length==0) return new ArrayList<>();
        Map<Character, Integer> map = new HashMap<>();
        for(char c: chars){
            map.put(c, map.getOrDefault(c,0)+1);
        }
        List<Map.Entry<Character, Integer>> frequencyList = new ArrayList<>(map.entrySet());
        frequencyList.sort(Comparator.comparingInt(Map.Entry::getValue));

        Set<Integer> seenFreqs = new HashSet<>();
        for(Map.Entry<Character, Integer> e: frequencyList){
            seenFreqs.add(e.getValue());
        }
        if(seenFreqs.size()<3) return new ArrayList<>();
        int thirdFreq = new ArrayList<>(seenFreqs).get(2);
        List<Character> result = new ArrayList<>();
        for(Map.Entry<Character, Integer> e: frequencyList) {
            if(e.getValue()==thirdFreq) result.add(e.getKey());
        }
        return result;
    }


    //reverse a string
    //input: “abc”, output: “cba”
    public String reverseString(String input){
        if(input==null) return null;
        return new StringBuilder(input).reverse().toString();
    }


    //given an integer array and target, return all the pairs sum to the target,
    // each element can only be used once
    //input: [1, 2, 3, 4] target = 5, return [[1, 4],[2, 3]]
    public List<int[]> pairSum(int[] input,int target){
        Map<Integer, Integer> map = new HashMap<>();
        for(int i: input) map.put(i, map.getOrDefault(i,0)+1);
        List<int[]> result = new ArrayList<>();
        for(int x: input){
            int c = target-x;
            if(map.getOrDefault(x,0)==0) continue;
            if(x==c){
                if(map.get(x)>=2){
                    result.add(new int[]{x,x});
                    map.put(x, map.get(x)-2);
                }
            } else if(map.getOrDefault(c,0)>0){
                result.add(new int[]{x,c});
                map.put(x, map.get(x)-1);
                map.put(c,map.get(c)-1);
            }
        }
        return result;
    }
```

