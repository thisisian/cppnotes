# Concurrency notes
## From A Tour of C++

Standard library supports system-level concurrency. 

_task_ - computation which can be executed in parallel with other computations
_thread_ - system-level representation of task

```
void f(); 

void user() {
    thread t1 {f}; // Execute f in separate thread
    t1.join();     // wait for t1
}
```

Passing arguments:

```
void f(vector<double& v); // function does something with v
struct F {
    vector<double>& v;
    F(vector<double>& vv) : v{vv} {}
    void operator()();
};

int main()
{
    vector<double> some_vec {1,2,3,4,5};
    vector<double> vec2 {10,11,12,13,14};

    thread t1 {f, ref(some_vec)}; // f(some_vec) executes in a separate thread
    thread t2 {F(vec2}};          // F(vec2)() executes in a separate thread

    t1.join();
    t2.join();
}
```

This example shows two ways of passing arguments.
`F{vec2}` creates `F` with a `vec2` reference and executes.
`{f, ref(some_vec)}` uses `thread` variadic template. `ref` ensures that we get a reference to `some_vec`. This style creates a function object with necessary information and executes. 
Both these cases are pretty much identical.

We can get a result by passing in a location to store a result separately from the argument we want to modify. 

### Locks
```
mutex m; 
int sh;  // shared data

void f()
{
    scoped_lock lck {m}; // acquire mutex
    sh += 7;
} // lock is implictly release
```

Using `scoped_lock` or `unique_lock` allows for RAII to implicitly unlock the mutex. Safer than explicitly calling `m.lock`.

Good idea to put locks into structure along with the data that is to be modified.

`scoped_lock` allows for enabling of several locks simultaneously, helping to avoid deadlock

`share_mutex` allows for multiple readres to access data

Communication through external events is provided by `condition_variable`s. Allows a thread to wait until some condition is fulfilled.

```
class Message {
    //...
};

queue <Message> mqueue;   // queue of messages
condition_variable mcond; // for communicating events
mutex mmutex;             // for synchronizing access to mcond

void consumer()
{
    while(true) {
        unique_lock lck {mmutex};   // acquire lock
        mcond.wait(lck, [] {return !mqueue.empty();}); // release lck and wait
                                                       // re-acquire lck upon wakeup
                                                       // don't wake up unless mqueue is non-empty
        auto m = mqueue.front();    // get message
        mqueue.pop();
        lck.unlock();               // release lock
    }
}

void producer() 
{
    while(true) {
        Message m;
        // .. fill the message ...
        scoped_lock lck  {mmutex};  // protect operations
        mqueue.push(m);
        mcond.notify_one();         // notify
    }   // lock release at end of scope
}

```

From `consumer`:
- Waiting on the condition variable 

### `future` and `promise`

`future` and `promise` allow transfer of value between two tasks without explicit use of a lock
when a task wants to pass a value to another, it puts the value into a `promise`.
That value appears in a corresponding `future`. 

`X v = fx.get();`
`fx` is a future. If the value isn't in `fx` the thread is blocked until it arrives. 

We put a value into a promise with `px.set_value(val)`. If we want to throw an exception use `px.set_exception(current_exception());`
### `packaged_task`

`packaged_task` is provided to simplify set up of taks connected with `future`s and `promise`s. 




