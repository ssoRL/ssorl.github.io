---
layout: post
title: "Efficient Animation With MATLAB"
excerpt: "A guide to fast, flexible animation techniques in MATLAB"
tags: [MATLAB, thesis]
<!-- image:
  feature: latech.jpg -->
---

I used animation to help me visualize some of the work I did for my [honours thesis](/work/thesis.pdf) (PDF warning). Prior to [MATLAB r2014b's major graphics changes](http://www.mathworks.com/help/matlab/graphics-changes-in-r2014b.html), it wasn't exactly easy to create efficient animations with MATLAB, so I spent some time figuring out the best way to do it. This is what I had in the end:

<figure>
	<a href="/matlab-animation/shock.gif"><img src="/matlab-animation/shock.gif"></a>
	<figcaption>Shockwave created by a driving piston in a one-dimensional nonlinear lattice</figcaption>
</figure>

### The Mistake

Say you want to animate a particle trajectory - position versus time, perhaps. The most obvious way would be to use `plot()` in a `for` loop, kind of like so:

{% highlight matlab %}
% Animation loop
for i = 1:totalFrames
	% Compute the necessary data
	particlePosition(i) = rand;

	% Plot the result of the computation
	plot(timeFrame(i), particlePosition(i));
end
{% endhighlight %}

If you've ever tried doing this you would notice that it scales very badly. Even if the plot is just a little bit complex this method becomes very slow, with lots of flashes of the figure window. Every time `plot()` is called MATLAB has to work to repeat a lot of unnecessary work.[^1] 

[^1]: This is no longer the case after r2014b.

### A Better Method

In [MathWorks' article on animation techniques](http://www.mathworks.com/help/matlab/creating_plots/animation-techniques.html), we can see that they recommend 

> Update the properties of a graphics object and display the updates on the screen. This technique is useful for creating animations when most of the graph remains the same. For example, set the `XData` and `YData` properties repeatedly to move an object in the graph.

Instead of using `plot()`, the `set()` function skips a lot of unnecessary work, making it more efficient. An implementation example would look like this:

{% highlight matlab %}
% Initialize the plot
h = plot(timeFrame(1), particlePosition(1));

% Animation loop
for i = 2:totalFrames
	% Compute the necessary data
	particlePosition(i) = rand;

	% Change the data in the plot
	set(h, 'XData', timeFrame(i));
	set(h, 'YData', particlePosition(i));
end
{% endhighlight %}

I wrote a simple script that uses this technique to animate a particle in a sine trajectory. You can download it [here](/matlab-animation/Animate.m). What it should look like:[^2]

<figure>
	<a href="/matlab-animation/sine.gif"><img src="/matlab-animation/sine.gif"></a>
	<figcaption>A particle with sine trajectory</figcaption>
</figure>

[^2]: This GIF is 60 FPS.

### Animating Multiple Trajectories

I ran into some problems when I needed to animate variable number of trajectories. I wanted to visualize how changing the number of particles in my system changed the trajectories, but MATLAB was giving me errors when I used the `set()` method. 

It turns out that animating multiple trajectories in stored one variable[^3] with the *same time vector* requires a trick. I will illustrate with an example:

[^3]: Because of the way `ode45` worked, it was most convenient this way.

{% highlight matlab %}
% Initialize the plot
h = plot(timeFrame(1), particlePosition(:,1));

% Animation loop
for i = 2:totalFrames
	% Compute the necessary data
	particlePosition(1,i) = rand;
	particlePosition(2,i) = rand;
	particlePosition(3,i) = rand;

	% Change the data in the plot
	set(h, 'XData', timeFrame(i));
	set(h, {'YData'}, num2cell(particlePosition(:,i)));
end
{% endhighlight %}

Here, `particlePosition` contains three trajectories. Note the use of `num2cell` in setting `YData` is required for the animation to work properly because of the way graphics data are structured in MATLAB. 

It's important to know that the `YData` you are setting must be a **column cell vector**. So if your data is structured such that each row represents a frame in the animation, you must transpose your data in `set()`.

I wrote another script to show how this can be implemented [here](/matlab-animation/Animate2.m). It looks like this:

<figure>
	<a href="/matlab-animation/sine2.gif"><img src="/matlab-animation/sine2.gif"></a>
	<figcaption>Three particles with different trajectories</figcaption>
</figure>

### Useful Things

- Sometimes, especially if your animation update command is after complicated computation, you need to use `drawnow` to force the animation to occur in real time.

- I learned a lot from MathWorks' animation example here: <https://www.mathworks.com/examples/matlab/4020-animation>, including how to export as GIFs.


