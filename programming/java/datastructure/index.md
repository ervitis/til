# Data structure

## LinkedHashMap, HashMap, SortedMap, TreeMap

![Map class diagram](https://media.geeksforgeeks.org/wp-content/uploads/20200807195934/SortedMap.png)

- Hashmap: offers O(1) lookup and insertion. Its keys are not ordered, and they are unique.
- LinkedHashMap: offers O(1) lookup and insertion. Keys are ordered and its implementation is a double linked data structure.
- SortedMap: is an interface that provides order of insertion to its keys.
- TreeMap: offers O(log n) lookup and insertion. Keys are ordered, so the keys must implement the Comparable interface.

![Comparison](https://media.geeksforgeeks.org/wp-content/uploads/comparisonTable.png)

```
From:
- https://www.geeksforgeeks.org/differences-treemap-hashmap-linkedhashmap-java/
- https://www.softwaretestinghelp.com/linkedhashmap-in-java/
- https://www.softwaretestinghelp.com/hashtable-in-java/
- https://www.geeksforgeeks.org/sortedmap-java-examples/
```