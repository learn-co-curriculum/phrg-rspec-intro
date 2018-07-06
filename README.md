# RSpec Anatomy

At this point, you are familiar with how spec test files look. You have had to diagnosis them, drop breakpoints inside them, and perhaps even modify a test. Before we start writing our own `rspec` tests, let's pick apart the elements we have seen.

![RSpec Anatomy](https://raw.githubusercontent.com/powerhome/phrg-rspec-intro/master/rspec-anatomy.jpg?raw=true "RSpec Anatomy")

RSpec is a big framework. It is a hosted Domain Specific Language (DSL) for testing. Even though it might not always look like it, RSpec code is just plain Ruby. RSpec leverages the syntax and language features of Ruby to provide a specialized language for writing tests. For this intro, let's take a look at this example:

```ruby
require "spec_helper"

RSpec.describe Person, type: :model do
  describe "#first_name" do
    it "is the first name for the person" do
      person = described_class.new(first_name: "Expected First Name")

      expect(person.first_name).to eq("Expected First Name")
    end
  end

  describe "#last_name" do
    it "is the last name for the person" do
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

## Spec Helper

```ruby
require "spec_helper"
```

It is common to configure RSpec to suit the needs and preferences of a project, and very common to see a spec start off with a line that requires a RSpec helper file that handles this need. This usually looks like `require "spec_helper"` or `require "rails_helper"`. To learn about [RSpec configuration options, check out the RSpec documentation](https://relishapp.com/rspec/rspec-core/v/3-7/docs/configuration).

## Example Groups

```ruby
RSpec.describe Person, type: :model do
  describe "#first_name" do
    it "is the first name for the person" do
      person = described_class.new(first_name: "Expected First Name")

      expect(person.first_name).to eq("Expected First Name")
    end
  end
```

Let's zoom in the first line of this, and break down what's going on.

```ruby
RSpec.describe Person, type: :model do
```

This is an [Example Group definition](https://relishapp.com/rspec/rspec-core/v/3-7/docs/example-groups/basic-structure-describe-it). In RSpec, our tests are called examples, and we group them together using an example group. The top most example group definition gets prefixed with `RSpec`. Here we are defining an example group that describes the `Person` object as the subject. You will often also see a descriptive string used as the value for the subject. We are passing [meta data](https://relishapp.com/rspec/rspec-core/v/3-7/docs/metadata) to the example group here. This lets us provide extra configuration for this example group. Meta data is not required, and we also could have passed additional meta data.

We pass a block to the example group. This block will contain examples and also more example groups. RSpec provides a `context` method that lets us do the same thing as `describe`. They are aliases of one another.

Since `describe` and `context` perform the exact same way, why have two methods that do the same thing? Well, our tests are semantic, or "relating to meaning in language". So how we may use the words "context" and "describe" in real life will influence how we group our tests.

```ruby
describe "#first_name" do
```

Here we see a `describe` with a string. This creates an example group for the `first_name` instance method on the `Person` object. It is convention to describe instance methods by prefixing the method name with a `#`. To describe a class method convention dictates prefixing with a `.`. There will be times when you are creating an example group that does not describe a method. In those cases, the subject will be a descriptive string.

## Examples

```ruby
it "is the first name for the person" do
```

Tests in RSpec are called examples. They are declared with an `it` method. Here we are passing in a string `"is the first name for the person"` to describe the expected behavior of the interaction under test. The `it` method takes a block. The block defines the code which is executed when RSpec 'runs the example'. This code exercises and verifies the expectation stated in its description.

## Expectations and Matchers

```ruby
expect(person.first_name).to eq("Expected First Name")
```

Expectations are the parts of an example that verifies code has the correct behavior. Here we are asserting that we expect the value returned by sending the message `first_name` to the `person` object to return an object that is equal to the string `"Expected First Name"`. There is a lot going on here, and the syntax used is hiding some of the details. Lets take a look at what this method is doing with the missing syntax add back in:

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

You can see we have an object being created with the `expect` method. This takes a value, and lets us then make expectations on the value it contains. Next we define a matcher. The matcher is sent to the `expectation_object` via the `to` method, and the `expectation_object` checks if the matcher holds true. If it does, the expectation passes. If it doesn't, the test fails. You can read more about the built in [expectation and matching library of RSpec here](https://relishapp.com/rspec/rspec-expectations/docs).

## File Naming Conventions

It is best to end a spec file with `_spec.rb`, and to place it under the `spec` directory. This is a convention of RSpec, and how it comes configured out of the box. There are some other conventions related to naming and location that are good to keep in mind. If the above `Person` was a file in a Ruby Gem type project and lived at `lib/gem_name/person.rb`, a good place to put this `Person` spec would be `spec/person_spec.rb`. If this was a Rails app, and the `Person` was a file that lived in `app/models/person.rb`, then a good place for this would be `spec/models/person_spec.rb`.

## RSpec Output

Running `rspec` on this test file produces a breakdown of the `describe`, `context`, and `it` descriptions:

```
Person
  #first_name
    is the first name for the person
  #last_name
    is the last name for the person
  #full_name
    is the first name followed the last name for the person
```

## Before vs Let vs Neither

Another convention found in many test files is the use of `before` and `let` blocks to set up test state. Since these are used quite a lot in the learn.co curriculum, let's understand their differences, benefits, and drawbacks.

Here is the [documention on using before & after hooks](https://relishapp.com/rspec/rspec-core/v/3-7/docs/hooks/before-and-after-hooks). And here are the [docs on using `let` and `let!`](https://relishapp.com/rspec/rspec-core/docs/helper-methods/let-and-let).

Let's use creating a valid T-Shirt as an example. (Remember that `context` is an alias for `describe`.)

```ruby
Rspec.describe TShirt do
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

While the snippet above looks appealing in some ways, there is actually a lot of extra code getting run. Not only will `@valid_attributes` and `@invalid_attributes` be assigned a value for both of our tests, but they will also be assigned for all the tests in the `"other stuff"` context. Those variable assignments are unnecessary as we keep adding tests, and so it will slow down our test run cycle. This is not a great use for the `before` helper method.

Let's try writing the same tests with `let`:

```ruby
Rspec.describe TShirt do
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

`let` takes a symbol and a block of code, and executes lazily. Meaning, only if you invoke the `let` by using its symbol invocation will the code inside the `let` be run. This is a much better approach than `before`.

In our spec file using `before`, the `@valid_attributes` and `@invalid_attributes` objects were created in both the `"is valid with standard size"` and `"is invalid with too small a size"` examples. That's 4 object creations.

In our spec file using `let`, the `valid_attributes` object was only created for the `"is valid with standard size"` example. And the `invalid_attributes` object was only created for the `"is invalid with too small a size"` example. That's 2 object creations. (We also don't need to use instance variables. Which is wonderful!)

Finally, when using `let`, whatever tests may be in the `"other stuff"` `context` will not create an `attributes` object unless that object is explicitly invoked. Using the `before` approach will continue to create these extra things. And as our test file grows, that means its execution time will grow longer, needlessly.

But wait. What happens when our spec file starts using more and more `let`s?

```ruby
Rspec.describe TShirt do
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
Rspec.describe TShirt do
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

This is the best test file yet. Because we are not using `before` nor `let`, we don't need to search around our test examples to figure out what values are in use. We have reduced the cognitive burn of understanding our test code, and thus made them easier to understand and edit/improve as our `TShirt` class evolves. We also quickly understand what our tests do NOT require.

`before` and `let` can easily lead to the testing [Obscure Test](http://xunitpatterns.com/Obscure%20Test.html), [Fragile Test](http://xunitpatterns.com/Fragile%20Test.html#Fragile%20Fixture), and [Slow Test](http://xunitpatterns.com/Slow%20Tests.html) coding smells. Avoiding `before` and `let` can help us avoid these problems.

Nitro tests suffer greatly from issues of over using `before` and `let` helper methods. As do some learn.co labs. Understanding these method drawbacks puts you in a fantastic position to help Nitro clean up its tests. To learn more about why `before` and `let` should be avoided, [read this](https://robots.thoughtbot.com/lets-not).

## Example structure

When we write examples, we should try to structure the example so it tells a story. Good testing stories take this form:

1. Setup
1. Exercise
1. Verify
1. Teardown

Or:

1. Setup
1. Exercise / Verify
1. Teardown

This is called a [Four-Phase Test and can be read about here](https://robots.thoughtbot.com/four-phase-test).

* Setup - Prepare the system to test our scenario. Typically this requires us to setup an object(s)'s state so we may make assertions about the behavior we intend to test.

* Exercise - Use and exercise the system under test.

* Verify - Verify the exercise step produced the expected behavior.

* Teardown - System under test is reset to its pre-setup state. (This is often handled internally by `rspec`.)

## Many Features - What is Important

`rspec` has quite a lot of features and helper methods. Many of them are very convenient, and many are helpful. But worrying too much about the "syntactic sugar" around your tests is not the point. What makes good tests?

As you gain experience, you will form your own opinions on what kind of tests provide the most value. Here are some questions to keep in mind as you start writing your own tests:

* Is the intent of my test clear? What is my subject? What am I describing, and how am I verifying that?
* Am I coding only what is necessary for my test?
* Does my test enforce expectations that must stay intact?

Many developers have the mind set that simply having any tests in their project is a step in the right direction. And they are generally right! But, a poorly written test suite may be slow, and/or brittle, and it might not verify everything it implies.

The best way to learn about testing is to jump in and start creating your own spec files. Use examples you've seen in the past, and the guidance of your fellow Power developers as you continue to learn.
