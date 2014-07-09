---
layout: post
title: "Java Loops"
date: 2014-07-09 16:34:00
categories: java loops
---

## Solution to last week's program

> ### Program description
> You are programming a vending machine. Each time someone uses it, they put in 100 cents in change and then type the price of the item they're buying. The vending machine then calculates the number of quarters, dimes, nickels, and pennies that should be given as change.

> ### Sample input
> 80

> ### Sample output
> 2 dimes

There were two ways to create this program. The first way is to use conditionals--`if-statements`--to simulate the way that you would hand back change if you were a cashier.

{% highlight java %}
public class VendingMachine {
	public static void main(String[] args) {
		Scanner keyboard = new Scanner(System.in);
		int price = keyboard.nextInt();
		int change = 100 - price;
		int quarters = 0;
		int dimes = 0;
		int nickels = 0;
		int pennies = 0;

		if (change >= 25) {
			quarters = quarters + 1;
			change = change - 25;
		}
		if (change >= 25) {
			quarters = quarters + 1;
			change = change - 25;
		}
		if (change >= 25) {
			quarters = quarters + 1;
			change = change - 25;
		}

		if (change >= 10) {
			dimes = dimes + 1;
			change = change - 10;
		}
		if (change >= 10) {
			dimes = dimes + 1;
			change = change - 10;
		}

		if (change >= 5) {
			nickels = nickels + 1;
			change = change - 5;
		}

		pennies = change;

		System.out.println(quarters + " quarter(s)");
		System.out.println(dimes + " dime(s)");
		System.out.println(nickels + " nickel(s)");
		System.out.println(pennies + " penny(ies)");
	}
}
{% endhighlight %}

The second way is to calculate the change using math.

{% highlight java %}
public class VendingMachine2 {
	public static void main(String[] args) {
		Scanner keyboard = new Scanner(System.in);
		int price = keyboard.nextInt();
		int change = 100 - price;

		int quarters = change / 25;
		change = change - ( quarters * 25 );
		int dimes = change / 10;
		change = change - ( dimes * 10 );
		int nickels = change / 5;
		change = change - ( nickels * 5);
		int pennies = change;

		System.out.println(quarters + " quarter(s)");
		System.out.println(dimes + " dime(s)");
		System.out.println(nickels + " nickel(s)");
		System.out.println(pennies + " penny(ies)");
	}
}
{% endhighlight %}

## Loops introduction

Sometimes, you'll want to take a condition like the ones we've been working with and then run that code multiple times. We've already seen an example of this; when you're trying to take quarters away, you need to hand quarters back up to three times. In Java, there's a great way to do this: a *loop*. There are two types of loops: `for` loops and `while` loops. 

### While loops

This type of loop looks very similar to the `if` statement we were working with before, but instead of typing `if` you type `while`.

There are four important steps to remember when writing a loop:
1. initialization - initialize any loop-related variables
2. condition - the question asking if the loop should keep running
3. body - the steps the perform the processing of the loop
4. update - update the variables originally initialized

    [1. initialization]
    while ( [2. condition ] ) {
       [3. body];
       [4. update]
    }

A simple loop might sum up a set of numbers.

{% highlight java %}
int sum = 0;        // initialization
int n = 0;
while ( n < 10 ) {  // condition
	sum = sum + n;  // body
	n = n + 1;      // update
}
System.out.println("The sum is: " + sum);
{% endhighlight %}

There are two variables: `sum` and `n`. `n` up starting at 0 and ending when `n` is no longer greater than 10. `sum` contains the running total of the digits so far.

Now we can write the vending machine program in a better way. See if you can figure out which are the four parts of each loop.

{% highlight java %}
public class VendingMachine {
	public static void main(String[] args) {
		Scanner keyboard = new Scanner(System.in);
		int price = keyboard.nextInt();
		int change = 100 - price;
		int quarters = 0;
		int dimes = 0;
		int nickels = 0;
		int pennies = 0;

		while (change >= 25) {
			quarters = quarters + 1;
			change = change - 25;
		}

		while (change >= 10) {
			dimes = dimes + 1;
			change = change - 10;
		}

		while (change >= 5) {
			nickels = nickels + 1;
			change = change - 5;
		}

		pennies = change;

		System.out.println(quarters + " quarter(s)");
		System.out.println(dimes + " dime(s)");
		System.out.println(nickels + " nickel(s)");
		System.out.println(pennies + " penny(ies)");
	}
}
{% endhighlight %}

### For loop

A `for` loop has a similar purpose to the `while` loop, but it has a different format. It has the same four parts in a different format:

    for ( [1. initialization] ; [2. condition] ; [4. update] ) {
		[3. body]
	}

Using the same example from above, where we sum up the first 9 positive integers:

{% highlight java %}
int sum = 0;
for ( int n = 0; n < 10; n=n+1 ) {
	sum = sum + n;
}
{% endhighlight %}

`For` loops and `while` loops are generally interchangeable and many people choose which one they like best and stick with it. There are a few cases where one creates less code than the other, but those situations are rare.

## Program suggestion

> ### Description
> Write a program where you will enter a word and display the number of vowels in the word.

> ### Sample input
> alphabet

> ### Sample output
> 3 vowels

## After you're done

If you understand conditionals and loops, you've covered all the basics of Java. Now it's time to cover some of the more complex topics such as *functions* and  *objects*. Feel free to look up tutorials this week, but we'll cover some of that next week.
