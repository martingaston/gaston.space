---
title: Refactoring Gilded Rose in Elixir
date: "2019-03-28T10:53:24.030Z"
description: Working through the messy, tangled bedroom of code that is The Gilded Rose.
---

Firstly, it's named after stuff from Warcraft. I knew I recognised Sulfuras from somewhere. Ragnaros! [My original favourite Hearthstone legendary card](https://hearthstone.gamepedia.com/Ragnaros_the_Firelord). That was putting low-level pressure on my brain for hours.

But, more to the point: **refactoring**.

Like every other post on The Gilded Rose, it's worth pointing out that it's a [refactoring kata popularised by Emily Bache](https://github.com/emilybache/GildedRose-Refactoring-Kata). You're given a chunk of code as messy as a teenager's bedroom and have to tidy it up: you're its new beleaguered parent, after all.

There's quite a bit going on here, but I think it's also really useful from the perspective of picking up syntax and tooling you're not super familiar with - it's a bit of a lesson in inheriting legacy code, and although there's a good argument to sticking to a legacy language for maximum accuracy I think it's also a nice way to practice with a new language. Enter: Elixir, stage left. Wait - what's that behind it? Is it... an incredibly thorny 72-line function spread disharmoniously across two separate `cond` statements? *Avert your ey*-- oh no, it's too late!

```elixir
defmodule GildedRose do
  # Example
  # update_quality([%Item{name: "Backstage passes to a TAFKAL80ETC concert", sell_in: 9, quality: 1}])
  # => [%Item{name: "Backstage passes to a TAFKAL80ETC concert", sell_in: 8, quality: 3}]

  def update_quality(items) do
    Enum.map(items, &update_item/1)
  end

  def update_item(item) do
    item = cond do
      item.name != "Aged Brie" && item.name != "Backstage passes to a TAFKAL80ETC concert" ->
        if item.quality > 0 do
          if item.name != "Sulfuras, Hand of Ragnaros" do
            %{item | quality: item.quality - 1}
          else
            item
          end
        else
          item
        end
      true ->
        cond do
          item.quality < 50 ->
            item = %{item | quality: item.quality + 1}
            cond do
              item.name == "Backstage passes to a TAFKAL80ETC concert" ->
                item = cond do
                  item.sell_in < 11 ->
                    cond do
                      item.quality < 50 ->
                        %{item | quality: item.quality + 1}
                      true -> item
                    end
                  true -> item
                end
                cond do
                  item.sell_in < 6 ->
                    cond do
                      item.quality < 50 ->
                        %{item | quality: item.quality + 1}
                      true -> item
                    end
                  true -> item
                end
              true -> item
            end
          true -> item
        end
    end
    item = cond do
      item.name != "Sulfuras, Hand of Ragnaros" ->
        %{item | sell_in: item.sell_in - 1}
      true -> item
    end
    cond do
      item.sell_in < 0 ->
        cond do
          item.name != "Aged Brie" ->
            cond do
              item.name != "Backstage passes to a TAFKAL80ETC concert" ->
                cond do
                  item.quality > 0 ->
                    cond do
                      item.name != "Sulfuras, Hand of Ragnaros" ->
                        %{item | quality: item.quality - 1}
                      true -> item
                    end
                  true -> item
                end
              true -> %{item | quality: item.quality - item.quality}
            end
          true ->
            cond do
              item.quality < 50 ->
                %{item | quality: item.quality + 1}
              true -> item
            end
        end
      true -> item
    end
  end
end
```

## How do you even begin to unpick this?

Ignoring the fact that it probably looks exactly like a chunk of code you once came furiously across only to realise the incriminating finger of `git blame` was pointed back at you, it's safe to say, boy, this is some syntax in need of some TLC.

With that in mind, it's worth [checking the specifications for the code](https://github.com/emilybache/GildedRose-Refactoring-Kata/blob/master/GildedRoseRequirements.txt). In short, running `update_quality` on a List of `%Item{}` structs - which have `name`, `quality` and `sell_in` values, should accomplish various things based on certain conditions.

The wiggly worm of code is actually fully functional, bar the addition of a new feature that you're being asked to implement, so the obvious first step is to write tests for each story in the requirements. With this - as Sandi Metz would say - you've got the tests at your back, so you can start to refactor with confidence.

**Full disclosure**: Sandi Metz's '[All the Little Things](https://youtu.be/8bZh5LMaSmE)' talk from RailsConf 2014 is an absolute jackpot of a video. I'm not particularly familiar with Ruby or OOP but she absolutely and confidently takes you on the journey. I would go as far as to say it's one of the best tech talks I've ever seen, and her teachings are now irrevocably intertwined with my own thoughts on how to approach refactoring. I only wish I had watched it sooner.

## Out with the old...

With our tests in place it's safe to say we're feeling confident, so the first thing is to try and trap a condition - I chose Aged Brie, mainly because I was hungry at the time - and see if we can get all the tests to fail. So I squashed this at the top of the first `cond` expression...

```elixir
  def update_item(item) do
    item = cond do
      item.name == "Aged Brie" ->
        item

      ...
```

Job's done. This also reveals that the two `cond` statements separate out updating the `quality` and `sell_in` fields of our `Item` struct.  I'll come back to that, but right now I'm just rushing towards getting those tests back up and passing.

```elixir
      item.name == "Aged Brie" ->
        quality = item.quality + 1

        if item.quality > 50 do
          quality = 50
        end

        %{item | quality: quality }
```

**Warning:** this code doesn't work. But this was my very first attempt, now saved for future generations to gaze upon. It goes without saying that I'm still learning Elixir syntax. But a couple of jumped at this point:

- The `|` within that `Map` elsewhere in the code looked real interesting - it looks to be returning the original `item` but updating the `quality` key, and it turns out to be a nifty bit of shorthand for [updating existing keys](https://hexdocs.pm/elixir/Map.html#content).
- My use of the `if ` conditional clearly wasn't playing ball with the Elixir compiler - so a [trip to SO](https://stackoverflow.com/a/39550669) revealed that all conditionals in Elixir return from their expression.

```elixir
quality = if item.quality >= 50 do
  50
else
  item.quality + 1
end

%{item | quality: quality }
```

This is by no means idiomatic or efficient code, but it causes the tests to pass and helps us break out of the original code. And that's okay, we're on a journey.

## You're an archeologist now

One really key thing I picked up from doing this is to resist the temptation to initially delete **everything** and think, hey, let's start this whole thing again. That's how I'm used to working, but slowly migrating the code over and *then* deleting the old code blocks creates a much more straightforward refactoring process. Again, I think it's a case of confidence in the process - you'll get there eventually.

So in the code below I continue to trap out the old code with the first `cond` block before deleting the old expressions entirely:

```elixir
item = cond do
      item.name == "Aged Brie" ->
        quality = if item.quality >= 50 do
          50
        else
          item.quality + 1
        end
        %{item | quality: quality }

      item.name == "Sulfuras, Hand of Ragnaros" ->
        item

      item.name == "Backstage passes to a TAFKAL80ETC concert" ->
        quality_to_add = case item.sell_in do
          i when i < 0 -> -item.quality
          i when i < 5 -> 3
          i when i < 10 -> 2
          _ -> 1
        end
        %{item | quality: item.quality + quality_to_add}

      true ->
        quality = if item.quality <= 0 do
          0
        else
          item.quality - 1
        end
        %{item | quality: quality}
    end
```

Again, we're nowhere near idiomatic Elixir yet. There's plenty of work to be done. But we're already seeing progress. This is easier to read. Its intention is clearer. My headache is even getting better; man, that refactoring hits the spot even Advil® Cold & Sinus can't reach.

At this stage it's feels a bit like archaeology, slowly excavating away and unpicking the process. Peeling back layers until you can see things you didn't recognise before. It gets me to have my realisation that the second `cond` block is handling a couple of responsibilities:

 - Decrementing the `sell_in` value.
 - Further decrementing the `quality` of certain items in various confusing conditions.

But it's easier to see what's going on now, and we've started to build up our new bedrock of code, so once again we can start trapping conditions and moving functionality out of the old code and consolidating into the new code, ensuring the tests still pass.

## Starting to Bring it Together, Part I

With the flexibility and confidence of our iterative refactoring approach, we can start bringing in some of the more idiomatic features of the language - swapping our `cond` out with a `case` expression because we are matching against specific values.

```elixir
defmodule GildedRose do
  @sulfuras "Sulfuras, Hand of Ragnaros"
  @backstage_passes "Backstage passes to a TAFKAL80ETC concert"
  @aged_brie "Aged Brie"

  def update_quality(items) do
    Enum.map(items, &update_item/1)
  end

  def update_item(item) do
    case item.name do
      @sulfuras ->
        item

      @aged_brie ->
        quality = if item.quality >= 50, do: 50, else: item.quality + 1
        %{item | quality: quality, sell_in: item.sell_in - 1}

      @backstage_passes ->
        updated_quality = calc_backstage_pass_value(item)
        %{item | quality: item.quality + updated_quality, sell_in: item.sell_in - 1}

      _ ->
        sell_in = item.sell_in - 1
        multiplier = if sell_in < 0, do: 2, else: 1
        quality = if item.quality <= 0, do: 0, else: item.quality - 1 * multiplier
        %{item | quality: quality, sell_in: sell_in}
    end
  end

  defp calc_backstage_pass_value(%Item{} = item) do
    reset_to_zero = -item.quality
    add_two_value = 2
    add_three_value = 3
    add_one_value = 1

    case item.sell_in do
      days when days <= 0 -> reset_to_zero
      days when days < 5 -> add_three_value
      days when days < 10 -> add_two_value
      _ -> add_one_value
    end
  end
end
```

The night is still young for this particular piece of refactoring - there's a few too many magic numbers, I think there's more we can do with pattern matching and I think some of the logic (notably incrementing and decrementing quality) can be extracted out into additional methods to help eliminate some of that duplication.

But the code is already significantly more readable, and its intention is vastly clearer. This is code that now walks down the street with a little swagger - and we get to be particularly smug, proud parents about this.

To play us out, this advice from [Robert C. Martin's Clean Code](https://www.amazon.co.uk/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882) feels particularly poignant:

> When I write functions, they come out long and complicated. They have lots of indenting and nested loops. They have long argument lists. The names are arbitrary, and there is duplicated code. But I also have a suite of unit tests that cover every one of those clumsy lines of code.
>
> So then I massage and refine that code, splitting out functions, changing names, eliminating duplication. I shrink the methods and reorder them.
