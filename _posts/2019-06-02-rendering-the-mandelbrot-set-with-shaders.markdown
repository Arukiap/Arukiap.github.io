---
layout: post
title:  "Rendering the Mandelbrot Set with shaders"
subtitle: "Using OpenGL and GLSL"
date:   2019-06-02 22:49:00
categories: [fractals]
mathjax: true
---

# Introduction

A great way to start coding shaders is by rendering fractals with them.

In this post I will take you through the creation of a simple mandelbrot set explorer application that I built using GLSL shaders.



If you want to start learning to code shaders, I suggest taking a look at [ShaderToy](https://www.shadertoy.com/). ShaderToy is a website were you can code your own shaders right out-of-the-box, without having to worry about establishing a lower level application to deal with input, screen libraries and other steps that might demotivate you right from the beginning. You can also share your creations and there are a couple of useful tutorials out there for you to learn from and amazing shaders to explore!

If you are feeling adventurous or can't depend on an internet connection, you can clone [this git repository](https://github.com/Arukiap/OpenGL-Shaders-Base) where I host a base C++ and OpenGL application that allows you to dive right into shader coding locally on your machine.

I also greatly encourage you to visit [Inigo Quilez](https://www.iquilezles.org/www/articles/ftrapsgeometric/ftrapsgeometric.htm)'s website if you are insterested in fractal rendering techniques and the great explanation [The Art of Code](https://www.youtube.com/watch?v=6IWXkV82oyY) gives in his mandelbrot fractal introduction video. I largely based my development and learning through these two platforms.

# The mandelbrot set

Taking the definition straight out of its [Wikipedia page](https://www.shadertoy.com/), the mandelbrot set is a set of complex numbers c for which the function
$$ f_c(z) = z^2 + c $$
stays bounded between a certain range of values when iterated from z = 0.

## Complex numbers

Now, **don't let the complex numbers scare you**. A complex number is, as you might know, a number that has two distinct components: a real and an imaginary one. Thus, we can represent a complex number by using a two-dimensional vector, where the x component represents the real part of the number, and the y component represents the imaginary part. We will use z to denote our complex number.
$$ z = x + yi $$
$$ z = vec2(x,y) $$
Now, the only thing we need to worry about when doing math on imaginary numbers is the following property for this set:
$$ i^2 = -1 $$
Let's try to substitute the complex number z in the mandelbrot set function:
$$ f_c(z) = z^2 + c = (x + yi)^2 + c = (x^2 + y^2i^2 + 2xyi) + c = x^2 - y^2 + 2xyi + c $$
And using the two-dimensional vector representation for the complex number system we have:
$$ f_c((x,y)_z) = (x^2-y^2,2xy)_z + (x,y)_c $$

We now have all that is necessary to start rendering the mandelbrot set. We just need to iterate this function and see which values remain bounded in
$$ |z_n+1| <= 2 $$
or in other words, applying this to the vector representation
$$ |(x,y)_z| <= 2 $$


## Rendering the set

Now that we have our maths figured out, let's open up our fragment shader and write the core function of our application: the iteration over the function that defines the mandelbrot set.

~~~ GLSL
vec2 squareImaginary(vec2 number){
	return vec2(
		pow(number.x,2)-pow(number.y,2),
		2*number.x*number.y
	);
}

float iterateMandelbrot(vec2 coord){
	vec2 z = vec2(0,0);
	for(int i=0;i<maxIterations;i++){
		z = squareImaginary(z) + coord;
		if(length(z)>2) return i/maxIterations;
	}
	return maxIterations;
}
~~~

This function runs once every pixel of the screen. The coord 2D vector represents the current selected pixel coordinates. This works great because in mandelbrot plots, the y axis represents the imaginary part of the complex number and the x the real part.

We just follow the exact function definition we listed before, and return a value between 0 and 1 based on the closeness of our iteration steps to the maximum number of iterations. This is used to attribute a certain shade value to each pixel of the screen.

This produces the following image (Maximum number of iterations = 100):

![](../assets/mandelbrot/mandelbrot1bright.png)

## Zooming in

By implementing a simple interaction of zooming in and exploring the set, we can already start to see the complexity and beauty of this set.

![](../assets/mandelbrot/mandelbrot2bright.png)
![](../assets/mandelbrot/mandelbrot3bright.png)

But we surely don't want to stop here. While these images look great and mysterious, more beauty can be brought out of the plot representation of this set if we color it based on mathematical rules.

## Orbit trap coloring

To color our mandelbrot plot, we are going to use a popular technique used to color fractals called **orbit trapping**. This technique consists of storing the minimum, average, maximum or other kinds of mathematical functions that are applied on the distance of the current point that is being iterated throughout our iterative function to a certain point, line or curve in our coordinate system. We can then color our pixels based on those values.

A common point to compare the distance to is the origin (0,0). Doing so produces the following coloring of our set:

![](../assets/mandelbrot/mandelbrotOT1.png)
![](../assets/mandelbrot/mandelbrotOT2.png)
![](../assets/mandelbrot/mandelbrotOT3.png)
![](../assets/mandelbrot/mandelbrotOT4.png)
![](../assets/mandelbrot/mandelbrotOT7.png)
![](../assets/mandelbrot/mandelbrotOT6.png)

Because this is usually refered to as art, you can color the fractal however you feel like. Feel free to experiment: most of the images I posted here took a lot of trial and error to get the coloring done right.

If you are interested in orbit trapping techniques, you can learn more about it [here](https://www.iquilezles.org/www/articles/ftrapsgeometric/ftrapsgeometric.htm).

## The floating point problem

Because GPUs are typically designed to work on a big ammounts of batched data on the vertex shader, they don't really like to work with high precision floating point calculations. The maximum precision you can get out your normal GLSL shader is a 16-bit floating point and even for that you need to do the following declaration on your shader code.

~~~ GLSL
precision highp float;
~~~

This is an extreme limitation for the application I wanted to write: an almost infinite zoom into the mandelbrot set. Because of this, if I keep zooming in my application after almost 10 to 15 seconds I get the following renders.

![](../assets/mandelbrot/mandelbrotprecision.png)

**This is not a compressed image**, it is exactly the render I get when I zoom to much due to floating point number representation limits in the GPU.

Consequently, making deeper dives into this amazing fractal world would require a lower-level implementation that would allow for unlimited floating point precision. This is out of the scope of this experimentation.

# Conclusions

Creating this experimental application not only served as a great way to dive into the fractal world, but also to understand the fundamental limitations of shader development. I greatly encourage everyone that is interested in graphics computation techniques to give a go at the mandelbrot set. It sets you up for working with screen pixel coordinates, coloring and shading which are essential components of shader development. On top of that, the renders you can produce look amazing. Take a look at [this](https://www.youtube.com/watch?v=VPHbgHVxLYY) video produced by available software on the internet to render and zoom in on the mandelbrot set with millions of iterations!

I hope you enjoyed the read and I am always open for your questions and feedback.

Have a great day!

~~ Arukiap