# Rspec Mechanics

At this point, you are very familiar with how an `rspec` test file appears. You've had to diagnosis them, drop breakpoints inside them, and perhaps even modified them.

Before we start writing our rspec tests, let's pick apart the elements we have been seeing in our labs.

First take note that test file naming follow a distinct convention. Each file maps to just one other class, which typically means just one other file. So if we have a `tire.rb` with a `Pants` class in our application, the corresponding test file would be called `tire_spec.rb`, and be located in the `spec` directory.

One important item to remember is that the `rspec` API is just plain Ruby. Some of its syntax appears different what we are used to, but these unusual looking styles are really just Ruby.

## Test Suite Layout

Looking at the top of a spec file, you've probably noticed this line:

```ruby
require "spec_helper"
```

The `spec_helper.rb` is the central configuration file for `rspec`. It is a file that handles requiring all the necessary files in the application, additional testing libraries, and also serves as the central place extending configuration directly into `rspec`.

There's no reason to get into how this file works right now. Just know its there, setting up your ability to use the `rspec` command in your shell.

As you move into Rails, you'll see this line change to `require "rails_helper"`. That's because Rails adds more configuration into our test files. However, the `spec_helper.rb` file still exists, and gets required inside the `rails_helper.rb`.

Spec files typically start by opening up one big `describe` block:

```ruby
require "spec_helper"

Rspec.describe Tire do

end
```

This first line is just a Ruby method that takes the name of the class you will be testing, and a second argument of a block of code (do/end).

## Describe and Context

Rspec provides a great way to allow your test code to serve as documentation. The `describe` and `context` methods help by taking 2 arguments. First argument is a string that should identify what you are testing. And the second is a block that contains tests, or more descriptions that contain tests.

These methods do not trigger a test themselves. Instead, they give form to the test file. Providing a layout for what our tests do.

`describe` and `context` perform the exact same way. So why have two methods that do the same thing? Well, our tests are semantic, or "relating to meaning in language". So how we may use the words "context" and "describe" in real life will influence how we group our tests.

```ruby
require "spec_helper"

Rspec.describe Tire do
  describe "repels liquid" do
    context "on short trips" do
      # some tests
    end

    context "on long trip" do
      # some tests
    end

    context "in rain storms" do
      # some tests
    end
  end

  describe "has tread" do
    # some tests
  end
end
```

In the example above, a section of our `Tire` tests groups testing how it "repels liquid" with `describe`. Within that scope, there will be tests concerned with "short trips", "long trips", and "in rain storms", grouped using `context`.

Later on, there will be tests concerning `Tire` tread. Since that does not have to do with repeling liquid, it is not in the same scope.

## It Blocks

Rspec uses an `it` method to signify the place where actual tests go. `it` takes a description string and a block of test code.

```ruby
Rspec.describe Tire do
  describe "repels liquid" do
    # ...

    context "in rain storms" do
      it "resists water" do
        expect(Tire.new.through_rainstorm).to eq("Repelled water!")
      end

      it "resists mud" do
        expect(Tire.new.through_muddy_rainstorm).to eq("Repelled mud!")
      end

      it "resists acid" do
        expect(Tire.new.through_acid_rainstorm).to eq("Repelled acid!")
      end
    end

  # ...
```

Running `rspec` on this test file produces a breakdown of the `describe`, `context`, and `it` strings:

```
Tire
  repels liquid
    in rain storms
      resists water
      resists mud
      resists acid
```


## Before vs Let vs Neither


## Many Features - What is Important
