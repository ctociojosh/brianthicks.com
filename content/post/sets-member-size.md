---
date: "2016-11-21T09:00:00-06:00"
title: "Functional Sets, Part 2: Membership and Size"
tags: ["elm"]
featureimage: "/images/trees-by-jakub-sejkora.jpeg"
thumbnail: "/images/trees-by-jakub-sejkora-with-title.png"
section: "Technology"
draft: true

---

[Last time]({{< ref "sets-intro.md" >}}) we started making a set using a binary search tree.
Let's continue by adding more functionality to our set!
Specifically, we're going to answer two questions:

1. Is an item in the set?
2. How big is the set?

And we're going to do it recursively!

<!--more-->

## Quick Recap

Our sets are really [binary search tree](https://en.wikipedia.org/wiki/Binary_search_tree).
We've modelled this data structure in our Elm code like this:

```elm
type Set comparable
    = Set comparable (Set comparable) (Set comparable)
    | Empty
```

We can create empty sets using `empty`, sets with a single item using `singleton`, and bigger sets by using `insert` or `fromList`.

## Balanced Lies

I have a confession to make: our implementation of `insert` (and therefore `fromList`) behaves badly right now.
It's quite naive!
Ideally, if you ran `fromList [1, 2, 3, 4, 5]` you'd get this:

{{< figure src="/images/sets/one-through-five-balanced.png"
           caption="A balanced tree containing the numbers 1 through 5." >}}

But if you run it against the code we implemented last time, you'll get this instead:

{{< figure src="/images/sets/one-through-five-unbalanced.png"
           caption="An unbalanced tree, the result of a naive insert implementation" >}}
           
In a small set, this doesn't make a huge difference.
But it means that the functions we'll implement today won't be as fast as they could be on larger sets.
We're going to fix this later on in the series, don't worry.
For now, we'll just assume we're working with a well-balanced tree.
For 1 through 5, we can get it with `fromList [3, 2, 4, 1, 5]`.

## Membership / Contains

It's one thing to be able to store a set, but it's not much use if we can't tell what we've stored.
What would we want that API to look like?
Probably a function that takes an item and a set and returns a boolean value indicating if the value is in the set, right?

```elm
member 5 oneThroughFive == True

member 8 oneThroughFive == False
```

The core API for sets calls this function "member", but it sometimes makes more sense to call it "contains".
Think of it however makes the most sense for you.
Anyway, here's how we'll implement it:

```elm
member : comparable -> Set comparable -> Bool
member item set =
    case set of
        Empty ->
            False

        Set head left right ->
            if item == head then
                True
            else if item < head then
                member item left
            else
                member item right
```

A general piece of advice when working with recursive functions: start with the base case first.
We're doing that here with `Empty`.
An empty set does not contain any values by definition, so we return `False`.

If we have a non-empty set, we do a couple of checks.
First, if the item and the head are equal we're done, return `True`
This is the base case for non-empty sets.

If the item we're looking for is less than the head, we recurse down the left tree.
Otherwise, we recurse down the right tree.

### Lost and Found

Let's visualize how this works.
Given `oneThroughFive`, we want to find if `5` is in the set:

{{< figure src="/images/sets/one-through-five-balanced.png"
           caption="A set containing the numbers 1 through 5." >}}
           
We hit the `Set` case in our code, and examine the head.
`3` and `5` are not equal, but `5` is greater than `3` so we recurse down the right subtree.

{{< figure src="/images/sets/four-and-five.png"
           caption="A set containing the numbers 4 and 5." >}}
           
We again hit the `Set` case in our code.
`4` and `5` are not equal.
Surprise!
We're closer, but `5` is greater than `4`, so we recurse down the right subtree again.

{{< figure src="/images/sets/singleton-five.png"
           caption="A set containing only 5." >}}
           
This time, the head is `5` and our item is `5`.
They're euqal, so our code returns `True`.
Since we're not modifying the value on it's way back up the tree, we return `True` from the top level.

### Lost and Lost

What happens when we search for a value we don't have?
How about `8`?
We'd take exactly the same path as before, except where we returned `True` we'd see that `5` and `8` are not equal so we'd recurse down the right tree.
When we got there, we'd find that the subtree was `Empty`.

{{< figure src="/images/sets/empty.png"
           caption="An empty set." >}}
           
Our code would return `False`, and we'd get that back out the top.

## Count on Me

We're all good with testing for membership now.
Let's finish off by figuring out how many items are in our set.
In `oneThroughFive`, the answer is `5`.
We'll call the new function `size`.

```elm
size : Set comparable -> Int
size set =
    case set of
        Empty ->
            0

        Set head left right ->
            1 + size left + size right
```

So what are we doing here? 
Remember how in `member` we only ever returned a value, and never modified it on the way back from our recursive calls?
Well, we don't always have to do that!
In fact, it's necessary in this case.

Starting with our base case again, an `Empty` set contains no items.
That's easy, we'll return `0`.

When we have a non-empty set, we know it at least has one item in it.
So we'll say "I have one, plus however big my left and right subtrees are."
The recursive calls add up!

If we do `fromList [8, 15]`, we'll get a set that looks like this:

```elm
Set
    8
    Empty
    (Set 15 Empty Empty)
```

When we call `size eightAndFifteen`, we can substitude the subtrees for our addition:

```elm
1 + size Empty + size (Set 15 Empty Empty)
```

The left subtree will come back with `0`, so we're left with:

```elm
1 + 0 + size (Set 15 Empty Empty)
```

And replacing the second `size` call:

```elm
1 + 0 + (1 + size Empty + size Empty)
```

Again, we know that empty sets have size 0, so:

```elm
1 + 0 + (1 + 0 + 0)
```

When we add all these numbers together, we get `2`, which is the correct answer.
Yay!
When you're working with recursive functions, it can help to write out the values one step at a time.
You'll see exactly where your logic might be breaking down.
It's a very handy debugging tactic!

## Done!

So now we have two more tools in our `Set` toolbox: `member` and `size`.
And *you* have two more tools in your recursive function debugging toolbox:

- Start with the base case first
- Write out and substitute calls step-by-step to debug.

{{< elmSignup >}}