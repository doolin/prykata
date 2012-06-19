<h1 style="text-align: center">Dancing with Rails</h1>


<img src="http://pry.github.com/images/pry_logo.png">

<h3 style="text-align: center">An adventure in single stepping starring Pry</h3> 

# Before we get started...

### Who is coding Rails every day?

### Who is using Pry every day?

### Who uses Ruby debugger regularly?



# Purpose of presentation

### The ulterior motives...

1. Inspire interest in Pry and friends
1. Acquire audience feedback on future direction of this talk

## Feedback!

Stuff to think about during talk:

* Debugging and debuggers
* Ruby internals
* REPL

We'll revisit these notions at the end.

# Some back story

This presentation was intended to be a "code kata" for personal use.

Then, it turned into a Pry demo.

Now, it's still under development and (sort of) aimed at working with Rails apps.

[Get the code on github](https://github.com/doolin/prykata),
or follow along there.

### I am open to pull requests


# Why Pry?

The Ruby ecosystem is weak with respect to major
feature which is present in the same
class of languages: a full-featured REPL.

**REPL means `read-eval-print-loop`.**

When you fire up Scheme, Lisp or Smalltalk consoles, what you get is the
REPL.

Pry is a next step in implementing a powerful REPL for Ruby
development.

## Two primary uses for Pry

* A *better* Irb
* An alternative debugger

### No more "puts-driven development"

# But Irb is a REPL...?

Actually, yes, technically Irb is a REPL.

And it's not bad, especially when attached to large applications like
Rails.

Pry makes Irb a lot better.

>Pry, in some sense, is IRB turned on its head. Instead of having to
>bring your code to a REPL session (as with IRB) you instead bring a
>REPL session to your code... [John
>Mair](http://banisterfiend.wordpress.com/2011/01/27/turning-irb-on-its-head-with-pry/).


# REPL read-eval-print-loop

~~~~
@@@ scheme
$ sbcl 
This is SBCL 1.0.55, an implementation of ANSI Common Lisp.
* (defun foo ( * 3 4 ))
; in: DEFUN FOO
;     (SB-INT:NAMED-LAMBDA FOO
;         ( * 3 4)
;       (BLOCK FOO))
; ==>
;   #'(SB-INT:NAMED-LAMBDA FOO
;         ( * 3 4)
;       (BLOCK FOO))
; 
; caught ERROR:
;   Required argument is not a symbol: 3
; 
; compilation unit finished
;   caught 1 ERROR condition

debugger invoked on a SB-INT:COMPILED-PROGRAM-ERROR:
  Execution of a form compiled with errors.
Form:
  #'(NAMED-LAMBDA FOO
      ( * 3 4)
    (BLOCK FOO))
Compile-time error:
  Required argument is not a symbol: 3

Type HELP for debugger help, or (SB-EXT:QUIT) to exit from SBCL.

restarts (invokable by number or by possibly-abbreviated name):
  0: [ABORT] Exit debugger, returning to top level.

(#:EVAL-THUNK)
0] (quit)
~~~~


# Try again

~~~~
@@@ lisp
$ sbcl 
This is SBCL 1.0.55, an implementation of ANSI Common Lisp.
* (defun foo () ( * 3 4 ))

FOO
* (foo)

12
* (quit)
~~~~


# Features of Pry

