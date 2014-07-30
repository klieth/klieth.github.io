---
layout: post
title: "Java Graphics"
date: 2014-07-30 16:02:00
categories: java graphics
---

## Happy Face

Today we're going to be writing a Java program that displays a smiley face on the screen. This program requires good knowledge of functions and you also should be familiar with objects, so if you don't understand them I'd suggest taking a little time to refresh what we've learned in the past weeks.

Before we start programming, here are a few things you should know about graphics in Java:
- Your screen is made up of little tiny dots called *pixels*. These are really really tiny dots--you have millions of them in your screen. You'll see later that when we make a window that's 200 pixels wide and 200 pixels tall, it will only take up a little section of your screen. All graphics in Java depend on these pixels.
- The pixels are counted starting from the TOP LEFT of the screen. That means the very first pixel, called (0,0), means that the pixel is zero pixels from the right side of the screen and zero pixels down from the top of the screen. This is, of course, backwards from what you're used to seeing in Math class--make sure you have that correct!
- You're going to be drawing different shapes such as rectangles, ovals, and arcs. These are all defined in terms of a *bounding box*. This is easy to visualize for the rectangle--you'll tell the computer the coordinates of the top left corner of the box and then you'll specify the width and the hight. However, this is a little more difficult to understand for the oval and the arc. Imagine an invisible rectangle you've defined by specifying the top left corner and the width and height. Then, the oval will be drawn so that it touches the exact middle of each side of the rectangle.

This may not make sense to you right now, but that's ok. Let's jump into some code and try to show you what's going on with an example. First, we'll create a new Eclipse project--call it HappyFace. Then, add a new Java Class called HappyFace.java to your project. Make sure your code looks like this:

{%highlight java %}
public class HappyFace {
	public static void main(String[] args) {
	}
}
{% endhighlight %}

First, we have a very subtle change--I'm not going to explain this right now, but basically this change is going to make Java know that we want to treat this code as an area on the screen that we can draw on. Note the change on the first line after the import statement.

{%highlight java %}
import javax.swing.JPanel;

public class HappyFace extends JPanel{
		public static void main(String[] args) {
		}
}
{% endhighlight %}

Now, we need to put in all the functions we're going to need in order for this to work. Don't worry--we're only adding in two more methods for the whole program.


{%highlight java %}
import java.awt.Graphics;

import javax.swing.JPanel;

public class HappyFace extends JPanel{
		public static void main(String[] args) {
		}

		public HappyFace() {
		}
		
		public void paint(Graphics g) {
		}
}
{% endhighlight %}

We'll use the HappyFace() method to set up the drawing area when we create it and then we'll use the paint() method to actually tell the computer what to draw.

First, though, we need to create the window. This happens first thing when the program runs, so, of course, we'll put it in the main() function. The window is called a `JFrame` in Java. Then we create a `HappyFace`, add it to the window, and tell Java to show the window.


{%highlight java %}
import java.awt.Graphics;

import javax.swing.JPanel;
import javax.swing.JFrame;

public class HappyFace extends JPanel{
		public static void main(String[] args) {
			JFrame window = new JFrame();
			window.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
			HappyFace face = new HappyFace();
			window.add(face);
			window.pack();
			window.show();
		}

		public HappyFace() {
		}
		
		public void paint(Graphics g) {
		}
}
{% endhighlight %}


If you run this right now, you'll notice that nothing shows up. That's because the panel is zero pixels big. We'll have to fix that. Thankfully, it's pretty easy to do with one line of code.


{%highlight java %}
import java.awt.Graphics;
import java.awt.Dimension;

import javax.swing.JPanel;
import javax.swing.JFrame;

public class HappyFace extends JPanel{
		public static void main(String[] args) {
			JFrame window = new JFrame();
			window.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
			HappyFace face = new HappyFace();
			window.add(face);
			window.pack();
			window.show();
		}

		public HappyFace() {
			setPreferredSize(new Dimension(200,200)); 
		}
		
		public void paint(Graphics g) {
		}
}
{% endhighlight %}


Now we'll see a blank window! Now, inside the paint() method we can provide all the actual information to make it draw the face. We'll use two methods: `g.drawOval(x, y, width, height)` and `g.drawArc(x, y, width, height, degree_start, degrees_around)`.

{%highlight java %}
import java.awt.Graphics;
import java.awt.Dimension;

import javax.swing.JPanel;
import javax.swing.JFrame;

public class HappyFace extends JPanel{
		public static void main(String[] args) {
			JFrame window = new JFrame();
			window.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
			HappyFace face = new HappyFace();
			window.add(face);
			window.pack();
			window.show();
		}

		public HappyFace() {
			setPreferredSize(new Dimension(200,200)); 
		}
		
		public void paint(Graphics g) {
			// Draw the outline of the face
			g.drawOval(0,0,200,200);

			// Put your instructions to draw the eyes an nose here

			// Draw the mouth
			g.drawArc(50,120,100,40,190,160);
		}
}
{% endhighlight %}


## After you're done

Try clearing out the face and drawing anything you want! If you look at the documentation for [java.awt.Graphics](http://docs.oracle.com/javase/7/docs/api/java/awt/Graphics.html), you can see a listing of all the `draw...()` functions and you can draw some more exciting things. Next week, we'll start working on animations and will interact with the graphics.
