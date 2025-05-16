---
layout: post
title: "Weird Computer Questions"
categories: weird-computer-questions
excerpt: This is the result of shower thoughts, boredom, and way too much ambition. I am answering two weird computer questions and try to be scientific. Let's see if it worked.
banner: /2025/05/16/banner-computer-questions.svg
---

**This is the result** of shower thoughts, boredom, and way too much ambition. I am answering two weird computer questions and try to be scientific.

1. **Question 1:** [How long do I have to type random keys on my keyboard to write a working calculator in C?](#question-no-1-how-long-do-i-have-do-i-have-to-type-random-keys-on-my-keyboard-to-write-a-working-calculator-in-c)
2. **Question 2:** [If the internet were a single file, how long do I have to press backspace to delete it completely?](#question-no-2-if-the-internet-were-a-single-file-how-long-do-i-have-to-press-backspace-to-delete-it-completely)

# Question No. 1: How long do I have to type random keys on my keyboard to write a working calculator in C?

**Long. Very long. Extremely long. Longer than everything. Period.** This question is a nerdy spin on the classic [Infinite-Monkey-Theorem](https://en.wikipedia.org/wiki/Infinite_monkey_theorem).

All we need for this experiment is an imaginary monkey and a working calculator written in C. We can ignore the fact that there are endless ways of implementing a calculator in C because the requirements are hard: correct syntax, correct semantics, and correct functionality.

```c
#include <stdio.h>

int main(void)
{
    double a;
    double b;
    scanf("%lf", &a);
    scanf("%lf", &b);

    char op;
    getchar();
    scanf("%c", &op);

    switch (op) {
        case '+': {
            printf("%lf", a + b);
            break;
        }
        case '-': {
            printf("%lf", a - b);
            break;
        }
        case '*': {
            printf("%lf", a * b);
            break;
        }
        case '/': {
            printf("%lf", a / b);
            break;
        }
        default: {
            return 1;
        }
    }

    return 0;
}
```

This simple - and very ugly - calculator consists of $584$ characters. But because tabs, spaces, and new lines aren't important for the compiler, the smallest possible version looks like this and has $294$ characters:

```c
#include<stdio.h>
int main(void){double a;double b;scanf("%lf",&a);scanf("%lf",&b);char op;getchar();scanf("%c",&op);switch(op){case'+':{printf("%lf",a+b);break;}case'-':{printf("%lf",a-b);break;}case'*':{printf("%lf",a*b);break;}case'/':{printf("%lf",a/b);break;}default:{return 1;}}return 0;}
```

This is a reduction of around $50\%$ in character count just by removing unnecessary characters. In web technology, this type of compression is crucial to transmit long JavaScript code to the user in a short time. But back to our experiment. The C program can be written with the following alphabet: $$\Sigma = \left\{ a, \ldots, z, \langle, \rangle, ;, \%, (, ), +, -, *, /, ', ", \&, 0, 1, \{, \}, \# \right\}$$. $\Sigma$ contains 44 characters.

Before it gets unimaginable, let's make the mission of our monkey smaller. The new task for our monkey: Write the first line of code (`#include<stdio.h>`) with the given alphabet. To type this line of code, the monkey would need to try $44^{17}$ combinations. That's more combinations than stars in our universe! And that is not our only problem: time is a big problem. If the monkey needs $5$ seconds per input, it would take $4,3 \times 10^{28}$ seconds. It would take much longer than the age of the universe.

But we can help our monkey a bit: If the monkey types in a correct character, we store it and let him try typing the next character. Now, he would only need $44 \times 294 = 12936$ key presses in the worst case. Now, it would only take $3.5$ hours in the worst case.

These results don't show much, except that in the next time, monkeys will not replace us developers. Yes...

# Question No. 2: If the internet were a single file, how long do I have to press backspace to delete it completely?

**I have measured** how many keystrokes I can make in one minute. The result: $382$ keystrokes per minute (kpm)! If you delete one bit every time with the same kpm-rate, you would delete $382 \times 60 = 22920$ bits per hour, or $2865 \times 24 = 68760$ bytes per day. This is around $68$ kilobytes (this is approximately the size of a PDF with around four pages). Not quite enough to delete the whole internet.

But how big is _the whole internet_? This question is extremely hard to answer, because the internet is something you can't really measure. Kerry Coffman and Andrew Odlyzko [analyzed the traffic and transmission capacity of the public internet in 1998](https://firstmonday.org/ojs/index.php/fm/article/download/620/541). Their result: $75$ Gbps or $24300$ TB/month traffic at the end of the 20th century on the public internet.

These numbers are interesting to see, but not very helpful for us. The internet grew a lot in the past 22 years, and these numbers aren't reliable anymore. I found a few other articles and papers of the same age, but nothing really helpful to answer the question. An article in the [BBC Science Focus](https://www.sciencefocus.com/future-technology/how-much-data-is-on-the-internet) says something about $1200$ Petabytes held by the big four online storage companies: Google, Amazon, Microsoft, and Facebook. Other sources talk about zettabytes. Numbers so big and unimaginable.

Let's stay at the data from 1998 for now. To delete the whole traffic of 1997, you had to press backspace $75000$ times per second! With travel on your keyboard of 1.0mm (like on a thin notebook keyboard), your finger has to travel 1.0mm to press the key and 1.0mm to travel back. Your finger would be faster than a Bugatti Chiron Super Sport 300+, faster than a Formula 1 car, and faster than a landing Eurofighter. The distance covered by your finger would be $540$ km/h ($335$ MPH).

It would be easier if your backspace would delete one gigabit ($0.125$ gigabyte) when pressed. The speed of your finger would be $1.08$ km/h or $0.67$ MPH (for comparison, the speed of my $382$ kpm was about $0.04$ km/h or $0.02$ MPH). And still, we're talking about the internet 22 years ago.

One last funny calculation: Deleting one terabyte with the bit-backspace and my kpm-speed would take $5336$ years.

Klick, klick, klick...
