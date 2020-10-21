pthred_attr_t attr
========================
10 attributes can be specified by this type:
- detach state
    whether calling pthread_create with this attr will create a detached thread.
- scope
- stack size (!)
- stack address (!) # all cloned linux tasks can see each other's stack
```plain
pthread_attr_setaffinity_np (3) - set/get CPU affinity attribute in thread attributes object
pthread_attr_setdetachstate (3) - set/get detach state attribute in thread attributes object
pthread_attr_setdetachstate (3p) - set the detachstate attribute
pthread_attr_setguardsize (3) - set/get guard size attribute in thread attributes object
pthread_attr_setguardsize (3p) - set the thread guardsize attribute
pthread_attr_setinheritsched (3) - set/get inherit-scheduler attribute in thread attributes object
pthread_attr_setinheritsched (3p) - set the inheritsched attribute (REALTIME THREADS)
pthread_attr_setschedparam (3) - set/get scheduling parameter attributes in thread attributes object
pthread_attr_setschedparam (3p) - set the schedparam attribute
pthread_attr_setschedpolicy (3) - set/get scheduling policy attribute in thread attributes object
pthread_attr_setschedpolicy (3p) - set the schedpolicy attribute (REALTIME THREADS)
pthread_attr_setscope (3) - set/get contention scope attribute in thread attributes object
pthread_attr_setscope (3p) - set the contentionscope attribute (REALTIME THREADS)
pthread_attr_setstack (3) - set/get stack attributes in thread attributes object
pthread_attr_setstack (3p) - set the stack attribute
pthread_attr_setstackaddr (3) - set/get stack address attribute in thread attributes object
pthread_attr_setstacksize (3) - set/get stack size attribute in thread attributes object
pthread_attr_setstacksize (3p) - set the stacksize attribute
```

detach a thread
============
A thread is either joinable or detached.  Once it's detached,
it cannot be joined or made joinable.

When the main thread returns from `main()` or any threads calls
`exit()`, the detached thread still get terminated: Being detached
only determines what will happen when the detached thread end.


cond & condattr
=================
thread that wishes to block upon a cond must acquire the lock that will be associated with the cond first.
man 3p pthread_cond_timedwait:
    "The application shall ensure that these functions are called with mutex locked by the calling thread."

It's UB to pthread_cond_destroy a cond upon which some threads are being blocked. (man 3p pthread_cond_destroy)

attributes:
    process-shared
        determine whether the cond variable initialzied with this condattr can be shared across processes
        (TODO: linux shm; vma 1 of process A and vma 2 of process B are mapped to the same physical memory ... ?)
    clock
        related to the timing service of a cond variable (e.g. pthread_cond_timedwait() )
        TODO
