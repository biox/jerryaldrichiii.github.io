---
layout: post
subject: consulting
title: Sample Syntax Highlighting Document
summary: Testing syntax highlighters can be difficult without a sample document. Here is one in Ruby.
---

When selecting a syntax highlighting theme generally all that is presented to choose from is a short sample block. Most of the time it is a simple function that prints `Hello world` and if you're lucky has a comment. Unfortunately, making an informed decision with such little information is difficult.

The solution here would be to write a sample document that contains examples of what you would like to see highlighted. However, the task of writing a lengthy sample document can be quite daunting. The next best solution would be to search for prefabricated documents. When I started down that path I quickly learned that crafting a search query to elicit the results I wanted was difficult.

Eventually, I found [this](https://github.com/jneen/rouge/tree/master/spec/visual/samples) collection of documents. Below is and edited version of the Ruby sample:

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

"instance variables can be #@included, #@@class_variables\n and #$globals as well."
instance variables can be #@included, #@@class_variables\n and #$globals as well.
'instance variables can be #@included, #@@class_variables\n and #$globals as well.'
/instance variables can be #@included, #@@class_variables\n and #$globals as well./mousenix
:"instance variables can be #@included, #@@class_variables\n and #$globals as well."
:'instance variables can be #@included, #@@class_variables\n and #$globals as well.'
%'instance variables can be #@included, #@@class_variables\n and #$globals as well.'
%q'instance variables can be #@included, #@@class_variables\n and #$globals as well.'
%Q'instance variables can be #@included, #@@class_variables\n and #$globals as well.'
%w'instance variables can be #@included, #@@class_variables\n and #$globals as well.'
%W'instance variables can be #@included, #@@class_variables\n and #$globals as well.'
%s'instance variables can be #@included, #@@class_variables\n and #$globals as well.'
%r'instance variables can be #@included, #@@class_variables\n and #$globals as well.'
%x'instance variables can be #@included, #@@class_variables\n and #$globals as well.'

#%W[ but #@0illegal_values look strange.]

%s#ruby allows strange#{constructs}
%s#ruby allows strange#$constructs
%s#ruby allows strange#@@constructs

%( hash mark: # )
%r( hash mark: # )
%w( ! $ % # )

##################################################################
# HEREDOCS
<<-CONTENT.strip_heredoc
  D
  E
CONTENT

foo(<<-A, <<-B)
this is the text of a
A
and this is the text of b
B

<<~END
This is a Ruby 2.3 stripped heredoc
END

a = <<"EOF"
This is a multiline #$here document
terminated by EOF on a line by itself
EOF

a = <<'EOF'
This is a multiline #$here document
terminated by EOF on a line by itself
EOF

b=(p[x] %32)/16<1 ? 0 : 1

<<""
#{test}
#@bla
#die suppe!!!
\xfffff


super <<-EOE % [
    foo
EOE

<<X
X
X "foo" # This is a method call, heredoc ends at the prev line

%s(uninter\)pre\ted)            # comment here
%q(uninter\)pre\ted)            # comment here
%Q(inter\)pre\ted)              # comment here
:"inter\)pre\ted"               # comment here
:'uninter\'pre\ted'             # comment here

%q[haha! [nesting [rocks] ! ] ] # commeht here


##################################################################
class                                                  NP
def  initialize a=@p=[], b=@b=[];                      end
def +@;@b<<1;b2c end;def-@;@b<<0;b2c                   end
def  b2c;if @b.size==8;c=0;@b.each{|b|c<<=1;c|=b};send(
     'lave'.reverse,(@p.join))if c==0;@p<< c.chr;@b=[] end
     self end end ; begin _ = NP.new                   end


# Regexes
/
this is a
mutliline
regex
/

this /is a
multiline regex too/

also /4
is one/

this(/
too
/)

# this not
2 /4
asfsadf/

foo[:bar] / baz[:zot]


#from: http://coderay.rubychan.de/rays/show/383
class Object
  alias  :xeq :
  def (cmd, p2)
    self.method(cmd.to_sym).call(p2)
  end
end
p [1,2,3].('concat', [4,5,6]) # => [1, 2, 3, 4, 5, 6]
p [1,2,3].(:concat, [4,5,6]) # => [1, 2, 3, 4, 5, 6]
p "Hurra! ".(:*, 3) # => "Hurra! Hurra! Hurra! "
p "Hurra! ".('*', 3) # => "Hurra! Hurra! Hurra! "
# Leider geht nicht die Wunschform
# [1,2,3] concat [4,5,6]

class Object
  @@infixops = []
  alias :xeq :
  def addinfix(operator)
    @@infixops << operator
  end
  def (expression)
    @@infixops.each{|op|break if expression.match(/^(.*?) (#{op}) (.*)$/)}
    raise "unknown infix operator in expression: #{expression}" if $2 == nil
    eval($1).method($2.to_sym).call(eval($3))
  end
end
addinfix("concat")
p [1,2,3] concat [4,5,6] # => [1, 2, 3, 4, 5, 6]


# HEREDOC FUN!!!!!!!1111
foo(<<A, <<-B, <<C)
this is the text of a
   A!!!!
A
and this is text of B!!!!!!111
   B
and here some C
C
###################################

######
# testing the __END__ content and % as an operator
######

events = Hash.new { |h, k| h[k] = [] }
DATA.read.split(/\n\n\n\s*/).each do |event|
  name = event[/^.*/].sub(/http:.*/, '')
  event[/\n.*/m].scan(/^([A-Z]{2}\S*)\s*(\S*)\s*(\S*)(\s*\S*)/) do |kind, day, daytime, comment|
    events[ [day, daytime] ] << [kind, name + comment]
  end
end

foo = 3
foo %= 2
foo /= 10 # not a regex

conflicts = 0
events.to_a.sort_by do |(day, daytime),|
  [%w(Mo Di Mi Do Fr).index(day) || 0, daytime]
end.each do |(day, daytime), names|
  if names.size > 1
    conflicts += 1
    print '!!! '
  end
  print "#{day} #{daytime}: "
  names.each { |kind, name| puts "  #{kind}  #{name}" }
  puts
end

puts '%d conflicts' % conflicts
puts '%d SWS' % (events.inject(0) { |sum, ((day, daytime),)| sum + (daytime[/\d+$/].to_i - daytime[/^\d+/].to_i) })

string = % foo     # strange. huh?
print "Escape here: \n"
print 'Dont escape here: \n'

__END__
```
