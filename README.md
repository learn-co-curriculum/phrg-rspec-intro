# RSpec Anatomy

At this point, you are familiar with how spec test file looks. You have had to diagnosis them, drop breakpoints inside them, and even modified a test. Before we start writing our own `rspec` tests, let's pick apart the elements we have seen.

![RSpec Anatomy](images/rspec-anatomy.jpg?raw=true)

RSpec is a big framework. It is a hosted Domain Specific Language (DSL) for testing. Even though it might not always look like it, RSpec code is Ruby. RSpec leverages the syntax and language features of Ruby to provide a specialized language for writing tests. Here we are going to intorduce the different parts of a RSpec test file. For this introduction we are going to look at an example:

```ruby
require "spec_helper"

RSpec.describe Person, type: :model do
  describe "#first_name" do
    it "is the first name for the person" do
      person = described_class.new(first_name: "Expected First Name")

      expect(person.first_name).to eq("Expected First Name")
    end
  end

  describe "#first_name" do
    it "is the first name for the person" do
      person = described_class.new(last_name: "Expected Last Name")

      expect(person.last_name).to eq("Expected Last Name")
    end
  end

  describe "#full_name" do
    it "is the first name followed the last name for the person" do
      person = described_class.new(first_name: "Expected First Name" last_name: "Expected Last Name")

      expect(person.last_name).to eq("Expected First Name Expected Last Name")
    end
  end
end
```

Lets zoom in and take a look at some of the parts one by one.

# Spec Helper

```ruby
require "spec_helper"

...
```