* Source code browsing (including core C source with the pry-doc gem)
* Navigation around state (cd, ls and friends)
* Rubinius core source browing
* Documentation browsing
* Live help system
* Open methods in editors (edit-method Class#method)
* Syntax highlighting
* Command shell integration (start editors, run git, and rake from within Pry)
* Gist integration
* Runtime invocation (use Pry as a developer console or debugger)
* Exotic object support (BasicObject instances, IClasses, ...)
* A powerful and flexible command system
* Ability to view and replay history
* Many convenience commands inspired by IPython, Smalltalk and other
advanced REPLs


### That's a lot of features

(We're not going to look at all of them.)

# The Pry ecosystem

As noted at the beginning, Pry is being used as an alternative to
`ruby-debug`. Since Pry doesn't have explicit support for standard
debugging operations such as `step`, `continue`, etc., we look to
Rubygems and find:

* `pry-nav`
* `pry-doc` 
* `pry-rails` 
* `pry-debugger`
* `pry-stack_explorer`
* `pry-exception_explorer`
  
There are more, this will for now, see the links at the end of this
presentation.

### The Pry ecosystem is expanding rapidly

# Get started - Pry in your Gemfile

~~~~
@@@ruby
source 'http://rubygems.org'
gem 'pry'
gem 'pry-nav'
gem 'pry-doc'
gem 'pry-stack_explorer'
gem 'pry-exception_explorer'
~~~~

# Python documentation coolness

Python has this cool feature in the interpreter which returns a little
documentation about the object and method, when invoked without
parentheses:

~~~~
@@@ python
>>> file.read
<built-in method read of file object at 0x1004c38b0>
>>> file.read()
'This is the contents of the file'
~~~~

## Now with pry-doc

~~~~
@@@ruby
[5] pry(FileUtils):1> show-doc rm
~~~~

# show-doc rm

~~~~
@@@ruby 
[5] pry(FileUtils):1> show-doc rm

From:
/Users/daviddoolin/.rvm/rubies/ruby-1.9.3-p194/lib/ruby/1.9.1/fileutils.rb
@ line 556:
Number of lines: 9
Owner: FileUtils
Visibility: private
Signature: rm(list, options=?)

Options: force noop verbose

Remove file(s) specified in list.  This method cannot remove
directories.
All StandardErrors are ignored when the :force option is set.

  FileUtils.rm %w( junk.txt dust.txt )
  FileUtils.rm Dir.glob(' * .so')
  FileUtils.rm 'NotExistFile', :force => true   # never raises exception

[6] pry(FileUtils):1>
~~~~

# show-method rm

~~~~
@@@ruby
[7] pry(FileUtils):1> show-method rm

From:
/Users/daviddoolin/.rvm/rubies/ruby-1.9.3-p194/lib/ruby/1.9.1/fileutils.rb
@ line 556:
Number of lines: 10
Owner: FileUtils
Visibility: private

def rm(list, options = {})
  fu_check_options options, OPT_TABLE['rm']
  list = fu_list(list)
  fu_output_message "rm#{options[:force] ? ' -f' : ''} #{list.join ' '}"
if options[:verbose]
  return if options[:noop]

  list.each do |path|
    remove_file path, options[:force]
  end
end
[8] pry(FileUtils):1>
~~~~



# Defining a method in Pry

Fire up a terminal window, let's write a Ruby "macro" 

~~~~
@@@ruby
$ pry 
[1] pry(main)> def foo
[1] pry(main)*   'bar'
[1] pry(main)* end  
=> nil
[2] pry(main)> foo
=> "bar"
[3] pry(main)>
~~~~


Method indentation is handled very nicely, better than Irb.

Notice syntax highlighting as well.

### Elegant tools make coding fun

# Lingering in the Kernel...

[Sae
Romy Hong](http://syntacticsugar.github.com/blog/2012/02/23/lingering-in-the-kernel/)
captures the Pry experience well. Here's what she found...


“Navigate into the kernel.”

`[0] pry(main)> cd Kernel`

“Look around you, do you see anything?”

`[1] pry(Kernel)> ls` “Ah, glorious. We have our Kernel methods…”

“Yes. Navigate into String and look around.”

`[2] pry(String)> cd String`

`[3] pry(String)> ls` 

“Yes. Now, try creating an instance of a String.”

`[4] pry(String)> x = "wombat"`

“Navigate into that instance, look around.”

`[5] pry("wombat")> cd x`

`[6] pry("wombat")> ls`

“Tell me, what do you see?”

----

([Visit
Romy](http://syntacticsugar.github.com/blog/2012/02/23/lingering-in-the-kernel/).)


# [6] pry("wombat")> ls

This is what I see:

~~~~
@@@ ruby
Comparable#methods: <  <=  >  >=  between?
String#methods: %  *  +  <<  <=>  ==  ===  =~  []  []=  ascii_only?  bytes  bytesize  byteslice  capitalize  capitalize!  casecmp  center chars  chomp  chomp!  chop  chop!  chr  clear  codepoints  concat  count crypt  delete  delete!  downcase  downcase!  dump  each_byte  each_char each_codepoint  each_line  empty?  encode  encode!  encoding  end_with?  eql?  force_encoding  getbyte  gsub  gsub!  hash  hex  include?  index insert  inspect  intern  length  lines  ljust  lstrip  lstrip!  match next  next!  oct  ord  partition  prepend  replace  reverse  reverse!  rindex  rjust  rpartition  rstrip  rstrip!  scan  setbyte  shellescape shellsplit  size  slice  slice!  split  squeeze  squeeze!  start_with?  strip  strip!  sub  sub!  succ  succ!  sum  swapcase  swapcase!  to_c to_f  to_i  to_r  to_s  to_str  to_sym  tr  tr!  tr_s  tr_s!  unpack upcase  upcase!  upto  valid_encoding?  
self.methods: __binding_impl__
locals: _  _dir_  _ex_  _file_  _in_  _out_  _pry_
[7] pry("wombat"):1> 
~~~~

That's cool.

# pry-nav

`pry-nav` is a must have when using Pry as a debugger. It adds following
commands:

* `continue`
* `step`
* `next`


# Pry open Rails

Now that we've set the stage, there are 2 ways Pry is useful in Rails:

### 1. As a better `rails console`

### 2. As a drop-in for stepping code with pry-nav


# A better rails console

Stick this in your Gemfile (if you haven't already):

~~~~
@@@ruby
group :development do
  ...
  gem 'pry-rails'
  ...
end
~~~~

# binding.pry for great good

Add `binding.pry` anywhere in your Rails execution path, refresh.

You will dumped into the Rails session.

### This can be phenomenally useful


# Feedback!

Recall from start, some questions:

### REPL: Does Pry make sense *to you* as a Ruby REPL?

### Debugging and debuggers

### Ruby internals: instant access, useful or not?

There are no "right" answers to these questions, only answers which make
sense to your work flow.


# Links 
You might find these links useful:

* [Pry Github account](https://github.com/pry/pry)
* [Pry home page](http://pry.github.com/). Make sure to watch and
  rewatch the video on this page. 
* [Pry Railscast](http://railscasts.com/episodes/280-pry-with-rails)
* [Pry Wiki](https://github.com/pry/pry/wiki)
* [Turning Irb on its head with
  Pry](http://banisterfiend.wordpress.com/2011/01/27/turning-irb-on-its-head-with-pry/)
* [The Pry
  ecosystem](http://banisterfiend.wordpress.com/2012/02/14/the-pry-ecosystem/)
* [Debugging with
  Pry](http://yorickpeterse.com/articles/debugging-with-pry)
* [Pry-doc](https://github.com/pry/pry-doc)
* [The pry-debugger](https://github.com/nixme/pry-debugger)

Thanks for reading or listening along.

Feel free to ask questions:
[@doolin](http://twitter.com/doolin).

# That's enough for now.

# Comparison with ruby-debug

`ruby-debug` is pretty useful, provided it's properly employed.

Note: taking the time to really learn how a debugger works often changes
one's opinions of debuggers.


