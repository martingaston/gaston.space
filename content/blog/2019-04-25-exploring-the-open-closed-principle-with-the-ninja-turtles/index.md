---
title: Exploring the Open/Closed Principle with the Teenage Mutant Ninja Turtles
date: "2019-04-25T16:48:31.117Z"
description: Understanding the second SOLID principle in the only way I know how.
---

After a [sojourn with the Single Responsibility Principle](https://gaston.space/2019-04-16-with-great-power-comes-great-single-responsibility/), I wanted to progress to the 'O' in SOLID: the Open/Closed Principle. 

Here's the original summary, shamelessly yoinked from [Wikipedia](https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle), and written way back in 1988 by Bertrant Meyer:  

> Software entities (Classes, modules, functions) should be open for extension, not modification.

*Caveat:* Some people call the Open/Closed Principle 'OCP' for reasons of brevity, but I can't think of the term OCP without thinking about [the nasty corporation from RoboCop](https://robocop.fandom.com/wiki/Omni_Consumer_Products) so I will never do it. Although, initially trying to wrap my head around the Open/Closed principle was about as much fun as stumbling upon an ED-209, so maybe it does fit. 

Let's dig into those two seemingly contradictory statements a little more: 

- **Open for Extension**: If we're working under the belief that the requirements of our software will change over time, and we probably *should*, being open for extension means the structure and design of our software can accomodate new changes and features as the needs adjust over time.
- **Closed for Modification**: On the other side of that coin, however, is the principle that the parts of our software must be closed for modification - there can be no more changes to the source code. This is especially true for systems that have reached production. 

Now, this is the part where my brain gets stuck in a loop for a bit and then I get a headache. How do you achieve both of those things at once? Compared to the Single Responsibility Principle, Open/Closed comes across as aloof and overly academic. Surely code like that doesn't **really** exist in the real world? 

The first way I started to make sense of it was by admitting, yeah, there's a level of abstraction to the concept itself. Let's sit back, breathe deeply and let it wash over us. At its core, my read of the principle is that we don't want to be making dramatic, significant and potentially destructive changes every time our product requirements change. If clean, well-maintained OO code is reusable, then we don't want to tie ourselves too directly to our classes. 

Or, think of it like this: in an ideal world we shouldn't repeatedly be coming back into the same file and making changes, especially if that file is now production code.

So, our product owner has given us the brief of skateboarding through the sewers of New York: 

```java
public class Main {  
    public static void main(String[] args) {  
        Turtle Michelangelo = new Turtle("Michelangelo", "Party Dude", new Eyemask(), new Weapon());  
    }  
}

public class Turtle {  
    private String name;  
    private String description;  
    private Eyemask eyemask;  
    private Weapon weapon;  
  
    public Turtle(String name, String description, Eyemask eyemask, Weapon weapon) {  
        this.name = name;  
        this.description = description;  
        this.eyemask = eyemask;  
        this.weapon = weapon;  
    }  
  
    public String getName() {  
        return name;  
    }  
  
    public String getDescription() {  
        return description;  
    }  
  
    public Eyemask getEyemask() {  
        return eyemask;  
    }  
  
    public int getDamage() {  
        return weapon.getWeaponDamage();  
    }  
}
}

public class Weapon {  
    private String name = "Nunchuks";  
    private int stickOneDamage = 9;  
    private int stickTwoDamage = 9;  
  
    public int getWeaponDamage() {  
        return stickOneDamage + stickTwoDamage;  
    }  
}

public class Eyemask {  
    private String colour = "Orange";  
}
```

That's great, but now our product owner has decided she wants a new feature - one with a little bit more technical know-how. Someone who does machines, if you will. So we can easily accomodate for that inside our `Weapon` and `Eyemask` classes. 

```java
public class Weapon {  
    private String name;  
    private int weaponDamage;  
  
    public Weapon(String name) {  
        this.name = name;  
  
        switch (this.name) {  
            case "Nunchuks":  
		int stickOneDamage = 9;  
                int stickTwoDamage = 9;  
                this.weaponDamage = stickOneDamage + stickTwoDamage;  
                break;  
            case "Bo Staff":  
		int staffDamage = 15;  
                this.weaponDamage = staffDamage;  
                break;  
            default:  
		int fistsDamage = 4;  
                this.weaponDamage = fistsDamage;  
                break;  
        }  
    }  
  
    public int getWeaponDamage() {  
        return this.weaponDamage;  
    }  
}

public class Eyemask {  
    String colour;  
    public Eyemask(String colour) {  
        this.colour = colour;  
    }  
}

public class Main {  
    public static void main(String[] args) {  
        Turtle Michelangelo = new Turtle(  
                "Michelangelo",   
                "Party Dude",   
                new Eyemask(),    // drat
                new Weapon()      // double drat
        );  
        Turtle Donatello = new Turtle(  
                "Donatello",   
                "Does Machines",   
                new Eyemask("Purple"),   
                new Weapon("Bo Staff")  
        );  
    }  
}
```

Egads! We've gone and broken our app because we now need to supply arguments to the constructors of `Eyemask` and `Weapon`, which we didn't need to do before. An easy fix for this example, but this highlights one of the intentions behind the Open/Closed Principle - a little bit of tinkereing here and there and it's pretty easy to imagine how we could easily break a large whack of our code. That's why we want to keep them **closed** to modification. 

Also - look at that thorny `case` expression. Yikes. That's an obvious code smell and we haven't even got around to bringing in Leonardo and Raphael yet, which seems like extremely plausible new features for our app. 

In a statically compiled language like Java, one answer is to incorporate explicit interfaces to help clarify the API of what you're passing through - as most OO designers recommend composition over inheritance, this has been the preffered style since the 90s. With interfaces we could declare our API for implementing each turtle's weapon and have `getWeaponDamage` part of each one of those methods. We'd be able to *extend* our application with new features *without* modifing our existing code. 

```java
public class Main {  
    public static void main(String[] args) {  
        Turtle Michelangelo = new Turtle(  
                "Michelangelo",  
                "Party Dude",  
                new Eyemask("Orange"),  
                new Nunchuk()  
        );  
        Turtle Donatello = new Turtle(  
                "Donatello",  
                "Does Machines",  
                new Eyemask("Purple"),  
                new BoStaff()  
        );  
        Turtle Raphael = new Turtle(  
                "Raphael",  
                "Cool but crude",  
                new Eyemask("Red"),  
                new Sai()  
        );  
    }  
}

public interface Weapon {  
    int getWeaponDamage();  
}

public class BoStaff implements Weapon {  
    private String name = "Bo Staff";  
    private int staffDamage = 15;  
  
    @Override
    public int getWeaponDamage() {  
        return this.staffDamage;  
    }  
}

public class Nunchuk implements Weapon {  
    private String name = "Nunchuks";  
    private int stickOneDamage = 9;  
    private int stickTwoDamage = 9;  
  
    @Override
    public int getWeaponDamage() {  
        return this.stickOneDamage + this.stickTwoDamage;  
    }  
}

public class Sai implements Weapon {  
    private String name = "Twin Sai";  
    private int firstSaiDamage = 7;  
    private int secondSaiDamage = 7;  
  
    @Override  
    public int getWeaponDamage() {  
        return firstSaiDamage + secondSaiDamage;  
    }  
}

public class Eyemask {  
    String colour;  
    public Eyemask(String colour) {  
        this.colour = colour;  
    }  
}

public class Turtle {  
    private String name;  
    private String description;  
    private Eyemask eyemask;  
    private Weapon weapon;  
  
    public Turtle(String name, String description, Eyemask eyemask, Weapon weapon) {  
        this.name = name;  
        this.description = description;  
        this.eyemask = eyemask;  
        this.weapon = weapon;  
    }  
  
    public String getName() {  
        return name;  
    }  
  
    public String getDescription() {  
        return description;  
    }  
  
    public Eyemask getEyemask() {  
        return eyemask;  
    }  
  
    public int getDamage() {  
        return weapon.getWeaponDamage();  
    }  
}
```

Interfaces allow us to play without implementation, but they also increase our boilerplate. They require a strong API that's unlikely to change because interfaces bind us to our implementation, or at the very least require a lot of fiddling around if you want to make any updates.

It's also, well, highly likely that if you're going to convert something to conform to the Open/Closed principle you're still going to have to make a few changes before you're in a position to stop making changes. 

Like most design patterns, the best time to implement the Open/Closed Principle is when you need it - not before and certainly not a long time after. It's an effective way to help keep your code reusable, but it adds some complexity and a lot of boilerplate that's just not worth incorporating before you need it. 

Like OO master Sandi Metz says, [you should prefer duplication over the wrong abstraction](https://www.sandimetz.com/blog/2016/1/20/the-wrong-abstraction). Cowabunga! 