It is common to configure RSpec to suit the needs and preferences of the project. It is very common to see a spec start off with a line that requires a RSpec helper file. This will usually be `require "spec_helper"` or `require "rails_helper"`. To learn more about [RSpec configuration options read the RSpec documentation](https://relishapp.com/rspec/rspec-core/v/3-7/docs/configuration).

# Example Groups

```ruby
...

RSpec.describe Person, type: :model do
  describe "#first_name" do
    it "is the first name for the person" do
      person = described_class.new(first_name: "Expected First Name")

      expect(person.first_name).to eq("Expected First Name")
    end
  end

  ...
```

Lets zoom in the first line of this, and brake down what is going on.

```ruby
RSpec.describe Person, type: :model do
```

This is a [Example Group definition](https://relishapp.com/rspec/rspec-core/v/3-7/docs/example-groups/basic-structure-describe-it). In RSpec our tests are called examples, and we group them together using an example group. The top most example group definition gets prefixed with `RSpec`. Here we are defining a example group that describes the `Person` object as the subject. You will often also see a descriptive string used as the value for the subject. We are passing [meta data](https://relishapp.com/rspec/rspec-core/v/3-7/docs/metadata) to the example group here. This lets us provide extra configuration for this example group. The meta data is not required, and addition meta data could be assigned. We pass a block to the example group. This block will contain examples and or more example groups. RSpec provides a `context` method that lets us do the same thing as `descirbe`. `context is an alias for `describe`.

```ruby
...

describe "#first_name" do

...
```

Here we see a describe with a string. This creates an example group for the `first_name` instance method on the Person object. It is convention to describe instance methods by prefixing the method name with a `#`. To describe a class method convention dictates prefixing with a `.`. There will be times when you are creating an example group that does not describe a method. In those causes the subject will be a descriptive string.

# Examples

```ruby
...

it "is the first name for the person" do

...
```

Tests in RSpec are called examples. They are declared with an `it` method. Here we are passing in a string `"is the first name for the person"` to describe the expected behavior of the interaction under test. The `it` method takes a block. The block defines the code which is executed when RSpec 'runs the example'. This code exercise the and verifies expectation stated in the example's description

# Expectations and Matchers

```
...

expect(person.first_name).to eq("Expected First Name")

...
```

Expectations are the part of an example the verifies code has the expected behavior. Here was are saying we expect the value returned by sending the message `first_name` to the `person` object, to return an object that is equal to the string `"Expected First Name"`. There is a lot going on here, and the syntax used is hiding some of the details. Lets take a look at what this method is doing with the missing syntax add back in:

```ruby
expect(person.first_name).to(eq("Expected First Name"))
```

We could further expand this to:

```ruby
# Capture the value returned by sending person the first_name message
returned_first_name = person.first_name

# Define a expectation object that we can make expectations of
expectation_object = expect(returned_first_name)

# Define a matcher for comparing the returned_first_name value
matcher = eq("Expected First Name")


# Put it all together, and make the expectation
expectation_object.to(matcher)
```

You can see we have a object being created with the `expect` method. This takes a value, and lets us then make expectations on the value it contains. Next we define a matcher. The matcher is sent to the expectation object via the `to` method, and the expectation object then check if the matcher holds true. If it does, the expectation passes, if it does not, the test will fail. You can read more about the built in [expectation and matching library of RSpec here](https://relishapp.com/rspec/rspec-expectations/docs).

# File Naming Conventions

It is best to end a spec file with `_spec.rb`, and to place it under the `spec` directory. This will make RSpec identify it, and make running it easier. There are some other conventions related to naming and location that are good to keep in mind. If the above `Person` was a file in a Ruby Gem type project and lived under `lib/gem_name/person.rb` folder, a good place to put this `Person` spec would be `spec/person_spec.rb`. If this was a Rails app, and the `Person` was a file that lived in `app/modles/person.rb`, a good place for this file would be `spec/modles/person_spec.rb`.


# RSpec Output

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

This is the best test file yet. Because we are not using `before` nor `let`, we don't need to search around our test examples to figure out what values are in use. We have reduced the cognitive burn of understanding our test code, and thus made them easier to understand and edit/improve as our `TShirt` class evolves. We also quickly understand what are tests do NOT need. `before` and `let` can easily lead to the testing [Obscure Test](http://xunitpatterns.com/Obscure%20Test.html), [Fragile Test](http://xunitpatterns.com/Fragile%20Test.html#Fragile%20Fixture), and Slow Tests](http://xunitpatterns.com/Slow%20Tests.html) testing smells. Avoiding `before` and `let` can help us avoid these smells.

Nitro tests suffer greatly from issues of over using `before` and `let` helper methods. As do some learn.co labs. Understanding these method drawbacks puts you a fantastic position to help Nitro clean up its tests. To learn more about why `before` and `let` should be avoided, [read this](https://robots.thoughtbot.com/lets-not).

# Example structure

When we write examples, we should try to structure the example so it tells a story. Good testing stories take this form:

1. Setup
1. Exercise
1. Verify
1. Teardown

Or:

1. Setup
1. Exercise / Verify
1. Teardown

In a language like Ruby we can usually ignore Teardown. So lets just focus on Setup, Exercise, and Teardown. This is called a [Four-Phase Test and can be read about here](https://robots.thoughtbot.com/four-phase-test).

## Setup

This is where we create the testing fixtures and setup the system under test.

## Exercise

Use and exercise the system under test.

## Verify

Verify that the exercise step produced the expected behavior.

# Many Features - What is Important

`rspec` has quite a lot of features and helper methods. Many of them are very convenient, and many are helpful. But worrying too much about the "syntactic sugar" around your tests is not the point. What makes good tests?

As you gain experience, you will form your own opinions on what kind of tests provide the most value. Here are some questions to keep in mind as you start writing your own tests:

* Is the intent of my test clear? What is my subject? What am I describing, and how am I verifying that?
* Am I coding only what is necessary for my test?
* Does my test enforce expectations that must stay intact?

Many developers have the mind set that simply having any tests is a step in the right direction. The is wishful thinking. A poorly written can be slow, and brittle, and it might not verify what it says it will. The best way to learn what is the right test and how to wright them is to start trying it.

Use examples of you've seen in the past, and the guidance of your fellow Power developers as you continue to learn.
