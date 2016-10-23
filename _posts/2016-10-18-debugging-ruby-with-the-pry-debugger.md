---
layout: post
title: Debugging Ruby with the Pry debugger 
category: ruby
tags: [ruby, pry, debugging, code]
summary: What is Pry? How does it help me debug my Ruby? 
---

## What is Pry?
Pry is an interactive shell that provides an approachable interface to debugging Ruby. Most notably it allows for the ability to transverse a running Ruby program much like one would a filesystem.

## Using Pry

Pry can be installed using `gem install pry` and be ran directly from your shell using `pry`.

By adding `require pry` to your Ruby code you can use Pry in your current project. Just requiring Pry will not start a session during execution however. For that, you must also include a breakpoint by adding `binding.pry` to your code.

> NOTE: I prefer to invoke Pry with a breakpoint on one line (`require 'pry'; binding.pry`).
> This prevents me from forgetting to remove the `require 'pry'` line from the top of the file and accidentally submitting a PR.

## Navigating the Shell
Once inside a Pry shell you can get a list of available commands by running `help`.

From that output you can see there are a plethora of executable commands. Don't be discouraged though, I made it for embarrassingly too long using only `ls`, `step`, `puts my_variable_name`, and `exit-program`

## Simple Usage Guide
One thing I wish I had had when starting to use Pry was a "Simple Usage Guide". Below is just such a thing.

### Create a Sample File
Create a file called `pry_example.rb` with the following contents

```ruby
def output_hash(input_hash)
 puts input_hash
end

def set_baz(input_hash, input_value)
  input_hash[:baz] = input_value
end

require 'pry'; binding.pry

some_hash = {}
some_hash[:foo] = 'bar' 

output_hash(some_hash)

set_baz(some_hash, 'spam')

puts "Don't execute me"
```

### Starting Pry
Run `ruby pry_example.rb`

```
[~/Playground/Blog]$ ruby pry_example.rb

Frame number: 0/1

From: /home/jerry/Playground/Blog/pry_example.rb @ line 11 :

     6:   input_hash[:baz] = input_value
     7: end
     8:
     9: require 'pry'; binding.pry
    10:
 => 11: some_hash = {}
    12: some_hash[:foo] = 'bar'
    13:
    14: output_hash(some_hash)
    15:
    16: set_baz(some_hash, 'spam')

[1] pry(main)>
```

As you can see we have halted execution and are at line 11 of our example program

### Using `step` 
To execute the current line and proceed to the next type `step`

```
[1] pry(main)> step

From: /home/jerry/Playground/Blog/pry_example.rb @ line 12 :

     7: end
     8:
     9: require 'pry'; binding.pry
    10:
    11: some_hash = {}
 => 12: some_hash[:foo] = 'bar'
    13:
    14: output_hash(some_hash)
    15:
    16: set_baz(some_hash, 'spam')
    17:

[1] pry(main):1>
```

### Viewing Objects
To verify that line 11 was executed we can inspect the object by typing `some_hash`

```
[1] pry(main):1> some_hash
{}
[2] pry(main):1>
```

We can see here that `some_hash` is an empty array.

Run `step` again to execute line 12 and set `some_hash[:foo]` to `bar`

```
[2] pry(main):1> step

From: /home/jerry/Playground/Blog/pry_example.rb @ line 14 :

     9: require 'pry'; binding.pry
    10:
    11: some_hash = {}
    12: some_hash[:foo] = 'bar'
    13:
 => 14: output_hash(some_hash)
    15:
    16: set_baz(some_hash, 'spam')
    17:
    18: puts "Don't execute me"

[2] pry(main)>
```

Now we can see that `some_hash` contains the symbol `:foo` which has the value of `bar`

```
[2] pry(main)> some_hash
{:foo=>"bar"}
[3] pry(main)>
```

### Using `whereami`
At any time during our execution we can show lines surrounding us with the command `whereami`

```
[3] pry(main)> whereami

From: /home/jerry/Playground/Blog/pry_example.rb @ line 14 :

     9: require 'pry'; binding.pry
    10:
    11: some_hash = {}
    12: some_hash[:foo] = 'bar'
    13:
 => 14: output_hash(some_hash)
    15:
    16: set_baz(some_hash, 'spam')
    17:
    18: puts "Don't execute me"

[4] pry(main)>
```

After doing this we can see that the method `output_hash` is about to be executed

### Investigating Methods
We can view the `output_hash` method with `show-method output_hash`

