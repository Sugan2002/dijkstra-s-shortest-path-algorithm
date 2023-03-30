### EX NO : 03
### DATE  : 10-05-2022
# <p align="center"> Dijkstra's Shortest Path Algorithm </p>

## AIM

To develop a code to find the shortest route from the source to the destination point using Dijkstra's shortest path algorithm.

## THEORY
Explain the problem statement

## DESIGN STEPS

### STEP 1:
Identify a location in the google map:

### STEP 2:
Select a specific number of nodes with distance

### STEP 3:
Import required packages.

### STEP 4:
Include each node and its distance separately in the dictionary data structure.

### STEP 5:
End of program.

</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>


## ROUTE MAP
![AI_MAPF](https://user-images.githubusercontent.com/77089743/167614030-fbd72bc8-2275-4bd9-a99c-bb70837bc7ef.PNG)



## PROGRAM
```

# Project done by
# Student name : P.Suganya
# Reg No 212220230049
# Department of Artificial Intelligence and Datascience,
# Saveetha Engineering College. 602105. India.
```
```python
%matplotlib inline
import matplotlib.pyplot as plt
import random
import math
import sys
from collections import defaultdict, deque, Counter
from itertools import combinations
```
### Problems

#### This is the abstract class. Specific problem domains will subclass this.

```python
class Problem(object):
    """The abstract class for a formal problem. A new domain subclasses this,
    overriding `actions` and `results`, and perhaps other methods.
    The default heuristic is 0 and the default action cost is 1 for all states.
    When yiou create an instance of a subclass, specify `initial`, and `goal` states 
    (or give an `is_goal` method) and perhaps other keyword args for the subclass."""

    def __init__(self, initial=None, goal=None, **kwds): 
        self.__dict__.update(initial=initial, goal=goal, **kwds) 
        
    def actions(self, state):        
        raise NotImplementedError
    def result(self, state, action): 
        raise NotImplementedError
    def is_goal(self, state):        
        return state == self.goal
    def action_cost(self, s, a, s1): 
        return 1
    
    def __str__(self):
        return '{0}({1}, {2})'.format(
            type(self).__name__, self.initial, self.goal)
```
### Nodes

#### This is the Node in the search tree. Helper functions (expand, path_actions, path_states) use this Node class.
```python
class Node:
    "A Node in a search tree."
    def __init__(self, state, parent=None, action=None, path_cost=0):
        self.__dict__.update(state=state, parent=parent, action=action, path_cost=path_cost)

    def __str__(self): 
        return '<{0}>'.format(self.state)
    def __len__(self): 
        return 0 if self.parent is None else (1 + len(self.parent))
    def __lt__(self, other): 
        return self.path_cost < other.path_cost

failure = Node('failure', path_cost=math.inf) # Indicates an algorithm couldn't find a solution.
cutoff  = Node('cutoff',  path_cost=math.inf) # Indicates iterative deepening search was cut off.
```

### Helper functions
```python
def expand(problem, node):
    "Expand a node, generating the children nodes."
    s = node.state
    for action in problem.actions(s):
        s1 = problem.result(s, action)
        cost = node.path_cost + problem.action_cost(s, action, s1)
        yield Node(s1, node, action, cost)
        

def path_actions(node):
    "The sequence of actions to get to this node."
    if node.parent is None:
        return []  
    return path_actions(node.parent) + [node.action]


def path_states(node):
    "The sequence of states to get to this node."
    if node in (cutoff, failure, None): 
        return []
    return path_states(node.parent) + [node.state]
```

### Search Algorithm : Dijkstra's shortest path algorithm
```python
class PriorityQueue:
    """A queue in which the item with minimum f(item) is always popped first."""

    def __init__(self, items=(), key=lambda x: x): 
        self.key = key
        self.items = [] # a heap of (score, item) pairs
        for item in items:
            self.add(item)
         
    def add(self, item):
        """Add item to the queuez."""
        pair = (self.key(item), item)
        heapq.heappush(self.items, pair)

    def pop(self):
        """Pop and return the item with min f(item) value."""
        return heapq.heappop(self.items)[1]
    
    def top(self): return self.items[0][1]

    def __len__(self): return len(self.items)
    
    def best_first_search(problem, f):
    "Search nodes with minimum f(node) value first."
    node = Node(problem.initial)
    frontier = PriorityQueue([node], key=f)
    reached = {problem.initial: node}
    while frontier:
        node = frontier.pop()
        if problem.is_goal(node.state):
            return node
        for child in expand(problem, node):
            s = child.state
            if s not in reached or child.path_cost < reached[s].path_cost:
                reached[s] = child
                frontier.add(child)
    return failure

def g(n): 
    return n.path_cost
```
### Route Finding Problems
```python
class RouteProblem(Problem):
    """A problem to find a route between locations on a `Map`.
    Create a problem with RouteProblem(start, goal, map=Map(...)}).
    States are the vertexes in the Map graph; actions are destination states."""
    
    def actions(self, state): 
        """The places neighboring `state`."""
        return self.map.neighbors[state]
    
    def result(self, state, action):
        """Go to the `action` place, if the map says that is possible."""
        return action if action in self.map.neighbors[state] else state
    
    def action_cost(self, s, action, s1):
        """The distance (cost) to go from s to s1."""
        return self.map.distances[s, s1]
    
    def h(self, node):
        "Straight-line distance between state and the goal."
        locs = self.map.locations
        return straight_line_distance(locs[node.state], locs[self.goal])
       

class Map:
    """A map of places in a 2D world: a graph with vertexes and links between them. 
    In `Map(links, locations)`, `links` can be either [(v1, v2)...] pairs, 
    or a {(v1, v2): distance...} dict. Optional `locations` can be {v1: (x, y)} 
    If `directed=False` then for every (v1, v2) link, we add a (v2, v1) link."""

    def __init__(self, links, locations=None, directed=False):
        if not hasattr(links, 'items'): # Distances are 1 by default
            links = {link: 1 for link in links}
        if not directed:
            for (v1, v2) in list(links):
                links[v2, v1] = links[v1, v2]
        self.distances = links
        self.neighbors = multimap(links)
        self.locations = locations or defaultdict(lambda: (0, 0))

        
def multimap(pairs) -> dict:
    "Given (key, val) pairs, make a dict of {key: [val,...]}."
    result = defaultdict(list)
    for key, val in pairs:
        result[key].append(val)
    return result
   

# Create your own map and define the nodes

saveetha_nearby_locations = Map(
    {('Arakkonam', 'Thakkolam'):  13, ('Thakkolam', 'Perambakkam'): 12, ('Thakkolam', 'Tirumalpur'): 13, ('Perambakkam', 'Mappedu'): 8, ('Mappedu', 'Sengadu'): 40, 
     ('Mappedu', 'sunguvarchatram'): 30, ('Sengadu', 'Kuthambakkam'):  15, ('Sengadu', 'sriperumbudur'): 13, ('Kuthambakkam', 'Irungattukottai'): 9, ('Irungattukottai', 'sriperumbudur'): 8, 
     ('sriperumbudur', 'Vattambakkam'): 16, ('sriperumbudur', 'sunguvarchatram'):  9, ('Vattambakkam', 'SingaperumalKoil'): 11, ('Vattambakkam', 'Kattavakkam'): 21, 
     ('Vattambakkam', 'SingaperumalKoil'): 11, ('SingaperumalKoil', 'Chengalpattu'): 10, ('SingaperumalKoil', 'Palur'): 18, ('Palur', 'Chengalpattu'): 13,
     ('Palur', 'Kattavakkam'): 15, ('sunguvarchatram', 'Kattavakkam'): 13, ('Kattavakkam', 'Walajabad'): 4, ('Walajabad', 'Kanchipuram'): 15, ('Kanchipuram', 'Tirumalpur'): 17})


r0 = RouteProblem('Arakkonam', 'sunguvarchatram', map=saveetha_nearby_locations)
r1 = RouteProblem('Thakkolam', 'Palur', map=saveetha_nearby_locations)
r2 = RouteProblem('Perambakkam', 'Kattavakkam', map=saveetha_nearby_locations)
r3 = RouteProblem('Kuthambakkam', 'Palur', map=saveetha_nearby_locations)
r4 = RouteProblem('Mappedu', 'Walajabad', map=saveetha_nearby_locations)

goal_state_path=best_first_search(r4,g)

print("GoalStateWithPath:{0}".format(goal_state_path))

path_states(goal_state_path) 

print("Total Distance={0} Kilometers".format(goal_state_path.path_cost))

```


## OUTPUT:
![AI_Exp3_Output](https://user-images.githubusercontent.com/77089743/167748845-d7ef01f3-fa80-465e-9fb8-a4fb04960bcf.PNG)


## RESULT:
    Thus the program developed for finding route with drawn map and finding its shortest distance covered.
