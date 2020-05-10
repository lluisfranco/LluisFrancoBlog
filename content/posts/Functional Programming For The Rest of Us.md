---
title: "Functional Programming For The Rest of Us"
subtitle: "(Fragment)"
date: 2012-02-08T11:51:50+02:00
featuredImage: /images/posts/functionalprogramming.png
fontawesome: "true"
categories: 
- Development
- Fundamentals
tags:
- Functional
- Programming
- Lambda
- History
# draft: true
---
## Introduction

Programmers are procrastinators. Get in, get some coffee, check the mailbox, read the RSS feeds, read the news, check out latest articles on techie websites, browse through political discussions on the designated sections of the programming forums. Rinse and repeat to make sure nothing is missed. Go to lunch. Come back, stare at the IDE for a few minutes. Check the mailbox. Get some coffee. Before you know it, the day is over.

The only thing, every once in a while challenging articles actually do pop up. If you're looking at the right places you'll find at least one of these every couple of days. These articles are hard to get through and take some time, so they start piling up. Before you know it, you have a list of links and a folder full of PDF files and you wish you had a year in a small hut in the middle of the forest with nobody around for miles so you could catch up. Would be nice if someone came in every morning while you're taking a walk down the river to bring some food and take out the garbage.

I don't know about your list, but a large chunk of the articles in mine are about functional programming. These generally are the hardest to get through. Written in a dry academic language, even the "ten year Wall Street industry veterans" don't understand what functional programming (also referred to as FP) articles are all about. If you ask a project manager in Citi Group or in Deutsche Bank1 why they chose to use JMS instead of Erlang they'll say they can't use academic languages for industrial strength applications. The problem is, some of the most complex systems with the most rigid requirements are written using functional programming elements. Something doesn't add up.

It's true that FP articles and papers are hard to understand, but they don't have to be. The reasons for the knowledge gap are purely historical. There is nothing inherently hard about FP concepts. Consider this article "an accessible guide to FP", a bridge from our imperative minds into the world of FP. Grab a coffee and keep on reading. With any luck your coworkers will start making fun of you for your FP comments in no time.

So what is FP? How did it come about? Is it edible? If it's as useful as its advocates claim, why isn't it being used more often in the industry? Why is it that only people with PhDs tend to use it? Most importantly, why is it so damn hard to learn? What is all this closure, continuation, currying, lazy evaluation and no side effects business? How can it be used in projects that don't involve a university? Why does it seem to be so different from everything good, and holy, and dear to our imperative hearts? We'll clear this up very soon. Let's start with explaining the reasons for the huge gap between the real world and academic articles. The answer is as easy as taking a walk in the park.

## A Walk In The Park

Fire up the time machine. Our walk in the park took place more than two thousand years ago, on a beautiful sunny day of a long forgotten spring in 380 B.C. Outside the city walls of Athens, under the pleasant shade of olive trees Plato was walking towards the Academy with a beautiful slave boy. The weather was lovely, the dinner was filling, and the conversation turned to philosophy.

"Look at these two students", said Plato carefully picking words to make the question educational. "Who do you think is taller?" The slave boy looked towards the basin of water where two men were standing. "They're about the same height", he said. "What do you mean 'about the same'?", asked Plato. "Well, they look the same from here but I'm sure if I were to get closer I'd see that there is some difference."

Plato smiled. He was leading the boy in the right direction. "So you would say that there is nothing perfectly equal in our world?" After some thinking the boy replied: "I don't think so. Everything is at least a little different, even if we can't see it." The point hit home! "Then if nothing is perfectly equal in this world, how do you think you understand the concept of 'perfect' equality?" The slave boy looked puzzled. "I don't know", he replied.

So was born the first attempt to understand the nature of mathematics. Plato suggested that everything in our world is just an approximation of perfection. He also realized that we understand the concept of perfection even though we never encountered it. He came to conclusion that perfect mathematical forms must live in another world and that we somehow know about them by having a connection to that "alternative" universe. It's fairly clear that there is no perfect circle that we can observe. But we also understand what a perfect circle is and can describe it via equations. What is mathematics, then? Why is the universe described with mathematical laws? Can all of the phenomena of our universe be described by mathematics?2

