## 1. For every function you wrote (TODO 1 to 19), explain in your own words what it does and give its time complexity in terms of n, the number of scenes. One short paragraph each.

- **TODO 1 (Scene constructor):** If the scene has choices, it allocates an array with exactly numChoices\_ slots and sets every slot to nullptr. An ending allocates nothing, so choices\_ stays nullptr. The slots start as nullptr so the destructor can safely delete a scene that was only partly built. O(1), since a scene has at most 3 slots.

- **TODO 2 (Scene destructor):** Deletes every child in the slots, then the slot array, then the Moment. Deleting a child calls its destructor, so the whole subtree under the scene gets freed. O(n) when called on the root, since every scene is deleted once.

- **TODO 3 (isEnding):** Returns true if the scene has 0 choices. O(1).

- **TODO 4 (getChoice):** Checks that the index is between 0 and numChoices\_ - 1. If it is, it returns that slot. If not, it returns nullptr. O(1).

- **TODO 5 (setChoice):** Same bounds check as getChoice. If the index is valid, it puts the child scene into that slot. If not, it does nothing. O(1).

- **TODO 6 (countScenes):** Recursive. An empty scene (nullptr) counts as 0. Otherwise, it counts 1 for the current scene plus the count from each child. O(n), since every scene is visited once.

- **TODO 7 (countEndings):** Recursive. An ending returns 1. Otherwise, it adds up the endings from each child. O(n).

- **TODO 8 (height):** Recursive. nullptr returns -1. Otherwise, it finds the tallest child and adds 1. An ending has no children, so it returns -1 + 1 = 0. That is why the max starts at -1 instead of 0. O(n).

- **TODO 9 (clear):** Deletes the root, which frees the whole story because the destructor is recursive. Then it sets root\_ to nullptr so the tree does not delete the same memory twice later. O(n).

- **TODO 10 (load):** Opens the file and prints an error if it cannot. Then it reads START_HP, builds the whole story with readScene starting with the label "START", and gives the root to the tree. Returns false if anything fails. O(n), since readScene does all the work.

- **TODO 11 (readScene):** Recursive. Reads the SCENE, HP, LOOT, ITEM, and CHOICES lines. It checks that CHOICES is between 0 and 3 before allocating anything, so a bad file leaks nothing. Then it builds the Moment and the Scene, and for each choice reads the LABEL line and calls itself to build the child. If anything fails, it deletes the scene it was building, which also frees the children already built, and returns nullptr. O(n), since every scene is read once.

- **TODO 12 (askChoice):** Prints "Your choice: " and reads a whole line. It only accepts a single digit from 1 to numChoices, so input like banana, 7, or an empty line prints the reprompt and asks again. Returns 0 if the input ends. O(1) per attempt.

- **TODO 13 (play):** Walks the story from the root. Every time the player enters a scene, it adds the HP and loot, prints the scene, picks up the item if the bag does not already have it, and prints the HP and loot. It stops on GAME OVER or THE END, and otherwise prints the choices and moves to the one the player picks. At the end it prints the final HP, the final loot, and the inventory. O(h · k), where h is the length of the path and k is the size of the bag, because of search. In the worst case that is O(n²), but in practice it is small.

- **TODO 14 (printStats):** Prints the scene count, the ending count, and the height by calling the tree's three functions. O(n).

- **TODO 15 (printSpoilers):** Calls printEndings starting at the root with an empty path. O(n).

- **TODO 16 (printEndings):** Recursive. If the scene is an ending, it prints its path and text, or "(start)" if the root itself is an ending. Otherwise, it calls itself on each child with the path extended by that choice number, going in order from choice 1 to choice 3. O(n).

- **TODO 17 (greedyRun):** A loop. It enters scenes the same way play does, and stops if HP hits 0 or it reaches an ending. Otherwise, it moves to the child with the most loot, and ties go to the lower choice number. It only follows one path, so it does not need recursion. O(h), which is at most O(n).

