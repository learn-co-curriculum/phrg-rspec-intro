# Rspec Mechanics

At this point, you are familiar with how spec test file looks. You have had to diagnosis them, drop breakpoints inside them, and perhaps even modified a test. Before we start writing our very `rspec` tests, let's pick apart the elements we have been seen.

Take note that test file naming follows a distinct convention. Each file typically maps to just one other class, or file. So if we have a `tire.rb` that defines a `Tire` class, the corresponding test file would be named `tire_spec.rb`, and be located in the `spec` directory.

As we explore `rspec` syntax, remember that its API is written in plain old Ruby. The files appear different what we are used to, but its just Ruby methods taking arguements in different formats. There is no magic in the `rspec` library.

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

Another convention found in many test files is the use of `before` and `let` blocks to set up test state. Since these are used quite a lot in the learn.co curriculum, lets understand their differences, benefits, and drawbacks.

Here is the [documention on using before & after hooks](https://relishapp.com/rspec/rspec-core/v/3-7/docs/hooks/before-and-after-hooks). And here are the [docs on using `let` and `let!`](https://relishapp.com/rspec/rspec-core/docs/helper-methods/let-and-let).

Let's use creating a valid T-Shirt as an example.

```ruby
Rspec.desrcibe TShirt do
  before do
    @valid_attributes = { size: "XS" }
    @invalid_attributes = { size: "XXXXXXXXXS"}
  end

  it "is valid with standard size" do
    expect(TShirt.new(@valid_attributes)).to be_valid
  end

  it "is invalid with too small a size" do
    expect(TShirt.new(@invalid_attributes)).to_not be_valid
  end

  context "other stuff" do
    # some other tests
  end
end
```

The `before` method takes an optional argument of a symbol for how often the its code will run. `before(:each)` and `before(:all)` are two examples. In the snippet above, no argument is specified, so `before` defaults to `:each`, which means the code will run before every `it` block.

While the snippet above looks appealing in some ways, there is actually a lot of extra code getting run. Not only will `@valid_attributes` and `@invalid_attributes` be assigned a value for both of our tests, but they will also be assigned for all the tests in the `"other stuff"` context. Those variable assignments are unnecessary as we keep adding tests, and so it will slow down our test run cycle. This is not a example use for `before`.


```ruby
Rspec.desrcibe TShirt do
  let(:valid_attributes) do
    { size: "XS" }
  end

  let(:invalid_attributes) do
    { size: "XXXXXXXXXS" }
  end

  it "is valid with standard size" do
    expect(TShirt.new(valid_attributes)).to be_valid
  end

  it "is invalid with too small a size" do
    expect(TShirt.new(invalid_attributes)).to_not be_valid
  end

  context "other stuff" do
    # some other tests
  end
end
```

Here is a similar approach, but using `let`. `let` takes a symbol and a block of code, and it executes lazily. Meaning, only if you invoke the `let` by using its symbol like a method invocation will the code inside the `let` be run. This is a much better approach than `before`. The `"is valid with standard size"` test only creates `valid_attributes`, the `"is invalid with too small a size"` only creates `invalid_attributes`, and whatever tests in `"other stuff"` are not invoking this code at all. We also don't need to use instance variables. Wonderful!

But wait. What happens as our test file grows.

```ruby
Rspec.desrcibe TShirt do
  let(:valid_attributes) do
    { size: "XS" }
  end

  let(:invalid_attributes) do
    { size: "XXXXXXXXXS" }
  end

  it "is valid with standard size" do
    expect(TShirt.new(valid_attributes)).to be_valid
  end

  it "is invalid with too small a size" do
    expect(TShirt.new(invalid_attributes)).to_not be_valid
  end

  context "color" do
    let(:valid_attributes) do
      { color: "green", size: "XS" }
    end

    let(:invalid_attributes) do
      { color: "transparent", size: "XS" }
    end

    it "is valid with acceptable color" do
      expect(TShirt.new(valid_attributes)).to be_valid
    end

    it "is invalid with unacceptable color" do
      expect(TShirt.new(invalid_attributes)).to_not be_valid
    end

    # more tests, with the possibility of more and more tests as time goes on
  end

  it "can be ripped" do
    shirt = TShirt.new(valid_attriubtes)
    expect(shirt.rip!).to eq("Oh no! I'm torn!")
  end

  # yep... many many more tests
end
```

Hmmmm. The last snippet is a very realistic example of many test files in the Ruby universe. The `let`s inside the `"color"` context use identical symbols to define their code. The tests below it use the first `valid` and `invalid` `attributes` definition they find, so that means the ones in the "color" scope.

In this very simple example, this may seem straight ahead. However, most real world test files are much larger and much more complex. They may juggle quite a number of scenarios to test `TShirt`. And what about that "rip" test? What attributes is it using? Why do I have to scan the entire content of the test file to write a test for a ripped TShirt?

### The better alternative

Let's not use these `rspec` helper methods and give this another try.


```ruby
Rspec.desrcibe TShirt do
  it "is valid with standard size" do
    expect(TShirt.new({ size: "XS" })).to be_valid
  end

  it "is invalid with too small a size" do
    expect(TShirt.new({ size: "XXXXXXXXXS" })).to_not be_valid
  end

  context "color" do
    it "is valid with acceptable color" do
      expect(TShirt.new({ color: "green", size: "XS" })).to be_valid
    end

    it "is invalid with unacceptable color" do
      expect(TShirt.new({ color: "transparent", size: "XS" })).to_not be_valid
    end

    # more tests
  end

  it "can be ripped" do
    shirt = TShirt.new({ size: "XS" })
    expect(shirt.rip!).to eq("Oh no! I'm torn!")
  end

  # more tests
end
```

This is the best test file yet. Because we are not using `before` nor `let`, we don't need to search around our test examples to figure out what values are in use. We have reduced the cognitive burn of understanding our test code, and thus made them easier to understand and edit/improve as our `TShirt` class evolves. We also quickly understand what are tests do NOT need.

This change did violate one Ruby tenant though. We've actually made the code less DRY (Don't Repeat Yourself) because we have repeated our `valid_attributes` (`{ size: "XS" }`) in the last test. This is a trade-off to this approach to testing. But it is much less of a trade-off compared to the pitfalls of our previous approaches.

Nitro tests suffer greatly from issues of over using `before` and `let` helper methods. As do some learn.co labs. Understanding these method drawbacks puts you a fantastic position to help Nitro clean up its tests.

## Many Features - What is Important

`rspec` has quite a lot of [features and helper methods](https://relishapp.com/rspec/rspec-core/v/3-7/docs). Many of them are very convenient, and many are helpful. But worrying too much about the "syntactic sugar" around your tests is not the point. What makes good tests?

As you gain experience, you will form your own opinions on what kind of tests provide the most value. Here are some questions to keep in mind as you start writing your own tests:

* Is the intent of my test clear?
* Am I coding only what is necessary for my test?
* Does my test enforce expectations that must stay intact?

Many developers have the mind set that simply having any tests is a step in the right direction. And they are right :).

Use examples of you've seen in the past, and the guidance of your fellow Power developers as you continue to learn.
