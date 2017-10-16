---
title: answering-slack-questions-1
tags: Help Recursion
date: 2017-10-15 20:16:03
---

I "lurk" in a couple of general development chat rooms, and I try to answer questions from time to time. Today I started to answer someone's question on recursion. I fortunately thought to draft my response offline as my post became lengthy. I boiled my draft down to a more direct, slack-appropriate answer, but I finished my draft to this post. I hope this may help anyone working on understanding recursion with logic operators.

## The message

```
Dev [12:18 AM] 
Hi guys I have a problem with understanding recursion function. Can someone explain what happens after find(33) part in this http://jsbin.com/revajurica/edit?js,console
```

At the link is the following code:

```javascript
function findSolution(target) {
  function find(current, history) {
    if (current == target)
      return history;
    else if (current > target)
      return null;
    else
      return find(current + 5, "(" + history + " + 5)") ||
             find(current * 3, "(" + history + " * 3)");
  }
  return find(1, "1");
}

console.log(findSolution(13));

/*

Working 

find(1, "1")
  find(6, "(1 + 5)")
    find(11, "((1 + 5) + 5)")
      find(16, "(((1 + 5) + 5) + 5)")
        too big
      find(33, "(((1 + 5) + 5) * 3)") //why does this remove +5 from earlier and not *3 it just called
        too big 
    find(18, "((1 + 5) * 3)")
      too big
  find(3, "(1 * 3)")
    find(8, "((1 * 3) + 5)")
      find(13, "(((1 * 3) + 5) + 5)")
        found!
        
*/
```

## Breakdown

Lets start to break this down.

`findSolution(target)` defines the function `find(current, history)` within its scope, and then calls `return find(1, "1")`.

`find` checks if the its `current` argument loosely equals the `target` argument from the parent scope. If it does, then the `history` argument is returned. `else if` `current` is grater than `target`, then `null` is returned. `else` return `find(current + 5, "(" + history + " + 5)") || find(current * 3, "(" + history + " * 3)");`. 

If we think about the "truethiness" of `null`, we can think think of the code in the `else if` block of `find` like the following.

```javascript
function findSolution(target) {
  function find(current, history) {
    if (current == target)
      return history;
    else if (current > target)
      // replace null with truthiness for simplicity
      return false;
    else
      return find(current + 5, "(" + history + " + 5)") ||
             find(current * 3, "(" + history + " * 3)");
  }
  return find(1, "1");
}

console.log(findSolution(13));
```

Additionally, if we look closely at the code, we can see that `history` is a string that keeps track of which part of the logical statement was returned. If `history` is not returned, then the operation is not added to `history`. For our purposes, we can ignore it and think of the code like the following.

```javascript
function findSolution(target) {
  function find(current) {
    if (current == target)
      // history replaced with its "truethiness"
      return true;
    else if (current > target)
      return false;
    else
      // remove history for simplicity
      return find(current + 5) || find(current * 3);
  }
  return find(1);
}

console.log(findSolution(13));
```

### Logical operators at evaluation

The last piece of the puzzle is understanding how JavaScript evaluates `return find(current + 5) || find(current * 3);`.

`||` is the "or" logical operator. In JavaScript, the left-hand expression will be evaluated first and then checked for its "truthiness". If the value of the left-hand expression is "truthy" then the value will be return (not its truthiness), and the right-hand expression will not be evaluated. However, if the left-hand expression is not "truthy", then the right-hand expression will be evaluated and the value of the right-hand expression will be returned.


### Walkthrough of simplified code

So what happens? 

1. 13 gets plugged into `findSolution`. `find(1)` gets called. `!(1 == 13)` and `!(1 > 13)`, so then the we try `find(1 + 5)`.

2. `find(6)` gets called. `!(6 == 13)` and `!(6 > 13)`, so we try `find(6 + 5)`.

3. `find(11)` gets called. `!(11 == 13)` and `!(11 > 13)`, so we try `find(11 + 5)`.

4. `find(16)` gets called. `!(16 == 13)` but `(16 > 13)`, so we return false! Since we returned false, we have to evaluate the right-hand expression of the previous step (`find(11 * 3)`).

5. `find(33)` gets called. `!(16 == 13)` but `(16 > 13)`, so we return false! Since we returned false, we have to evaluate the right-hand expression of the previous step (`find(6 * 3)`).

6. `find(18)` gets called. `!(18 == 13)` but `(18 > 13)`, so we return false! Since we returned false, we have to evaluate the right-hand expression of the previous step (`find(1 * 3)`).

7. `find(3)` gets called. `!(3 == 13)` and `!(3 > 13)`, so we try `find(3 +5)`.

8. `find(8)` gets called. `!(8 == 13)` and `!(8 > 13)`, so we try `find(8 + 5)`.

9. `find(13)` gets called. `13 == 13`! so we return true and end. 

If we trace back all the returned "truthy true" values in order that returned our final call to find, we can build a "history" of `(((1 * 3) + 5) + 5)`.

### Answer to comment in code

The reason the "+ 5" gets "removed" from history is because that history value never gets returned. In step 4 above, we walk through `find(16)` having truthiness of false. Since the truthiness of the expression is false, `||` returns the right-hand expression's value from `return find(11 + 5) || find(11 * 3)`.

In step 5 we see `find(33)` also has a truthiness of "false", so we evaluate the right-hand side of an even earlier step `return find(6 + 5) || find(6 * 3)`.

This behavior continues until a complete "truthy path" is found and the corresponding history is returned. 
