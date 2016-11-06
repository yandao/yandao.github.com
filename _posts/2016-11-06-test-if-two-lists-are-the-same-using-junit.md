---
layout: post
title: "Test if Two Lists are the Same using JUnit"
description: ""
category: quick tips
tags: [java]
---
{% include JB/setup %}


## Using Standard JUnit

    org.junit.Assert.assertEquals()

    and

    org.junit.Assert.assertArrayEquals()


## Using Hamcrest

    Assert.assertThat(listUnderTest, IsIterableContainingInOrder.contains(expectedList.toArray()));

gives better output in case of failure, for example:

    expected List containing "1, 2, 3, ..." got list containing "4, 6, 2, ..."


## Using Google Guava

    Iterables.elementsEqual()


## Additional Note

From [Stack Overflow](http://stackoverflow.com/a/1075817):

    As everyone says here, using equals() depends on the order. If you don't care about order, you have 3 options.

    Option 1

    Use containsAll(). This option is not ideal, in my opinion, because it offers worst case performance, O(n^2).

    Option 2

    There are two variations to this:

    2a) If you don't care about maintaining the order ofyour lists... use Collections.sort() on both list. Then use the equals(). This is O(nlogn), because you do two sorts, and then an O(n) comparison.

    2b) If you need to maintain the lists' order, you can copy both lists first. THEN you can use solution 2a on both the copied lists. However this might be unattractive if copying is very expensive.

    This leads to:

    Option 3

    If your requirements are the same as part 2b, but copying is too expensive. You can use a TreeSet to do the sorting for you. Dump each list into its own TreeSet. It will be sorted in the set, and the original lists will remain intact. Then perform an equals() comparison on both TreeSets. The TreeSets can be built in O(nlogn) time, and the equals() is O(n).

    Take your pick :-).


## References

* [How to JUnit test that two List<E> contain the same elements in the same order? - Stack Overflow](http://stackoverflow.com/a/12495604)
* [Assert (JUnit API)](http://junit.sourceforge.net/javadoc/org/junit/Assert.html#assertEquals(java.lang.Object,%20java.lang.Object)
* [IsIterableContainingInOrder (Hamcrest)](http://hamcrest.org/JavaHamcrest/javadoc/1.3/org/hamcrest/collection/IsIterableContainingInOrder.html)
* [guava/Iterables.java](https://github.com/google/guava/blob/master/guava/src/com/google/common/collect/Iterables.java)