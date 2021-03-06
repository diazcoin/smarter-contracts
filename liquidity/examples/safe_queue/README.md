This contract contains a map-based queue.  One can add an element to the end of the queue, and remove an element from the front of the queue.  

In this contract the storage is of type record.  It contains the fields:  
- **map** or **big map** of type (int, string).  I.e., the keys are of type int, the elements of the queue are of type string.  See [this](http://www.liquidity-lang.org/doc/reference/liquidity.html#operations-on-maps).  Big maps are a specific kind of maps, optimized for large data.  You can only have one big map per smart contract, and it appears as the first element of the storage.
- **first key**.  Keys are positive integers.  The first key record field stores the key of the first element of the queue. 
- **last key**.  The last key record field stores the key of the last element of the queue.
- **unit**

The parameter is a *string option*.  If the parameter is *None*, then the first item on the queue is removed.  If the parameter is *Some string*, then that string is added to the end of the queue.

To verify that the script implements a queue correctly:

(1) compile safe_queue.liq or safe_queue.liq to .tz (by running ```liquidity safe_queue.liq```)

(2) typecheck the script with
~~~~ 
 tezos-client typecheck script [path to]/safe_queue.tz
~~~~
(3) run a tezos-node in the local network.  Make sure it's bootstrapped.  See [this](http://tezos.gitlab.io/mainnet/introduction/howtouse.html#rpc-interface).

(4) run the tezos-client ```run script``` command with various initial storage and inputs.

The storage or input data have to be in a specific format.  See [this](https://github.com/cryptiumlabs/smarter-contracts/blob/master/liquidity/examples/tezos-clients-data-format.md).

## Big Map

For the big map version, the output on storage is displayed in the form of diff:

- When the queue is empty or has the empty string, removing the first element of the queue (with the parameter of *'None'*) has no effect:

When the queue is empty:

```tezos-client run script safe_queue_big.tz on storage 'Pair {} (Pair 0 (Pair 0 Unit))' and input 'None'```

Output:
~~~~
storage
  (Pair {} (Pair 0 (Pair 0 Unit)))
emitted operations
  
map diff:
~~~~

When the queue has the empty string:

```tezos-client run script [path to]/safe_queue_big.tz on storage 'Pair { Elt 0 "" } (Pair 0 (Pair 0 Unit))' and input 'None'```

Output:

~~~~
storage
  (Pair {} (Pair 0 (Pair 0 Unit)))
emitted operations
 
map diff:
  + 0-> ""
~~~~

- When the parameter is *'(Some "string")'*, the string is added to the end of the queue.  The first and last key counters are updated properly as well.  Note that the storage required is larger and more expensive with a longer string.  

 ```tezos-client run script [path to]/safe_queue_big.tz on storage 'Pair { Elt 0 "" } (Pair 0 (Pair 0 Unit))' and input '(Some "a")'```

Output:

~~~~
storage
  (Pair {} (Pair 1 (Pair 1 Unit)))
emitted operations
 
map diff:
  + 0-> ""
  + 1-> "a"
~~~~

- The last key counter properly goes up by 1 when another string is added.

```tezos-client run script [path to]/safe_queue_big.tz on storage '(Pair {Elt 0 "";Elt 1 "a"} (Pair 1 (Pair 1 Unit)))' and input '(Some "b")'```

Output:

~~~
storage
  (Pair {} (Pair 1 (Pair 2 Unit)))
emitted operations
 
map diff:
  + 0-> ""
  + 1-> "a"
  + 2-> "b"
~~~~

- The first element is properly removed and the first key counter properly goes up by 1.

```tezos-client run script [path to]/safe_queue_big.tz on storage '(Pair {Elt 0 "";Elt 1 "a";Elt 2 "b"} (Pair 1 (Pair 2 Unit)))' and input 'None'```

Output: 

~~~~
storage
  (Pair {} (Pair 2 (Pair 2 Unit)))
emitted operations
 
map diff:
  + 0-> ""
  - 1
  + 2-> "b"
~~~~

## Map

For the map version, it works similarly, except the queue is displayed directly:

```tezos-client run script safe_queue.tz on storage '(Pair { Elt 1 "a" } (Pair 1 (Pair 1 Unit)))' and input 'None'```

Output:
~~~~
storage
  (Pair {} (Pair 2 (Pair 1 Unit)))
emitted operations
~~~~

Removing the first element again gives the same output, as desired:

```tezos-client run script safe_queue.tz on storage '(Pair {} (Pair 2 (Pair 1 Unit)))' and input 'None'```

Output:
~~~~
storage
  (Pair {} (Pair 2 (Pair 1 Unit)))
emitted operations
