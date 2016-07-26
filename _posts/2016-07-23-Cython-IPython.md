---
layout: post
title:  "Interactive Cython with IPython, no compilation anymore!"
date:   2016-07-23 18:52:21 -0500
categories: GSoC
comments: true
excerpt: "Use IPython to debug Cython code effortlessly."
---

Debugging Cython is sometimes very annoying because unfortunately, there aren't many blog posts or tutorials about Cython on the Internet. We often need to learn it by a trial-and-error manners. To make things even worse, unlike Python, Cython code need to be compiled everytime after we make some changes to it, which means that it will make our debugging process more tedious.

What if we can try out Cython in IPython notebooks, an interactive environment, without the need of compilation?

Let's see how this can be done.

## Ipython Notebooks

![](../../../../../assets/IPython.png)

**NOTE**: Feel free to skip this section if you are already familiar with it.

At first, let's see what [IPython notebooks](http://ipython.org/) is. 

An IPython notebook is a powerful interactive shell, which lets you write and execute Python code in your web browser. Therefore, it is very convenient to tweak your code and execute it in bits and pieces with IPython. Besides that, it also has great support for interactive data visualization and use of [GUI toolkits](http://ipython.org/ipython-doc/stable/interactive/reference.html#gui-event-loop-support). For these reasons, IPython notebooks are widely used in scientific computing.

For more installation details and tutorials, please see [this site](http://cs231n.github.io/ipython-tutorial/).

## Cython Problem

Traditionally, we leverage on module `distutils` to compile Cython code which gives us full control over every step of the process. However, The main drawback of this approach is that it requires a separate compilation step. This is definitely a disadvantage since one of Python’s strengths is its interactive interpreter, which allows us to play around with code and test how something works before committing it to a source file

Well, don't worry, IPython notebook is here to save us!


## %%cython Magic
IPython can integrage Cython flawlessly by typing some convenient commands that allow us to interactively use Cython from a live IPython session. These extra commands are IPython-specific commands called [magic commands](https://ipython.org/ipython-doc/3/interactive/magics.html), and they start with either a single (%) or double (%%) percent sign. They provide functionality beyond what the plain Python interpreter supplies. IPython has several magic commands to allow dynamic compilation of Cython code, see [here](https://ipython.org/ipython-doc/2/config/extensions/cythonmagic.html) for more details.

Before we can use these magic Cython commands, we first need to tell IPython to load them. We do that with the `%load_ext` metamagic command from the IPython interactive interpreter, or in an IPython notebook cell:

```In [1]: %load_ext Cython
```
There will be no output if `%load_ext` is successful, and IPython will issue an error message if it cannot find the Cython-related magics.

Great! Now we can use Cython from IPython via the `%%cython` magic command:

```
In [2]: %%cython
        cdef int add(int x, int y):
            return x + y 
```

The `%%cython` magic command allows us to write a block of Cython code directly in the IPython interpreter. After exiting the block with two returns, IPython will take the Cython code we defined, paste it into a uniquely named Cython source file, and compile it into an extension module. If compilation is successful, IPython will import everything from that module to make the function we defined available in the IPython interactive namespace. The compilation pipeline is still in effect, but it is all done for us **automatically**.We can now call the function we just defined:

```
In [3]: add(1, 2)
```

Cool! Now IPython will print the result of your function, i.e., 3, under this block of code.

## Generated C code
Sometimes, it is a good practice to inspect the generated C source files to check the sanity of our program. The generated source files are located in the `$IPYTHONDIR/cython` directory (`~/.ipython/cython` on an OS X or *nix system). The module names are not easily readable because they are formed from the md5 hash of the Cython source code, but all the contents are there.

## Summary

I really hope I would have known this convenient tips to debug my Cython code when I first knew Cython, it can really save tons of your time and efforts of your fingers :)

Let's run Cython code without overhead!
