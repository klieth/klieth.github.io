---
layout: post
title: "Java Graphics Part 2"
date: 2014-08-06 14:35:00
categories: java graphics
---

## Recap of Java Graphics

Last week, we talked about how Java graphics work and drew a smiley face. I've repeated the basic information you need to know here:

- Your screen is made up of little tiny dots called *pixels*. These are really really tiny dots--you have millions of them in your screen. You'll see later that when we make a window that's 200 pixels wide and 200 pixels tall, it will only take up a little section of your screen. All graphics in Java depend on these pixels.
- The pixels are counted starting from the TOP LEFT of the screen. That means the very first pixel, called (0,0), means that the pixel is zero pixels from the right side of the screen and zero pixels down from the top of the screen. This is, of course, backwards from what you're used to seeing in Math class--make sure you have that correct!
- You're going to be drawing different shapes such as rectangles, ovals, and arcs. These are all defined in terms of a *bounding box*. This is easy to visualize for the rectangle--you'll tell the computer the coordinates of the top left corner of the box and then you'll specify the width and the hight. However, this is a little more difficult to understand for the oval and the arc. Imagine an invisible rectangle you've defined by specifying the top left corner and the width and height. Then, the oval will be drawn so that it touches the exact middle of each side of the rectangle.

Today, we'll work on getting the smiley face to move around the screen by itself and then use the arrow keys to tellit where to move.

## Animation

On a computer, animations, such as movies you see at the theater, are made up of a series of still images. These still images are known as `frames`. Each frame is slightly different from the previous one in the series. When they are displayed on the screen in order, they make us think we're seeing an image that moves perfectly smoothly. However, unlike in the movies, the series of frames isn't exactly the same every time you run a program that has animations. On the computer, you need to give enough information to the computer to tell it how to calculate which pixels will be colored in the frame the frame, without giving exact pixels to draw.

Let's use the face we drew last week as an example. This is going to be a fairly simple animation--just a face moving around the screen. Instead of telling the computer exactly which pixels to turn black on any given frame, we're going to tell it how to draw the face *once* in a generic position called `(x,y)` and we're going to outline a path for the face to follow when it moves. Then, we'll tell the face to follow the path every frame.

First, we're going to start out where we left off last week: a window with a face drawn inside it. However, I'm going to first make a couple big changes. I'm going to split all the code from last week into two separate files. One of them will create the window and the other will draw the face. So you'll need to start off by creating a new Java Class in your project called `Window.java`. There are a number of small changes that we're going to make, including renaming a couple methods so be careful that you find all of them!

At the same time, I'm going to make one minor change: I'm going to use variables to represent the size of the window, and I'm going to make it a little bigger because we need some room for the face to move.

{% highlight java %}
// Window.java

import java.awt.Dimension;
import java.awt.Graphics;

import javax.swing.JPanel;
import javax.swing.JFrame;

public class Window extends JPanel {
	private static final int WINDOW_WIDTH = 400;
	private static final int WINDOW_HEIGHT = 400;

	public static void main(String[] args) {
		JFrame container = new JFrame();
		container.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

		container.add(new Window());
		container.pack();
		container.show();
	}




	private HappyFace face = new HappyFace();

	public Window() {
		setPreferredSize(new Dimension(WINDOW_WIDTH, WINDOW_HEIGHT));
	}

	public void paint(Graphics g) {
		face.paint(g);
	}
}
{% endhighlight %}

{%highlight java %}
// HappyFace.java

import java.awt.Graphics;

public class HappyFace {