- **TODO 18 (bestLoot):** Recursive. It enters the scene and returns -1 if HP drops to 0 or less. If the scene is an ending, it saves the path and returns the scene's loot. Otherwise, it tries every child, each with its own path string, and keeps the best total and its path. If every child returns -1, it returns -1. Otherwise, it returns the best child's total plus this scene's loot. O(n).

- **TODO 19 (optimalRun):** Calls bestLoot from the root with startHp\_. If the result is -1, it prints that no path survives. Otherwise, it prints the best path and the loot. O(n).

## 2. Why does each scene allocate exactly numChoices slots instead of declaring Scene<T>\* choices[3]? What does an ending hold in choices\_, and why?

The number of choices in a scene is either 0, 1, 2, or 3, and in practice, most scenes have 0 or 1 choice. Scene<T>\* choices[3] allocates 3 slots for each scene without any regard to its actual number of choices. I have 15 scenes in my story, which means 45 slots allocated in total. The exact-size allocator needs only 14: 3 (root) + 2 (date) + 9 (scenes with 1 choice) + 0 (endings). An ending sets choices\_ to nullptr. It has no choices and thus does not allocate an array in the constructor.

## 3. Walk through your Scene destructor. What gets deleted, and in what order? What would go wrong if choices\_ were deleted first?

First, Iterate through each slot and delete choices\_[i]. This delete operation calls the destructor for the child, which deletes its children first. Hence, the entire tree below the current node is deleted recursively. The slots that have nothing in them do not cause any harm. Then, delete[] choices\_ deletes the slots themselves since they were created using new[]. The [] in delete[] is used because they were allocated using new[]. Finally, delete data\_ deletes the Moment. If the slots had been deleted first, the step 1 operation would try to access choices\_[i] from memory that has already been deleted. This is an after-use delete, and leads to undefined behavior.

## 4. Show the path greedy took and the path optimal took on your story, with the loot for each. Explain why greedy lost. If greedy did not lose, change your story until it does, and explain what you changed.

**Greedy:** path 1-1-1, loot 12, died (HP 15 → -1).
**Optimal:** path 2-1-1-1-1-1, loot 18, reached a conclusion with 12 HP.

How greedy lost: at the top level, greedy chooses only based on the loot from the next scene: "Reply wyd" (5) vs "Ask them on a real date" (2) vs "Leave them on read" (1). Greedy takes the situationship path. This path consists of a series of decisions that cost 4 + 4 + 8 = 16 HP, one more than your starting HP, and so there's no escape for greedy once he makes this choice. Greedy never looks ahead and never checks the HP before making his choice. bestLoot searches all paths, sees that situationship returns -1 (all paths lead to death), and chooses the date path, which starts slow but ends up adding up to 18.

## 5. bestLoot tries every path, yet it runs in O(n) time. Explain why it is not exponential, using what you know about paths and leaves in a tree.

The bestLoot function tries every path, but on a tree there is exactly one parent for every scene, so there is exactly one path from the root to every scene. Recursion makes a call to bestLoot on every scene, and every recursive call performs constant-time computation (enter the scene, loop over its maximum of three children). This results in O(n). By “every path to an ending”, we mean one path per leaf, and a tree can have no more than n leaves (in my case, there are 4). When paths start at the same point, such as 2-1-… and 2-2-…, this portion of the recursion is only performed once: the “date” scene is only examined once, not once per every ending below it. It would only grow exponentially in complexity if we could reach the same scene from multiple parents, as on a graph.

## 6. Which List methods does your game use, and why? What does one search cost as the bag grows?

The play uses search, addFront, size, print, deleteFront. The search function is O(k), where k is the number of elements in the bag. As it is a linked list, it must check each node node-by-node, using operator== on its name. Its complexity increases linearly with the growth of the bag. In my story, the maximum number of items that could be carried in one run of the game is 3 (Coffee Receipt, Group Chat Invite, Relationship Status). Thus, it is cheap. A very large bag should have a better data structure such as hash set, however, this is a standard STL container, not allowed by this assignment.
