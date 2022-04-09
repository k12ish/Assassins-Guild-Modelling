# Statistical modelling for the Assassins' Guild

### Creating the Graph

The graph must meet a few requirements:

- Every player targets three assassins and is targeted by three assassins
- A player cannot target themselves
- The graph cannot be easily reverse-engineered



### Modelling Assassin Activity

Some assassins may successfully execute dozens of targets, but many assassins fail to kill anyone at all.
This empirically follows some sort of [80/20 rule](https://en.wikipedia.org/wiki/Pareto_principle), where the top assassins conduct majority of assassinations.
Hence, we can use a [Pareto Distribution](https://en.wikipedia.org/wiki/Pareto_distribution) to model this behaviour.

### Retargeting Assassins

When an assassin dies, we must modify 7 nodes:

- one node for the dead assassin
- three nodes that attack the assassin
- three nodes that assassin used to attack


Three assassins are missing a target and three targets are missing an assassin. In the simplest case, there's a total of six possible solutions:

```
Problem:

    A -> _
    B -> _
    C -> _
    _ -> D
    _ -> E
    _ -> F

------------------------------------------------------

Solution #1:        Solution #2:        Solution #3:
    A -> D              A -> E              A -> F  
    B -> E              B -> D              B -> D  
    C -> F              C -> F              C -> E  
            
Solution #4:        Solution #5:        Solution #6:
    A -> D              A -> E              A -> F  
    B -> F              B -> F              B -> E  
    C -> E              C -> D              C -> D  
```

Analysis becomes more difficult when there are pre-existing relationships between `ABC` and `DEF`.
Considering the previous example, suppose that `A` also targets `D`. Now solutions #1 and #4 are invalid, since `A` cannot target `D` twice.

Consider a more complicated problem which we solve in a series of intermediary steps:
```
Problem:

    A -> _
    A -> E
    B -> _
    B -> F
    C -> _
    C -> D
    _ -> D
    _ -> E
    _ -> F

------------------------------------------------------

Assassin A targets D:

    A -> D
    A -> E
    B -> _
    B -> F
    C -> _
    C -> D
    _ -> E
    _ -> F

Assassin C targets F:

    A -> D
    A -> E
    B -> _
    B -> F
    C -> F
    C -> D
    _ -> E

Assassin B targets E:

    A -> D
    A -> E
    B -> E
    B -> F
    C -> F
    C -> D

```
Note that there is only one solution to this problem. If we added more constraints, the problem would become unsolveable.

We model this particular problem with a matrix, which is successively simplified down:

```
Problem:

    D E F
    -----
 A |0 0 1
 B |1 1 0
 C |0 1 1

Assassin A targets F.

    D E F
    -----
 A |0 0 0
 B |1 1 0
 C |0 1 0

Assassin C targets E.

    D E F
    -----
 A |0 0 0
 B |1 0 0
 C |0 0 0

Assassin B targets D.

```
