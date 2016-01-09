---
layout: post
title:  "Ruby Custom Sort Method - taking things for granted"
date:   2016-01-08 15:07:43 -0700
categories: jekyll update
---

Recently I was challenged with the simple task of creating my own custom sort method. Ruby has one built in that works great (which we will see when I post the timing numbers), but because this task wasn't as simple as I orignally imagined I wanted to dive into it. As Ruby programmers we tend to take all the methods that come with Ruby for granted and don't give much thought as to how they work and what it would take to re-create them.

To start, let's look at the quick solution what we originally came up with: (As noted, I did not come up with this by myself)

{% highlight ruby %}
def my_sort(arr)
  changed = true
  while changed do
    changed = false
    for i in 0..(arr.length-2)
      current = arr[i]
      next_number = arr[i + 1]
      if current > next_number 
        arr[i] = next_number
        arr[i+1] = current
        changed = true
      end
    end
  end
  arr
end

a = [-1, 8, 7, -6, 7, 9, 10]
p mysort(a)
{% endhighlight %}

We are not immediately proud of this solution, however it works. We quickly note that we are using the [bubble sort](https://en.wikipedia.org/wiki/Bubble_sort) algorithm which works as shown below, but has inherent speed problems which makes it impractical due to its inability to scale. 

![Bubble Sort - From Wikipedia](/assets/bubble-sort-example.gif)

Here is the timing of a randomly generated array with 10,000 numbers ranging from 0 to 100.

![Bubble Sort Timing](/assets/bubble-sort-timing.png)

As you can see, it took roughly 11 seconds to sort. Let compare this to the sort method that comes built into Ruby's Enumerable class

![Bubble Sort Timing](/assets/ruby-sort-timing.png)

Much faster. 

Even though the original hacky solution isn't pretty and it's limited by design, rather than trying to fix it up, the quest begins to find a more efficient method of sorting and see how fast we can get it compared to the ruby method.

According to wikipedia, we have some options. Insertion sort, quicksort, heapsort, and mergesort to mention a few. Admittedly, a quick google search and reading some source code could point directly to how Ruby is doing it, but for fun i'm waiting to check that out.

I'm going to go with Merge Sort, because after reading about it, it sounds pretty hopeful that it can be quick if implemented correctly. According to wikipedia: "Merge Sort is an efficient, general-purpose, comparison-based sorting algorithm." Sounds hopeful to me. A merge sort is basically a divide and conquer approach that splits the array into two halves and recursively sorts them and then combines them. In theory it looks like this:

![Merge Sort - From Wikipedia](/assets/merge-sort-example.gif)

Now the challenge of writing a merge sort. It was definitely more difficult then I thought it would be (as was the simple bubble sort) but after a few hours and thanks to this [pluralsight article](http://www.sitepoint.com/sorting-algorithms-ruby/) I ended up with a solution.

Here is what I came up with

{% highlight ruby %}
def merge_sort(arr)
  def merge(left, right)
    sorted = []
    left_index = 0
    right_index = 0
    loop do
      break if right_index >= right.length and left_index >= left.length

      if right_index >= right.length or (left_index < left.length and left[left_index] < right[right_index])
        sorted << left[left_index]
        left_index += 1
      else
        sorted << right[right_index]
        right_index += 1
      end
    end

    sorted 
  end

  def split_sort(arr)
    return arr if arr.length <= 1

    middle = arr.length / 2
    left_half = arr[0...middle]
    right_half = arr[middle..-1]
    left_half = split_sort(left_half) 
    right_half = split_sort(right_half)

    merge(left_half, right_half)   
  end

  split_sort(arr)
end

a = [-1, 8, 7, -6, 9, 10]

p merge_sort(a)
{% endhighlight %}

and here is the test so we can compare it to the ruby sort.

![Merge Sort Timing](/assets/merge-sort-timing.png)

Much faster than the Bubble Sort, but Ruby's sort method is still faster. It makes me curious, other than Ruby Magic, what method is the built in Sort method using?

A quick Google Search yields fast results, according to the [this](http://stackoverflow.com/questions/855773/which-algorithm-does-rubys-sort-method-use) Stack Overflow question, Ruby uses the Quicksort Algorithm. It appears to only downfall to this algorithm is when the array is almost sorted, the complexity becomes n^2. Other than this minor downfall, the Quicksort is a very effective sorting algorithm. Here is what it looks like in theory:

![Quicksort Example - Wikipedia](/assets/quicksort-example.gif)

Another thing to note, as a lot of Ruby's source code is in fact written in C, this sort method is no exception. Running this query at the lower level does make it faster than if we were to write an implementation in Ruby, which I will not be doing for this post. You can see this code if you look at the [sort!](http://ruby-doc.org/core-2.2.0/Array.html#method-i-sort-21) method on the ruby-docs
and click "Toggle Source"

In Summary, the three methods we looked at are:

<ul>
  <li>Bubble Sort</li>
  <li>Merge Sort</li>
  <li>Quicksort</li>
</ul>

Knowing there are variables I haven't taken into account, in our case the Quicksort used by Ruby is definitely faster than my other two custom implementations. 

Algorithms are a complex subject, there are many books and courses written on them. Sorting Algorithms typically have some units of measurement, including: Worst Case Performance, Best Case Performance, Average Case Performance, and Worst Case Complexity. 

Typically when working with Ruby, you won't need to recreate these yourself, but it is eye opening to dive into something that seems so simple. After attempting to recreate what I thought was simple, it definitely made me appreciate the time others have taken to build the beautiful language of Ruby (and all other languages) and the effort they've put in.

I won't be taking methods like .sort! for granted anymore, they have now gained my utmost respect!
