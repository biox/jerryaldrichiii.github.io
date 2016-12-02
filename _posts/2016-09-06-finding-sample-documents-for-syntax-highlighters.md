---
layout: post
title: Finding sample documents for syntax highlighters
category: other
tags: [syntax, other]
summary: Testing syntax highlighters can be difficult without a sample document. Here is a sample document in Ruby that I found.
---

When selecting a syntax highlighting theme, generally all that is presented to choose from is a short sample block. Most of the time it is merely a simple function that prints `Hello world` and if you're lucky it has a comment. Unfortunately, making an informed decision with such little information is difficult.

One solution would be to write a sample document that contains examples of what you would like to see highlighted. However, the task of writing a lengthy sample document can be quite daunting. The next best solution would be to search for prefabricated documents. When I started down that path I quickly learned that finding such a document is difficult.

Eventually, I found [this](https://github.com/jneen/rouge/tree/master/spec/visual/samples) collection of documents. Below is an edited version of their Ruby sample:

```ruby
#######
# ruby 1.9 examples
#######

state :foo do
  rule %r(/) do
    token Operator
    goto :expr_start
  end

  rule(//) { goto :method_call_spaced }
end

state :foo do
  rule %r(/), Thing
end

%i(this is an array of symbols)
%I(this is too, but with #{@interpolation})
hash = { answer: 42 }
link_to 'new', new_article_path, class: 'btn'

#####
# ternaries
# NB [jneen]: MRI ruby actually has different parsing behavior depending on
# ~what variables are defined~, which we can't know in a highlighting context.
# So... we're going to be wrong here, sometimes. Whatever. These cases look
# okay though, but if they break I don't care a whole lot, because Ruby itself
# doesn't parse them consistently.
a ? b::c : :d
a ? b:c
method_that_takes_a_char ?b
cond??b::c : d

# lol
?????:??
//?//://

a/1 # comment
a / b # comment
a /1 \r[egex]\//
a/ b #comment
x.a / 1 # comment
Foo::a / 3 + 4

########
# method calls
########

foo.bar
foo.bar.baz
foo.bar(123).baz()

foo.
  bar

foo
  .bar()
  .baz

foo.
  bar

foo.
  bar().
  baz

########
# function definitions
########

class (get_foo("blub"))::Foo
  def (foo("bar") + bar("baz")).something argh, aaahaa
    42
  end
end

def -@
  0 - self
end

class get_the_fuck("out")::Of::My
  def parser_definition
    ruby!
  end
end

###############
# General
###############

a.each{|el|anz[el]=anz[el]?anz[el]+1:1}
while x<10000
  b=(p[x]%32)/16<1 ? 0 : 1

  (x-102>=0? n[x-102].to_i : 0)*a+(x-101>=0?n[x-101].to_i : 0)*e+n[x-100].to_i
  n[x+199].to_i*b+n[x+200].to_i*d+n[x+201].to_i*b

g=%w{}
x=0

test //, 123

{staples: /\ASTAPLE?S?\s*[0-9]/i,
 target: "^TARGET "}

while x<100
 puts"#{g[x]}"
 x+=1
end

puts""
sleep(10)

1E1E1
puts 30.send(:/, 5) # prints 6

# fun with class attributes
class Foo
  def self.blub x
    if not x.nil?
      self.new
    end
  end
  def another_way_to_get_class
    self.class
  end
end

# ruby 1.9 "call operator"
a = Proc.new { 42 }
a.()
```
