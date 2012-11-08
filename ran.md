This document describes [ran.erl] and its utilities.

Pseudo py-code for [ran.erl]:

```python
def run(N):
    """This is the main entry point to the benchmark.
    
    N is the number of detected cores in the system multiplied by:
       1 for "short" benchmark
       2 for "intermediate" benchmark
       4 for "long" benchmark
    """

    # self() returns a pid of the current process
    Parent = self()
    # Pid list of spawned processes
    PList = []
    for I in range(1, N):
        # Spawn a new process which executes MyFun. "spawn" returns
        # pid of the new process.
        Pid = spawn(lambda: MyFun(Parent))
        # Add its pid to the children array
        PList.append(Pid)

    # Send a tuple (Parent, "go") to each child
    for Pid in PList:
        send((Parent, "go"))

    # receive (Pid, dontcare) from every child. Note: dontcare is a special
    # value in Erlang which accepts anything.
    for Pid in PList:
        # receive function blocks until message is received
        receive((Pid, dontcare))

def MyFun(Parent):
    """This function is called in a new "child" process.

    Parent is the PID of the parent process."""

    # Block (wait) until Parent sends us a message "go"
    receive((Parent, "go"))
    # Send parent a tuple (self, ...)
    send(Parent, (self(), random(100)))

def random(N):
	Len = 100000
    RandList = mk_ranlist(Len, N*2)
    # Splits RandList to two, assigns FirstHalf and SecondHalf
    # accordingly.
    (FirstHalf, SecondHalf) = split_list(N div 2, sorted(RandList))
    return SecondHalf[0]

def mk_ranlist(Len, Max):
    """Create a random list of Len, with a maximum element Max"""

    # Initialize the random number generator
    random.seed(Len, Max, Max * 2)
    Ret = []
    for i in range(1, Len):
        Ret.append(random.uniform(Max))
    return Ret

```

Here is how random.seed and how random.uniform work. The original code is in
[random.erl].

```python

PRIME1 = 30269
PRIME2 = 30307
PRIME3 = 30323

def seed(A1, A2, A3):
    """Initialize random seed, works per process"""

    # "put" puts a value to the process dictionary, which can be
    # later retrieved by "get". Syntax: put(Key, Value).
    put("random_seed", (A1, A2, A3))

def uniform():
    """Return a random float between 0 and 1"""

    # "get" gets a value from the process dictionary. It must be
    # set previously by "put".
    (A1, A2, A3) = get("random_seed")
    B1 = (A1*171) mod PRIME1
    B2 = (A2*172) mod PRIME2
    B3 = (A3*170) mod PRIME3
    put(random_seed, (B1,B2,B3))
    R = B1/PRIME1 + B2/PRIME2 + B3/PRIME3
    return R - trunc(R)

def uniform(N):
    """Return a random value with maximum value of N"""
    return trunc(uniform() * N) + 1

```

### Notes

Erlang does not have arrays, it uses doubly linked list. So `mk_ranlist` should
return a linked list rather than array (which would be more efficient).

[random.erl]: https://github.com/erlang/otp/blob/master/lib/stdlib/src/random.erl#L92
[ran.erl]: https://github.com/softlab-ntua/bencherl/blob/master/bench/ran/src/ran.erl#L52
