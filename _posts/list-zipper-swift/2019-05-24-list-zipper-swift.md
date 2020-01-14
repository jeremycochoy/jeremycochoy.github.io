---
layout: post
title: List Zipper applied to iOS swift
description: A simple and elegant implementation of the list zipper data structure.
author: Jérémy Cochoy
date: 2019-05-24 +0100
categories: swift ios programming functional language
lang: en
...

Recently, I was implementing a model for a music application where the user
edit a collection of tracks, while editing only one track at a time.
After a little thought, the best representation of this “focus” in the data
model was obviously a zipper.
Since this is not a common pattern, I thought it would be a wonderful example
of how such a simple pattern can encode the intention into the data structure.

## Abstract idea
The Zipper is a well known pattern in Haskell,
and although this language is rarely used in production,
some of its usage translate very well to other languages.
The List Zipper is basically a list where the element we are focusing at
was taken out.
It allow shifting the attention to the previous or next element,
and obviously to retrieve the whole list if required.

## Usage
Using a zipper is pretty simple.
You build it from a list, move inside it, and insert/remove elements.
Here is a short usage:

```swift
var zippy = ListZipper<String>(from: ["I", "love", "pancakes"])
print(zippy.cursor!) // I
zippy.right() // move to the right
print(zippy.cursor!) // love
zippy.cursor = "hate"
print(zippy.toList().joined(separator: " ")) // I hate pancakes
zippy.remove()
zippy.cursor = "Eat"
print(zippy.toList().joined(separator: " ")) // Eat pancakes
```

## Implementation
Let’s start by the data itself.
We need to store the element of the list we are looking at (the cursor).
Since we are focusing on an element of the list,
we also need to remember what was before this element, and what comes next.

```swift
struct ListZipper<T> {

  private var leftList: [T] = []
  private var rightList: [T] = []

  /// Cursor to the selected Track
  var cursor: T? = nil

}
```

Now that we have the informations,
we need a way to convert from a list to a ListZipper,
and from a ListZipper to a list.
We add an initializer with as a default an empty list,
and a function that glue together the beginning of the list, the cursor,
and the remaining part of the list.

```swift
  init(from list: [T] = []) {
    cursor = list.first
    rightList = list.reversed()
  }

  /// Convert to a swift array
  func toList() -> [T] {
    if let c = cursor {
      return leftList + [c] + rightList.reversed()
    } else {
      return leftList + rightList.reversed()
    }
  }
```

Notice how we handle the case where the curser is pointing to nothing.
(A case which could occurs from an empty list / empty zipper.)

Our zipper can store the information,
but is always pointing to the first element of the list.
This is pretty boring and not really useful.
We introduce two functions that allow to move on the right or the left of
our band.

```swift
/// Select the left element if available
mutating func left() {
  guard leftList.isNotEmpty else {return}

  if let c = cursor {rightList.append(c)}
  cursor = leftList.popLast()
}

/// Select the right element if available
mutating func right() {
  guard rightList.isNotEmpty else {return}

  if let c = cursor {leftList.append(c)}
  cursor = rightList.popLast()
}
```

I personally choose to add guards for my particular usage,
because I always want to be pointing to a music track if my song is not empty.
But you could safely remove them.
In this setting you’d know you are on the bounds simply by checking if the
cursor is pointing to nothing.

One crucial step remaining to make this Zipper fully usable is a way to insert
and remove elements at the current location.
I choose to always insert on the right of the element pointed.
Symmetrically, when I remove the pointed element,
I automatically look to the left.

```swift
/// Insert a new track
mutating func insert(element: T) {
  if let c = cursor {leftList.append(c)}

  cursor = element
}

/// Remove the current pointed track
mutating func remove() {
  guard let _ = cursor else {return}

  if leftList.isNotEmpty {
    cursor = leftList.popLast()
  } else {
    cursor = rightList.popLast()
  }
}
```

Notice that in the case you removed the guard in the left/right functions,
you can also simplify remove by only poping the value from `leftList`.

Now you have a simple but powerful zipper in your hands.
For more fanciness, one could also enjoy the natural functoriality of the list
zipper to add the `map` function and allow us to map functions inside it :)

```swift
/// Map a function to the tracks
func map<R>(f: (T) -> R) -> ListZipper<R> {
  var zipper = ListZipper<R>()
  zipper.leftList = leftList.map(f)
  zipper.cursor = cursor.map(f)
  zipper.rightList = rightList.map(f)
  return zipper
}
```

I hope you liked this little taste of functional programming. 😉
