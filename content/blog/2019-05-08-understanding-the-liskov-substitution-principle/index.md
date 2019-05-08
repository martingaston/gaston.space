---
title: Where's my inheritance? Understanding the Liskov Substitution Principle 
date:  "2019-05-08T09:36:39.873Z"
description: How to stop a good inheritance going real bad. 
---

What's the Liskov Substitution Principle? In the SOLID team, Liskov must have been subbed on more times than Olivier Giroud during his Arsenal tenure. In all seriousness, Barbara Liskov is a totally awesome [pioneer of computer science whose Wikipedia page is well worth a read](https://en.wikipedia.org/wiki/Barbara_Liskov). 

The design principle, essentially, specifies that a subclass should be substitutable for its superclass. Let's stop right there just to clarify some of that jargon: 

- **Superclass**: The *parent* class (object) being inherited from. 
- **Subclass**: The *child* class receiving (via the `extends` keyword) the inheritance. 

Yeah, we're talking about inheritance. Now, you probably do what I do when you hear the word *inheritance*: format your computer and run for the hills. It's the safest way.

Inheritance is powerful, sure. Having easy, natural access to all those `Array` and `String` methods in JavaScript is great. Getting all that functionality from descending from `Object` in Ruby is pretty awesome. I'm usually pretty delighted to use inheritance when it's baked into the language spec. But maybe it's also too powerful, like the Hulk, and when it loses control it smashes up your application and breaks your heart. I don't think we should always avoid inheritance, but I do think being nervous around it is a healthy, survivalist instinct. It's fire. 

To illustrate the situation, let's totally violate the Liskov Substitution Principle. Let's [create tests](https://jestjs.io/) for a prominent and respectable business tycoon, who can give interviews and work for their enormous fortune. Then let's build a quick class to pass our contrived example. 

### liskov.test.js
```javascript
const Liskov = require("./liskov");

describe("Tycoon", () => {
  const tycoon = new Liskov.Tycoon();
  it("Gives Long Interviews To Big Newspapers", () => {
    expect(tycoon.interview()).toBeTruthy();
  });

  it("Goes to work for their fortune", () => {
    expect(tycoon.work()).toBeTruthy();
  });
});
```

### liskov.js
```javascript
class Tycoon {
  constructor() {
    // initialisation
  }

  work() {
    return true;
  }

  interview() {
    return true;
  }
}
```

Awesome! What an impressive display of industrious capitalism. Now let's create a child `Socialite` class of our `Tycoon`, who should be able to neatly get swapped in to any instance we might want to use our `Tycoon` to do glamorous interviews or enterprising work. 

```javascript
class Socialite extends Tycoon {
  constructor() {
    // initialisation
    super();
  }

  work() {
    throw new Error("Sounds horrible");
  }

  interview() {
    return true;
  }
}
```

Drat. Our industry, the backbone of our economy, will *totally collapse* if we try and swap in our `Socialite` class! 

We're inheriting our classes, then, but their intended **behaviour** is all different even though they can often look the same. Commonly, inheritance is used to refer to `IS-A` relationships that can inherit via (language permitting) class or interface. Academically it can [boil down to the Square/Rectangle problem](https://en.wikipedia.org/wiki/Circle-ellipse_problem), and in real life in can be neatly distilled to everything breaking two minutes before you were due to go home for the day. 

Violating the Liskov Substitution Principle is great way to realize how the real-world concept of objects doesn't directly map to object oriented *programing*, which totally kind of scared me when I first realised it as it would totally undo some of those contrived 'intro to OOP' examples, rife with analogy, we're so used to seeing.  

In its place within the SOLID principles, Liskov Substitution sits quite happily alongside the Open/Closed Principle, which states that a class should be closed for modification - in short, [we should override the original class to add new functionality to our application](https://gaston.space/2019-04-25-exploring-the-open-closed-principle-with-the-ninja-turtles/). Our fancy new subclasses can bring additional functionality to the party, but they'll still be able to hit all of the same notes as the classes they're descending from. 

It's always worth remembering not to over-engineer ourselves, though, and not to add the complexity of these patterns until they're needed to scale our application further. Together, and used in the appropriate situations, the Liskov Substitution Principle is another design pattern that can help us plan for the inevitability of change in our applications as they grow. 
