# TArray 

## Sorting TArray elements using a Predicate

By default, sorting occurs automatically if the `<` operator is defined in the element type the TArray holds, otherwise a **predicate** should be defined

### Example
For sorting an TArray<FOnlineSessionSearchResult> based on ping, `FOnlineSessionSearchResult` has a member called ping which is set by Unreal Engine when you request for available online session for an online match. 

You would need to sort the match results by ping so that online sessions with the lowest ping would be at the top of the list.

``` C++
TArray<FOnlineSessionSearchResult> SortedSearchResults = SearchResults; // Assume this has different search results
SortedSearchResults.Sort(IsLowerPing); // Predicate function passed by reference

/* The predicate would be the following */
bool static IsLowerPing(const FOnlineSessionSearchResult& ResultA, const FOnlineSessionSearchResult& ResultB)
{
  return ResultA.PingInMs < ResultB.PingInMs; 
};
```

## Removing elements while Iterating without Shrinking

Sometimes we would have to remove from a huge TArray while iterating it. But TArray is a dynamic array which means everytime you remove an element, the array elements have to be moved in memory, which is expensive and waste of time. 
### How to remove elements while iterating it?
- Use `RemoveAtSwap` and `Shrink` functions
- Swap the last element with the element you want to remove using RemoveAtSwap
- RemoveAtSwap function has a 3rd argument, when you pass false to it, the array won't shrink
- Finally, after iterating the array called Shrink on the array

!!! info "Order Of Elements"
    This is only useful in cases where the order of elements in the array does not matter

### Example
Choosing an Actor based on chance, and removing the chosen Actor from the array

``` C++
for (int i = 0; i < OutActors.Num(); i++)
{
	if (FMath::RandRange(1, 100) > ChanceOfChoosingLootSpawnActor) // ChanceOfChoosingLootSpawnActor has a value from 1 to 100, so higher the value, the lesser the chance of choosing this actor to remove
	{
		OutActors.RemoveAtSwap(i, 1, false);
		i--;
    }
}
OutActors.Shrink(); // Finally shrink the array
```

####References

- <https://www.unrealengine.com/en-US/blog/optimizing-tarray-usage-for-performance>
- <https://dev.epicgames.com/documentation/en-us/unreal-engine/array-containers-in-unreal-engine>