[Philosophy of mathematics](http://en.wikipedia.org/wiki/Philosophy_of_mathematics) is a very complex subject. Like most philosophical disciplines it is far more adept at posing questions rather than providing answers. Much of the consensus revolves around the fact that mathematics is really a puzzle: we set up a set of basic non-conflicting principles and a set of rules on how to operate with these principles. We can then stack these rules together to come up with more complex rules. Mathematicians call this method a "formal system" or a "calculus". We can effectively write a formal system for Tetris if we wanted to. In fact, a working implementation of Tetris is a formal system, just specified using an unusual representation.

A civilization of furry creatures on Alpha Centauri would not be able to read our formalisms of Tetris and circles because their only sensory input might be an organ that senses smells. They likely will never find out about the Tetris formalism, but they very well might have a formalism for circles. We probably wouldn't be able to read it because our sense of smell isn't that sophisticated, but once you get past the representation of the formalism (via various sensory instruments and standard code breaking techniques to understand the language), the concepts underneath are understandable to any intelligent civilization.

Interestingly if no intelligent civilization ever existed in the universe the formalisms for Tetris and circles would still hold water, it's just that nobody would be around to find out about them. If an intelligent civilization popped up, it would likely discover some formalisms that help describe the laws of our universe. They also would be very unlikely to ever find out about Tetris because there is nothing in the universe that resembles it. Tetris is one of countless examples of a formal system, a puzzle, that has nothing to do with the real world. We can't even be sure that natural numbers have full resemblance to the real world, after all one can easily think of a number so big that it cannot describe anything in our universe since it might actually turn out to be finite.

## A Bit of History

Let's shift gears in our time machine. This time we'll travel a lot closer, to the 1930s. The Great Depression was ravaging the New and the Old worlds. Almost every family from every social class was affected by the tremendous economic downturn. Very few sanctuaries remained where people were safe from the perils of poverty. Few people were fortunate enough to be in these sanctuaries, but they did exist. Our interest lies in mathematicians in Princeton University.

The new offices constructed in gothic style gave Princeton an aura of a safe haven. Logicians from all over the world were invited to Princeton to build out a new department. While most of America couldn't find a piece of bread for dinner, high ceilings, walls covered with elaborately carved wood, daily discussions by a cup of tea, and walks in the forest were some of the conditions in Princeton.

One mathematician living in such lavish lifestyle was a young man named Alonzo Church. Alonzo received a B.S. degree from Princeton and was persuaded to stay for graduate school. Alonzo felt the architecture was fancier than necessary. He rarely showed up to discuss mathematics with a cup of tea and he didn't enjoy the walks in the woods. Alonzo was a loner: he was most productive when working on his own. Nevertheless Alonzo had regular contacts with other Princeton inhabitants. Among them were Alan Turing, John von Neumann, and Kurt Gödel.

The four men were interested in formal systems. They didn't pay much heed to the physical world, they were interested in dealing with abstract mathematical puzzles instead. Their puzzles had something in common: the men were working on answering questions about computation. If we had machines that had infinite computational power, what problems would we be able to solve? Could we solve them automatically? Could some problems remain unsolved and why? Would various machines with different designs be equal in power?

In cooperation with other men Alonzo Church developed a formal system called lambda calculus. The system was essentially a programming language for one of these imaginary machines. It was based on functions that took other functions as parameters and returned functions as results. The function was identified by a Greek letter lambda, hence the system's name4. Using this formalism Alonzo was able to reason about many of the above questions and provide conclusive answers.

Independently of Alonzo Church, Alan Turing was performing similar work. He developed a different formalism (now referred to as the Turing machine), and used it to independently come to similar conclusions as Alonzo. Later it was shown that Turing machines and lambda calculus were equivalent in power.

This is where the story would stop, I'd wrap up the article, and you'd navigate to another page, if not for the beginning of World War II. The world was in flames. The U.S. Army and Navy used artillery more often than ever. In attempts to improve accuracy the Army employed a large group of mathematicians to continuously calculate differential equations required for solving ballistic firing tables. It was becoming obvious that the task was too great for being solved manually and various equipment was developed in order to overcome this problem. The first machine to solve ballistic tables was a Mark I built by IBM - it weighed five tons, had 750,000 parts and could do three operations per second.

The race, of course, wasn't over. In 1949 an Electronic Discrete Variable Automatic Computer (EDVAC) was unveiled and had tremendous success. It was a first example of von Neumann's architecture and was effectively a real world implementation of a Turing machine. For the time being Alonzo Church was out of luck.

In late 1950s an MIT professor John McCarthy (also a Princeton graduate) developed interest in Alonzo Church's work. In 1958 he unveiled a List Processing language (Lisp). Lisp was an implementation of Alonzo's lambda calculus that worked on von Neumann computers! Many computer scientists recognized the expressive power of Lisp. In 1973 a group of programmers at MIT's Artificial Intelligence Lab developed hardware they called a Lisp machine - effectively a native hardware implementation of Alonzo's lambda calculus!

## Functional Programming

Functional programming is a practical implementation of Alonzo Church's ideas. Not all lambda calculus ideas transform to practice because lambda calculus was not designed to work under physical limitations. Therefore, like object oriented programming, functional programming is a set of ideas, not a set of strict guidelines. There are many functional programming languages, and most of them do many things very differently. In this article I will explain the most widely used ideas from functional languages using examples written in Java (yes, you could write functional programs in Java if you felt particularly masochistic). In the next couple of sections we'll take Java as is, and will make modifications to it to transform it into a useable functional language. Let's begin our quest.

Like this post? Continue reading the original post in [Slava Akhmechet's blog](http://www.defmacro.org/2006/06/19/fp.html)
