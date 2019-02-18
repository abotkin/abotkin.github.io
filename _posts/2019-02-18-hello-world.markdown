---
layout: post
title:  "Hello,  🌎!"
date:   2019-02-18 13:37:00 -0800
categories: misc helloworld 🌎
---
In the beginning the Programmer created the `main.m`.

{% highlight objc %}
#import <Foundation/Foundation.h>

int main(int argc, char *argv[]) {
    @autoreleasepool {
        return 0;
    }
}
{% endhighlight %}

The `main.m` was empty, simply returning 0 out of its auto-release pool.  And the Programmer's IDE idly rendered the source file.

Then the Programmer said, `Hello,  🌎!` and there was a  🌎.  And the Programmer saw the  🌎, that is was Unicode; and the Programmer delimited the string literal with `@""` to divide it from Objective-C's keywords & functions for the compiler.  The Programmer then called upon the function `NSLog` with the string literal, so as to print the string to the console for all the 🌎 to see.

{% highlight objc %}
#import <Foundation/Foundation.h>

int main(int argc, char *argv[]) {
    @autoreleasepool {
        NSLog(@"Hello, 🌎!");
        return 0;
    }
}
{% endhighlight %}

The Programmer ran "Build & Run" and saw that the 🌎 was printed and that it was good. And so the first program was written.
