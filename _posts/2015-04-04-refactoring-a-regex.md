---
layout: post
title: 'Refactoring a Regex'
categories: []
tags:
- regex
- regular expression
- refactoring
status: publish
type: post
published: true
author: Philip Szalwinski
---
Regular expressions are a powerful way to search through and manipulate text.
But as we know from *Spiderman*, with great power comes greatly reduced readability.
In my current side project, I'm using regexes as part of a dynamic comment generator for teachers to provide meaningful feedback to students.
The intent of this tool is to reuse comments that a teacher has already written and apply them mad-libs style to similarly performing students.
With the goal of writing extendable, readable code that adheres to the SOLID principles, let's refactor a shamelessly rigid example in which we replace all pronouns with their grammatical form so that they can later be re-substituted with male or female pronouns.

```ruby
  text.gsub!(/(\bhis\b|\bher\b|\bhe\b|\bshe\b|\bhim\b)/i,
            "his" => ":possessive_pronoun:",
            "His" => ":possessive_pronoun:",
            "her" => ":possessive_pronoun:",
            "Her" => ":possessive_pronoun:",
            "he"  => ":personal_pronoun:",
            "He"  => ":personal_pronoun:",
            "she" => ":personal_pronoun:",
            "She" => ":personal_pronoun:",
            "him" => ":dative_personal_pronoun:",
            "Him" => ":dative_personal_pronoun:",
           )
```

The `gsub!` method takes in a regular expression as its first argument.
The second argument could be a single word so that any time a match is found a single substitution could be made.
But here we want to conditionally substitute.
For that, the hash of matches to their substitutes is provided as the second argument.
For example, when `gsub!` finds "his" it looks to the value stored under the "his" key, which is `":possessive_pronoun:"`.
The first thing to do here is to give this method a good name. How about `substitute_pronouns`.

```ruby
  def substitute_pronouns(text)
    text.gsub!(/(\bhis\b|\bher\b|\bhe\b|\bshe\b|\bhim\b)/i,
              "his" => ":possessive_pronoun:",
              "His" => ":possessive_pronoun:",
              "her" => ":possessive_pronoun:",
              "Her" => ":possessive_pronoun:",
              "he"  => ":personal_pronoun:",
              "He"  => ":personal_pronoun:",
              "she" => ":personal_pronoun:",
              "She" => ":personal_pronoun:",
              "him" => ":dative_personal_pronoun:",
              "Him" => ":dative_personal_pronoun:",
             )
  end
```

Next, the regex could use a name.
It's looking for pronouns, so that sounds like a reasonable name.

```ruby
  def substitute_pronouns(text)
    pronouns = /(\bhis\b|\bher\b|\bhe\b|\bshe\b|\bhim\b)/i
    text.gsub!(pronouns,
              "his" => ":possessive_pronoun:",
              "His" => ":possessive_pronoun:",
              "her" => ":possessive_pronoun:",
              "Her" => ":possessive_pronoun:",
              "he"  => ":personal_pronoun:",
              "He"  => ":personal_pronoun:",
              "she" => ":personal_pronoun:",
              "She" => ":personal_pronoun:",
              "him" => ":dative_personal_pronoun:",
              "Him" => ":dative_personal_pronoun:",
             )
  end
```

Let's make this a bit easier to digest by extracting the substitution mappings into a separate method.

```ruby
  def substitute_pronouns(text)
    pronouns = /(\bhis\b|\bher\b|\bhe\b|\bshe\b|\bhim\b)/i
    text.gsub!(pronouns, substitutions)
  end

  def substitutions
    {
      "his" => ":possessive_pronoun:",
      "His" => ":possessive_pronoun:",
      "her" => ":possessive_pronoun:",
      "Her" => ":possessive_pronoun:",
      "he"  => ":personal_pronoun:",
      "He"  => ":personal_pronoun:",
      "she" => ":personal_pronoun:",
      "She" => ":personal_pronoun:",
      "him" => ":dative_personal_pronoun:",
      "Him" => ":dative_personal_pronoun:",
    }
  end
```

This is getting easier to read!
But now the duplication is starting to stand out.
Each of the keys is something we're looking for in the text.
We can use the keys to dynamically generate our regular expression.
In addition, we can remove the `i` flag for case insensitivity now that we're explicitly searching for both "Her" and "her".

```ruby
  def keys_as_regexes
    substitutions.keys.map { |pronoun| /\b#{pronoun}\b/ }
  end

  def substitutions
    {
      "his" => ":possessive_pronoun:",
      "His" => ":possessive_pronoun:",
      "her" => ":possessive_pronoun:",
      "Her" => ":possessive_pronoun:",
      "he"  => ":personal_pronoun:",
      "He"  => ":personal_pronoun:",
      "she" => ":personal_pronoun:",
      "She" => ":personal_pronoun:",
      "him" => ":dative_personal_pronoun:",
      "Him" => ":dative_personal_pronoun:",
    }
  end
```

Now that the keys have each been converted to a regular expression, we're ready to complete this refactor by joining them together into one regex.
To do this, we use `Regexp.union`.

```ruby
  def substitute_pronouns(text)
    text.gsub!(Regexp.union(keys_as_regexes), substitutions)
  end

  def keys_as_regexes
    substitutions.keys.map { |pronoun| /\b#{pronoun}\b/ }
  end

  def substitutions
    {
      "his" => ":possessive_pronoun:",
      "His" => ":possessive_pronoun:",
      "her" => ":possessive_pronoun:",
      "Her" => ":possessive_pronoun:",
      "he"  => ":personal_pronoun:",
      "He"  => ":personal_pronoun:",
      "she" => ":personal_pronoun:",
      "She" => ":personal_pronoun:",
      "him" => ":dative_personal_pronoun:",
      "Him" => ":dative_personal_pronoun:",
    }
  end
```
Almost there! We're very close to complying with the [Open Closed Principle](http://en.wikipedia.org/wiki/Open/closed_principle): the `substitutions` are open to extension, but closed to modification.
By optionally requiring substitutions to be passed in, the method that calls `substitute_pronouns` can extend the substitutions without needing to modify the `substitutions` method that contains pronoun mappings.

```ruby
  def substitute_pronouns(text, substitutions = substitutions)
    text.gsub!(Regexp.union(keys_as_regexes(substitutions)), substitutions)
  end

  def keys_as_regexes(substitutions = substitutions)
    substitutions.keys.map { |word| /\b#{word}\b/ }
  end

  def substitutions
    {
      "his" => ":possessive_pronoun:",
      "His" => ":possessive_pronoun:",
      "her" => ":possessive_pronoun:",
      "Her" => ":possessive_pronoun:",
      "he"  => ":personal_pronoun:",
      "He"  => ":personal_pronoun:",
      "she" => ":personal_pronoun:",
      "She" => ":personal_pronoun:",
      "him" => ":dative_personal_pronoun:",
      "Him" => ":dative_personal_pronoun:",
    }
  end
```

For instance, if we want to replace `"Sally"` with `":first_name:"` we can call `substitute_pronouns(text, substitutions.merge!({"Sally" => ":first_name:"})`.

Not only does this refactoring make the code easier to read and understand, but it also allows the code to be more easily extended.
Do yourself and the others who have to work with your code a favor: write shameless code, then shamelessly refactor!