	public HappyFace() {
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

Right now, the face only knows how to draw itself in the top left corner of the screen. Remember, this is position is called (0,0) meaning it's zero pixels away from the left side of the screen and zero pixels down from the top of the screen. We need to change this so that the face can draw itself anywhere on the screen. 

Let's give the face an `x` and a `y` coordinate. This should look very similar to when we created the `Car` object some time ago--just like the `speed` described an aspect of the car, `x` or `y` describes an aspect of the face. You'll also notice that inside the `constructor`, the special method that initializes any details about the HappyFace, I gave an initial value of 0 to both `x` and `y`.

{%highlight java %}
// HappyFace.java

import java.awt.Graphics;

public class HappyFace {

	private int x;
	private int y;

	public HappyFace() {
		x = 0;
		y = 0;
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

However, this doesn't magically make the face move its position. You'll notice that if you modify the value of `x` or `y` the face stays put--always in the top left corner. Let's fix that. First, we need to remember the details about the `draw` instructions we were using.

- g.drawOval(x, y, width, height);
- g.drawArc(x, y, width, height, start, degrees);

Both of these `draw` instructions say that you need to give an x and a y position, which is exactly what we need to change to use our `x` and `y` variables. This is easy for the first `drawOval()`, because we can replace `0,0` with `x,y`. However, it's slightly more complicated for the mouth. Remember, `x,y` points to the top left corner of the *face*, and the mouth needs to be offset 50 pixels to the right and 120 pixels down from the top left corner of the face. That can be solved with some simple addition.

{% highlight java %}
// HappyFace.java
import java.awt.Graphics;

public class HappyFace {

	private int x;
	private int y;

	public HappyFace() {
		x = 0;
		y = 0;
	}

	public void paint(Graphics g) {
		// Draw the outline of the face
		g.drawOval(x,y,200,200);

		// Put your instructions to draw the eyes and nose here

		// Draw the mouth
		g.drawArc(x + 50,y + 120,100,40,190,160);
	}
}
{% endhighlight %}

Now, if you change the initial values of `x` and `y` you should see the face move to a different place on the screen. If only certain parts of the face move, make sure you've changed the eyes, nose, and whatever else is part of your face to use the `x,y` coordinate method instead of the numbers you had before.

But the face still isn't moving. Let's get some motion there. The first thing to understand about animation in Java is the concept of the `animation loop`. This is a loop that runs forever (well, until you decide to stop it) and it does two things: 

1. it tells all the different items on the screen to move and prepare their positions for the next frame
2. it tells all the different items on the screen to draw themselves

A bunch of different changes were made here. I've put notes in the code to describe various things. Any variable that has `time` in it is some fancy code to make your graphics run more smoothly. Go ahead and copy it in, but I'm not going to spend a long time explaining how it works for now.

{% highlight java %}
// Window.java

import java.awt.Dimension;
import java.awt.Graphics;

import javax.swing.JPanel;
import javax.swing.JFrame;

public class Window extends JPanel {
	private static final int WINDOW_WIDTH = 400;
	private static final int WINDOW_HEIGHT = 400;

	public static void main(String[] args) {
		JFrame container = new JFrame();
		container.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

		Window w = new Window();
		container.add(w);
		container.pack();
		container.show();

		long frameStartTime;

		// Here's the animation loop
		while (true) {
			frameStartTime = System.currentTimeMillis();

			// Step 1. Move everything
			w.moveAllObjects();
			// Step 2. Draw everything
			// repaint() is a special method that prepares the screen for
			// drawing and then calls paint() for you.
			w.repaint();

			try {
				long sleepTime = 50 - (System.currentTimeMillis()
								- frameStartTime);
				Thread.sleep(sleepTime > 0 ? sleepTime : 0);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}




	private HappyFace face = new HappyFace();

	public Window() {
		setPreferredSize(new Dimension(WINDOW_WIDTH, WINDOW_HEIGHT));
	}

	public void moveAllObjects() {
		// The face knows how to move itself, so go ahead and call its move()
		// function
		face.move();
	}

	public void paint(Graphics g) {
		// By default, the face from the previous frame is left on the screen
		// when the new frame is drawn. We have a special clearRect() function
		// to make the screen blank.
		g.clearRect(0, 0, WINDOW_WIDTH, WINDOW_HEIGHT);
		face.paint(g);
	}
}
{% endhighlight %}
{% highlight java %}
// HappyFace.java
import java.awt.Graphics;

public class HappyFace {

	private int x;
	private int y;

	public HappyFace() {
		x = 0;
		y = 0;
	}

	public void move() {
		// Every frame, the face will move 2 pixels to the right
		x = x + 2;
		// And one pixel down.
		y = y + 1;
	}

	public void paint(Graphics g) {
		// Draw the outline of the face
		g.drawOval(x,y,200,200);

		// Put your instructions to draw the eyes and nose here

		// Draw the mouth
		g.drawArc(x + 50,y + 120,100,40,190,160);
	}
}
{% endhighlight %}

Well, the face moves now! But if you let it go for a second, it wanders right off of the screen--that's not good. We'll let it do that for now, though, and take a look at a few other things in the meantime.

## Keyboard input

Pushing buttons on the keyboard in Java is a little complicated to understand, so I'm going to write some code for you guys to copy and make it a little simpler for yourselves. This code will be a separate Java Class file in your project called `ArrowKeys.java`

{% highlight java %}
// ArrowKeys.java

import java.awt.event.KeyListener;
import java.awt.event.KeyEvent;

public class ArrowKeys implements KeyListener {
	private static boolean right = false;
	private static boolean left = false;
	private static boolean up = false;
	private static boolean down = false;

	public static boolean isRightPressed() {
		return right;
	}
	public static boolean isLeftPressed() {
		return left;
	}
	public static boolean isUpPressed() {
		return up;
	}
	public static boolean isDownPressed() {
		return down;
	}

	@Override
	public void keyPressed(KeyEvent e) {
		switch(e.getKeyCode()) {
			case KeyEvent.VK_LEFT:
				left = true;
				break;
			case KeyEvent.VK_RIGHT:
				right = true;
				break;
			case KeyEvent.VK_UP:
				up = true;
				break;
			case KeyEvent.VK_DOWN:
				down = true;
				break;
			default:
				System.out.println("No action defined");
				break;
		}
	}
	@Override
	public void keyReleased(KeyEvent e) {
		switch(e.getKeyCode()) {
			case KeyEvent.VK_LEFT:
				left = false;
				break;
			case KeyEvent.VK_RIGHT:
				right = false;
				break;
			case KeyEvent.VK_UP:
				up = false;
				break;
			case KeyEvent.VK_DOWN:
				down = false;
				break;
			default:
				System.out.println("No action defined");
				break;
		}
	}
	@Override
	public void keyTyped(KeyEvent e) {
	}
}
{% endhighlight %}

Once that's included in your project, you can use these four functions to test whether the keys on the keyboard have been pressed:

- ArrowKeys.isRightPressed()
- ArrowKeys.isLeftPressed()
- ArrowKeys.isUpPressed()
- ArrowKeys.isDownPressed()

Here's an example of how you might use it:

{% highlight java %}
// HappyFace.java

import java.awt.Graphics;

public class HappyFace {

	private int x;
	private int y;

	public HappyFace() {
		x = 0;
		y = 0;
	}

	public void move() {
		if (ArrowKeys.isRightPressed()) {
			x = x + 2;
		} else if (ArrowKeys.isLeftPressed()) {
			x = x - 2;
		}
		// Do the same thing here, but using up/down keys and change y
	}

	public void paint(Graphics g) {
		// Draw the outline of the face
		g.drawOval(x,y,200,200);

		// Put your instructions to draw the eyes and nose here

		// Draw the mouth
		g.drawArc(x + 50,y + 120,100,40,190,160);
	}
}
{% endhighlight %}