```
[4] pry(main)> show-method output_hash

From: pry_example.rb @ line 1:
Owner: Object
Visibility: private
Number of lines: 3

def output_hash(input_hash)
 puts input_hash
end
[5] pry(main)>
```

After viewing the method we can step into it using `step`

```
[5] pry(main)> step

From: /home/jerry/Playground/Blog/pry_example.rb @ line 2 Object#output_hash:

    1: def output_hash(input_hash)
 => 2:  puts input_hash
    3: end

[5] pry(main)>
```

We can now see that we are inside the `output_hash` method

> Technically, we now know that `output_hash` is a procedure and not a method, but such semantics are out of scope for this post. 

We can view local variables in `output_hash` by using `ls`

```
[5] pry(main)> ls
self.methods: inspect  to_s
locals: _  __  _dir_  _ex_  _file_  _in_  _out_  _pry_  input_hash
[6] pry(main)>
```

We can inspect any of these variables with `ls` as well

Inspect `input_hash` by running `ls input_hash`

```
[6] pry(main)> ls input_hash
Enumerable#methods:
  all?   collect         cycle   drop_while  each_slice       redacted 
  any?   collect_concat  detect  each_cons   each_with_index  redacted
  chunk  count           drop    each_entry  each_with_object redacted
Hash#methods:
  ==     clear                 default=       delete_if  redacted
  []     compare_by_identity   default_proc   each       redacted
  []=    compare_by_identity?  default_proc=  each_key   redacted
  assoc  default               delete         each_pair  redacted
[7] pry(main)>
```

> NOTE: Output here is redacted to shorten line length

As you can see `input_hash` is indeed a Hash

Also listed are the available methods for both the Hash and Enumerable classes

Run `step` to execute the `puts input_hash` line and return to our main code

```
[7] pry(main)> step
{:foo=>"bar"}

From: /home/jerry/Playground/Blog/pry_example.rb @ line 16 :

    11: some_hash = {}
    12: some_hash[:foo] = 'bar'
    13:
    14: output_hash(some_hash)
    15:
 => 16: set_baz(some_hash, 'spam')
    17:
    18: puts "Don't execute me"

[7] pry(main)>
```

As you can see the output of `output_hash` is sent to screen (`{:foo=>"bar"}`)

### Modifying Code During Runtime
Run `step` to go into the `set_baz` method

```
[7] pry(main)> step

From: /home/jerry/Playground/Blog/pry_example.rb @ line 6 Object#set_baz:

    5: def set_baz(input_hash, input_value)
 => 6:   input_hash[:baz] = input_value
    7: end

[7] pry(main)>
```

Since we are in a running Ruby session we can modify code during runtime

Let's change the value of `input_value` to `eggs` by running `input_value = 'eggs'`

```
[7] pry(main)> input_value = 'eggs'
=> "eggs"
[8] pry(main)>
```

Now execute line 6 with `step`

```
[8] pry(main)> step

From: /home/jerry/Playground/Blog/pry_example.rb @ line 18 :

    13:
    14: output_hash(some_hash)
    15:
    16: set_baz(some_hash, 'spam')
    17:
 => 18: puts "Don't execute me"

[8] pry(main)>
```

Now we can see `set_baz` set `some_hash[:bar]` to `eggs` by running `puts some_hash;`

```
[8] pry(main)> puts some_hash;
{:foo=>"bar", :baz=>"eggs"}
[9] pry(main)>
```

> TIP: Adding `;` to the end of Ruby commands will suppress extra shell output

### Exiting Pry
After running `whereami` we see that we do not want to execute line 18

So run `exit-program` to end our Pry session

```
[9] pry(main)> whereami

From: /home/jerry/Playground/Blog/pry_example.rb @ line 18 :

    13:
    14: output_hash(some_hash)
    15:
    16: set_baz(some_hash, 'spam')
    17:
 => 18: puts "Don't execute me"

[10] pry(main)> exit-program
[~/Playground/Blog]$
```

## Conclusion
As you can see, with just a few simple commands we can do some valuable debugging of our code. I personally use Pry daily in my current job (*shakes fist at Ruby's `nil` errors*)

While this post just scratches the surface of Pry, I highly recommend that you dig deeper. Learning more about Pry will significantly reduce headaches and increase efficiency.

## Extra Resources
Below are some extra resources that I've found helpful for learning Pry

  - [Pry GitHub Page](https://github.com/pry/pry)
  - [Pry Wiki](https://github.com/pry/pry/wiki)
  - [YouTube Talk by Luke Bergen](https://www.youtube.com/watch?v=o90CCPjcIKE)
