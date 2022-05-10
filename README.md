# Dijkstra's Shortest Path Algorithm
## AIM

To develop a code to find the shortest route from the source to the destination point using Dijkstra's shortest path algorithm.

## THEORY
Explain the problem statement

## DESIGN STEPS

### STEP 1:
Identify a location in the google map:

### STEP 2:
Select a specific number of nodes with distance

### STEP -> Write your own steps:


## ROUTE MAP
![AI_MAPF](https://user-images.githubusercontent.com/77089743/167614030-fbd72bc8-2275-4bd9-a99c-bb70837bc7ef.PNG)



## PROGRAM
```
# Prepared by 
# C. Obed Otto, 
# Department of Artificial Intelligence and Datascience,
# Saveetha Engineering College. 602105. India.

# Experiment done by
# Student name : P.Suganya
Reg No 212220230049
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
   def __init__(self, state, parent=None, action=None, path_cost=0):
       self.__dict__.update(state=state, parent=parent, action=action, path_cost=path_cost)

   def __str__(self): 
       return '<{0}>'.format(self.state)
   def __len__(self): 
       return 0 if self.parent is None else (1 + len(self.parent))
   def __lt__(self, other): 
       return self.path_cost < other.path_cost

failure = Node('failure', path_cost=math.inf) 
cutoff  = Node('cutoff',  path_cost=math.inf)
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

FIFOQueue = deque
```

### Search Algorithm :  Breadth First Search
```python
def breadth_first_search(problem):
   "Search shallowest nodes in the search tree first."
   node = Node(problem.initial)
   if problem.is_goal(problem.initial):
       return node
   frontier = FIFOQueue([node])
   reached = {problem.initial}
   while frontier:
       node = frontier.pop()
       for child in expand(problem, node):
           s = child.state
           if problem.is_goal(s):
               return child
           if s not in reached:
               reached.add(s)
               frontier.appendleft(child)
   return failure
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
   

nearby_locations = Map(
    {('Sholinghur', 'Jyotipuram'):  10, ('Jyotipuram', 'Salai'): 6, ('Salai', 'Itchiputhur'): 12, ('Salai', 'Arakkonam'): 12, ('Salai', 'Patittur'): 8, 
     ('Itchiputhur', 'Thirutani'): 10, ('Thirutani', 'Sathuranjayapuram'):  11, ('Sathuranjayapuram', 'Kanakammachathram'): 12, ('Kanakammachathram', 'Poondi'): 20, 
     ('Kanakammachathram', 'Thiruvalangadu'): 16, ('Thiruvalangadu', 'Arakkonam'): 21, ('Thiruvalangadu', 'Tiruvallur'):  28, ('Arakkonam', 'Manavur'): 25, 
     ('Manavur', 'Tiruvallur'): 15, ('Manavur', 'Polivakkam'): 18, ('Tiruvallur', 'Veppambattu'): 13, ('Veppambattu', 'Avadi'): 12, ('Avadi', 'Ambattur'): 10, 
     ('Polivakkam', 'Perumalpattu'): 20, ('Perumalpattu', 'Avadi'): 18, ('Tiruvallur', 'Veppambattu'): 13, ('Salai', 'Arakkonam'): 12 , ('Arakkonam','Thiruvalangadu'): 21 , 
     ('Arakkonam', 'Manavur'): 25, ('Salai', 'Patittur'): 8, ('Patittur','Thakkolam'): 9, ('Thakkolam', 'Perambakkam'): 12, ('Perambakkam', 'Polivakkam'): 16, ('Perambakkam', 'Kuthambakkam'): 18, ('Kuthambakkam', 'Poonamallee'): 13, ('Poonamallee','Amabattur'): 16,})


r0 = RouteProblem('Sholinghur', 'Poondi', map=nearby_locations)
r1 = RouteProblem('Sholinghur', 'Ambattur', map=nearby_locations)
r2 = RouteProblem('Arakkonam', 'Poonamallee', map=nearby_locations)
r3 = RouteProblem('Manavur', 'Poonamallee', map=nearby_locations)
r4 = RouteProblem('Thiruvalangadu', 'Ambattur', map=nearby_locations)

goal_state_path=breadth_first_search(r1)

print("GoalStateWithPath:{0}".format(goal_state_path))

path_states(goal_state_path) 

print("Total Distance={0} Kilometers".format(goal_state_path.path_cost))

```


## OUTPUT:

Justify that the algorithm always returns the shortest path:

## RESULT:


