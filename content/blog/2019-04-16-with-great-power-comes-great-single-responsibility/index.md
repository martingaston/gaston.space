---
title: With Great Power Comes Great Single Responsibility
date: "2019-04-16T17:15:50.472Z"
description: Applying the most famous OOP design pattern of all.
---

Here's a truth universally acknowledged: it is **impossible** to write about the Single Responsibility Principle without mentioning Robert C. Martin - aka the superclass of the SOLID principles - who coined the term at some point in the *past* and gently nurtured it until it was perhaps the single biggest and most popular idiom of responsible OOP design.

That's right, the Single Responsibility Principle is to OOP what *Friends* is to sitcoms. 

To save you having to open Wikipedia in another tab, here's the famous summary that I've lifted straight out of Chapter 10 of *[Clean Code](https://www.amazon.co.uk/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882/ref=sr_1_1?keywords=clean%20code&qid=1555325666&s=books&sr=1-1)*: 
> The Single Responsibility Principle states that a class or module should have one, and only one, reason to change.  

Now, Clean Code is a mighty fine book that's well worth reading, yet I've always felt the correct response to that quote is to quietly utter a few "mmms" and nod your head sagely - but I've always found it a pretty tricky sentence to unpack. *What* is change? What are the *reasons* for change?

I think one of the stumbling blocks here, at least for me, is how these concepts can, and in the real world likely *will*, mean different things to different people. We won't all interpret change in the same way, which means the Single Responsibility Principle becomes something that we can use to create uniformity and alignment across teams. We often focus on the 'Single Responsibility' and ignore the 'Principle', but it's through adherence to the latter that we can create a common design structure a whole team can follow. 

Let's cook up an very basic illustration of a game of Connect 4 in Ruby: 

```ruby
class Connect4
  def generate_empty_state
    # generates an empty board state
  end

  def update_board_state
    # updates our board state

  def display_board_in_stdout
    # displays our state in the terminal/stdout
  end

  def get_input_from_user
    # updates the board state
  end

  def play
    # play the game
  end
end
```

What's going on here? `Connect4` is responsible for tracking the state of the board *and* displaying the board to the terminal *and* getting input from the user. We're creating a burden of responsibility. In our small, contrived example this is probably no big deal, but what happens if (or when) our requirements change? 

`Connect4` currently has multiple potential opportunities for change: we could look to change how we display our board, or adjust how the rules are calculated, or change our data structure for managing the board state itself. All of these potential changes are currently tightly coupled within this class, and it's impossible to reuse any of this code elsewhere. 

Of course, maybe you won't need to reuse any of this code. Maybe you don't need to apply a design pattern to this. This is one thing that I find a little tricky: there's a conceptual side to implementing these patterns. The code runs without them, so learning patterns is less of the kind of binary - it either works or it doesn't - feedback that you'd get from learning new syntax. In a production application it's never going to be as clear cut as you want it to be, either. 

So, okay, you'll have to take a leap of faith with me. We need to apply some design principles to our theoretical Connect 4 example, and we're going to change it into a GUI app  There's a lot of money riding on this and the deadlines are pretty strict. The stakes are high. We've all had a chat and we're going to break out the single responsibility principle. It's go time. 

The idea, of course, is relatively straightforward. An unweidly application is better maintained if broken down into distinct, individual pieces which encapsulate their data and interact with one another via their behaviour/methods. Shorter methods are easier to comprehend and update, and shorter classes ensure generally ensure they remain focused. 

One mental model I like to follow is from Sandi Metz's *[Practical Object-Oriented Design In Ruby](https://www.amazon.co.uk/Practical-Object-Oriented-Design-Agile-Primer/dp/0134456475/ref=dp_ob_title_bk)*, because Sandi Metz is basically OOP incarnate: 

> Another way to hone in on what a class is actually doing is to attempt to describe it in one sentence. Remember tht a class sould do the smallest possible useful thing. That thing ought to be simple to describe. If the simplest description you can devise uses the word "and", the class likely has more than one responsibility. 

That's right, this the same judgement we originally applied a few paragraphs ago. It works a treat! 

So far, so good. Small = bonza. Big = inevitable heartache down the line. 

One approach might be to stuff our new business requirements as some additional small methods into our big current class

```ruby
class Connect4
  def generate_empty_state
    # generates an empty board state
  end

  def update_board_state
    # updates our board state

  def display_board_in_stdout
    # displays our state in the terminal/stdout
  end

  def display_board_in_gui
    # displays our state in our gui
  end

  def get_input_from_user_cli
    # updates the board state from the cli/terminal app
  end

  def get_input_from_user_gui
    # updates the board state from gui input (e.g. mouse click)
  end

  def play
    # play the game
  end
end
```

This isn't always a bad start, to be honest, and this spike helps find patterns of duplication - which is usually a good sign in OOP that we can extract this functionality into its own class. I'd be hesistant to leave this as our *final* implementation for these new requirements, though, as we'd probably be creating a jolly bad time for our future selves if there were ever to be any new requirements added. We're still not really creating *single* responsibilities for our classes, so it might be a good idea to extract these methods out into classes of their own. 

Our display and input classes will probably feature quite a bit of duplication, so this is a good place to start. 

```ruby
class Connect4
  def initialize(board_state, board_display, board_input)
    @board_state = board_state.new
    @board_display = board_display.new(@board_state)
    @board_input = board_input.new(@board_state)
  end

  def play
    # play the game
  end
end

class BoardState
  attr_reader :board
  def initialize(board_state)
    @board = create_blank_board
  end
  
  def create_blank_board
    # generates an empty board state
  end

  def update_board
    # updates the board state
  end
end

class BoardDisplay
  def initialize(board_state)
    @board_state = board_state
  end
  
  def display_board
    # render away
  end
end

class BoardInput
  def initialize(board_state)
    @board_state = board_state
  end

  def get_input
    # get all the needed input
  end
end
```

You'll see, in making our classes smaller we've made the codebase larger. There's a chunk of boilerplate in all the `initialize` methods, even though we've just straight up removed the repetition from the `display` and `get_input` methods. 

`BoardDisplay` and `BoardInput` are now essentially interfaces (though there is no formal concept of interfaces in Ruby) to their respective responsibilities, and can be composed to encapsulate the internal logic of taking user input and rendering the board. We've separated out those concerns - the `BoardState` doesn't need to know *how* to display anything, just that it can call another object that responds to the `display_board` method, and we can create modularity in our code by creating classes which conform to its overall shape - Ruby's duck typing means that any class with a `display_board`, whether it displays to a GUI, CLI or even a complicated series of LED lights hooked up to a Raspberry Pi, can be slotted in as needed. 

Where I start to get unstuck is in the process of bringing it all back together. I've designed a bunch of individual, testable classes and now I need to combine them to create my overall application, which is somewhat more intricate than the contrived examples of `Car` or `Animal` we tend to see in OO examples. After all, the thread needs to start *somewhere*. This is where `Connect4` sits, which essentially 'builds' the application in its constructor (via having those individual objects passed into it at instantiation, commonly known as dependency injection) and then runs everything together with its `play` method. 

There's still plenty of opportunities to refactor the code further to give it a more robust design - I'm not delighted by having the `BoardState` data structure directly exposed with `attr_reader`, for instance - but with a few revisions to the design we've made the system more modular and open to *future* change if (or when) it arrives. It's almost certainly going to be more work upfront, but by making use of the single responsibility principle we can help protect us from big codebase-busting changes and give us an easier time as developers to make changes when they arise. 
