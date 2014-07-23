---
layout: post
title: "Java Objects"
date: 2014-07-09 16:34:00
categories: java objects
---

## About Cars

Last week we spent the last couple minutes of our meeting discussing Objects. Here's a quick refresher.

An `Object` is a separate file in your project that contains variables and functions. These variables describe a certain *thing*--an "object"--and the functions describe all the actions that this object can perform.

In the example last week, we were describing a `Car`. First, we started by creating a `New Java Project` in Eclipse. Then, after selecting the `src` folder in the new project we just created, we added a `New Class` and named it `Car`.

{% highlight java %}
// file: Car.java
public class Car {
	int wheels;
	String color;
	int speed;
	public Car(int w, String c, int s) {
		wheels = w;
		color = c;
		speed = s;
	}
	public void speedUp(int amount) {
		speed = speed + amount;
	}
}
{% endhighlight %}

The variables `wheels`, `color`, and `speed` all describe information about the car. The function `speedUp()` describes a specific action of the car. 

In order to test the program, we added a second `New Class` to the project and named it `CarDemo`. It contained a `main()` function, which is the starting point when we run the entire program and a few lines of code to test our `Car` object.

{% highlight java %}
// file: CarDemo.java
public class CarDemo {
	public static void main(String args[]) {
		Car c = new Car(4, "blue", 0);
		System.out.println("Wheels: " + c.wheels);
		System.out.println("Color: " + c.color);

		// The car is stopped here
		System.out.println("Speed: " + c.speed);
		c.speedUp(100);
		// The car has sped up now
		System.out.println("New speed: " + c.speed);
	}
}
{% endhighlight %}

Using these concepts, you should now be able to add on more features to the car. Here are a few suggestions for things to do:
1. Add a `slowDown()` function so that the car can stop.
2. Add a `direction` variable and `turnLeft()` and `turnRight()` functions. The direction can be described in terms of "north", "south", "east", and "west".

## Combining input and objects

Programs are much more interesting when you are able to interact with them. Let's change this program so that you can enter the information needed to create the car. We'll be changing the `CarDemo` class.


{% highlight java %}
// file: CarDemo.java
public class CarDemo {
	public static void main(String args[]) {
		Scanner k = new Scanner(System.in);
		System.out.println("How many wheels does the car have?");
		int wheels = k.nextInt();
		System.out.println("What color is the car?");
		String color = k.next();
		System.out.println("What speed is it going?");
		int speed = k.nextInt();

		Car c = new Car(wheels, color, speed);
		System.out.println("Wheels: " + c.wheels);
		System.out.println("Color: " + c.color);

		// The car prints out the speed you entered
		System.out.println("Speed: " + c.speed);
		c.speedUp(100);
		// The car should be going faster now
		System.out.println("New speed: " + c.speed);
	}
}
{% endhighlight %}

Using the same pattern for input I've used before, I create a `Scanner` to read numbers (`k.nextInt()`) and words (`k.next()`). Then, using the variables that now contain the correct input from the user, I create a new `Car` object.

## Program suggestion

> ### Description
> Write an address book program where you can interactively add information about your friends and search for a friend based on the information you entered.

> Each entry in the address book should at least contain a first name and a last name. You can also include any other information you'd like to store.

> You should also include at a function to display an entry in the address book and a function that returns the full name of a contact.

### Program skeleton

This is an outline of the files you should use. You can copy and paste this to get you started. First, you need to create a new Eclipse project for your Address Book. Then, create two new classes: `AddressBook` and `Contact`.

{% highlight java %}
// file: AddressBook.java
public class AddressBook {
	// This is a list that will contain all the contacts you create.
	List<Contact> allContacts = new ArrayList<Contact>();

	public static void main(String args[]) {
		// Put your code here to start your program
	}

	// Use this function to create a new Contact and add it to
	// the allContacts list
	public static void addContact(String firstName, String lastName) {
		// You can create a new contact using a line of code
		// similar to this:
		Contact c = new Contact( .........  , .......... );
		// Then, you can add it to the allContacts list using
		// a line of code similar to this:
		allContacts.add(c);
	}

	// Use this function to find a contact that you've already created
	public static Contact findContact(String firstName, String lastName) {
		// You will need to use a loop to run through the allContacts list.
		// Remember the four parts of a loop: initialization, condition,
		// body, and update.
		for (int i = 0; i < allContacts.size(); i++) {
			// You can get one contact at a time by using:
			// allContacts.get(i)
			// Then, you need to see whether the first name of the contact
			// is equal to the first name you're searching for
			// Then, repeat for the last name. If both are true, return
			// the Contact you found.
			// This is what the code might look like. You'll need to
			// fill in the blank parts.
			Contact c = allContacts.get(i);
			if ( c.firstName.equals( ...... ) && c.lastName ......  ) {
				return c;
			}
		}
		// "null" is a special value that means "nothing"--in this case,
		// "return null" indicates that there was NO matching Contact.
		return null;
	}
}
{% endhighlight %}

{% highlight java %}
// file: Contact.java
public class Contact {
	// list all your variables here
	// ........

	// This function allows you to create a Contact using the code:
	// ..... = new Contact(first, last);
	// If you want to add more information to your Contact, you'll need to add
	// extra parameters to this function and extra lines in the function to
	// initialize all the appropriate variables.
	public Contact(String f, String l) {
		firstName = f;
		lastName = l;
	}

	public String fullName() {
		// You need to return the combination of the first name and
		// the last name.
		// First, create a variable to store the full name
		String fullName = firstName + ........ ;
		// Then, you can return the full name
		return fullName;
	}
}
{% endhighlight %}

## After you're done

This is just barely scraping the surface of Objects in Java. If you would like to learn more about how to use Objects, you should read about a few more topics. Here are some links to some tutorials to help you go further:

#### General
- [Java classes](http://www.homeandlearn.co.uk/java/write_your_own_java_classes.html)

#### Inheritance
- [Tutorialspoint](http://www.tutorialspoint.com/java/java_inheritance.htm)

Next week, we're going to start creating programs that have graphics that show up on the screen so we can start creating a Pong game. If you would like to learn a little about graphics before next week, you can read through the first couple parts of this tutorial:

[Java Swing tutorial](http://zetcode.com/tutorials/javaswingtutorial/)

