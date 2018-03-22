# perl-stuff
This is currently where I've decided to dump my notes and example code related to perl - because let's face it, unless you work at Google or <insert hot new tech startup you probably won't be working at a year from now either because a) they weren't actually in the business of writing new libraries for the exotic language they chose to develop on, or b) you've simply moved on to a place that's been around awhile, and by reasonable extension will most certainly have a large amount of (at the very, very least) legacy perl - where they oddly haven't needed to write their own dmidecode library.>




As a start, the following is some useful general info excerpted from <ins>Effective Perl Programming: Ways to Write Better, More Idiomatic Perl</ins>, Second Edition
by Brian D. Foy; Joseph N. Hall; Joshua A. McAdams
Published by Addison-Wesley Professional, 2010 obtained from https://www.safaribooksonline.com and reproduced in accordance with the [Terms of Service](https://www.safaribooksonline.com/membership-agreement/).

## Regular expressions are made up of atoms and operators. Atoms are generally single-character matches. For example:

```
a      # matches the letter a
\$     # matches the character
\n     # matches newline
[a-z]  # matches a lowercase letter
.      # matches any character except \n
\1     # arbitrary length-backreference to 1st capture
```

There are also special “zero-width” atoms. For example:

```
\b     # word boundary--transition from \w to \W
^      # matches start of a string
\A     # absolute beginning of string
\Z     # end of a string or newline at end --
       # this may or may not be zero-width
\z     # absolute end of string with nothing after it
```

Atoms are modified or joined together by regular-expression operators. As in arithmetic expressions, there is an order of precedence among these operators.


## Regular expression precedence

Fortunately, there are only four precedence levels. Imagine if there were as many as there are for arithmetic expressions!

Parentheses and the other grouping operators have the highest precedence. The below table shows the precedence order of regular-expression operators.

**Regular Expression Operator Precedence, from Highest to Lowest**

| Precedence | Operators | Description |
| :---       |     :---: |        ---: |
| Highest   | () (?:) etc.     | Parentheses and other grouping  |
|      | ? + * {m,n} +? ++ etc.    | Repetition     |
|      | ^ $ abc \G \b \B [abc]    | Sequence, literal characters, character classes, assertions |
| Lowest | a\|b  | Alternation  |

## Use hashes to pass named parameters

Although Perl provides no method of automatically naming parameters in the function to which you pass them (in other words, no “formal parameters”, i.e. not named, typed parameters like in most other languages...), there are a variety of ways that you can call functions with an argument list that provides both names and values. All of these mechanisms require that the function you call do some extra work while processing the argument list. In other words, this feature isn’t built into Perl either, but it’s a blessing in disguise. Different implementations of named parameters are appropriate at different times. Perl makes it easy to write and use almost any implementation you want. A simple approach to named parameters constructs a hash out of the argument list:

```
sub uses_named_params {
  my %param = (
    foo => 'val1'
    bar => 'val2',
  );
  my %input = @_; # read in args as a hash
  
  # combine params read in with defaults
  @param{ keys %input } = values %input;
  
  # now, use $param{foo}, $param{bar}, etc.
  ...
}
```

You would call `uses_named_params` with key-value pairs just as if you were constructing a hash: `uses_named_params( bar => 'myval1', bletch => 'myval2' );` That wasn’t very many lines of code, was it? And they were all fairly simple. This is a natural application for hashes.You may want to allow people to call a subroutine with either positional parameters or named parameters. The simplest thing to do in this case is to prefix parameter names with minus signs. Check the first argument to see if it begins with a minus. If it does, process the arguments as named parameters. Here’s one straightforward approach:

```
sub uses_minus_params {
  my %param = ( -foo => 'val1', -bar => 'val2' );
  my %input;
  
  if ( substr( $_[0], 0, 1 ) eq '-' ) {
    # read in named params as a hash
    %input = @_;
  }
  else {
    my @name = qw(-foo -bar);
    # give positional params names and save in a hash
    %input = map { $name[$_], $_[$_] } 0 .. $#_;
  }
  
    # overlay params on defaults
    @param{ keys %input } = values %input;
    
    # use $param{-foo}, $param{-bar}
}
```

You can call this subroutine with either named or positional parameters (although it’s better to choose one method and stick with it):

```
uses_minus_params( -foo => 'myval1', -xtra => 'myval2' );
uses_minus_params( 'myval1, 'myval2' );
```

Stay away from single character parameter names --for example, `-e` and `-x`. In addition to being overly terse, those are file test operators.

If you use this method for processing named parameters, you refer to the arguments inside your subroutine by using a hash whose keys are prefixed with minus signs (e.g., `$param{-foo}`, `$param{-bar}`). Using identifiers preceded by minus signs as arguments or keys may look a little funny to you at first (“Is that really Perl?”), but Perl actually treats barewords preceded by minus signs as though they were strings beginning with minus signs. This is generally convenient, but this approach does have a couple of drawbacks. First, although an identifier with a leading minus sign gets a little special treatment from Perl, the identifier isn’t forcibly treated as a string, as it would be to the left of `=>` or alone inside braces. Thus, you have to quote a parameter like `-print`, lest it turn into `-1` (while also printing the value of `$_`). Second, if you want to use the positional argument style and need to pass a negative first argument, you have to supply it as a string with leading whitespace or do something else equally ungainly.
   
    
### Things to remember
* Use hashes to name subroutine arguments.
* Set default values for parameters by merging hashes.
* Choose either positional or named parameters, and stick with your choice.


## Use prototypes to get special argument parsing

The below table shows the characters you can use in a prototype.

| Prototype characters | Meaning |
| :---                 | :---    |
| \$, \@, \%, \&, \*   | Returns reference to variable name or argument |
|    $   | Forces scalar context    |
| @, %   | Gobbles the rest of the arguments; forces list context |
| & | Coderef; sub keyword optional if first argument |
| * | Typeglob |
| ; | Separate mandatory from optional arguments |


### Multiple array arguments

How about a subroutine that takes two array arguments and “blends” them into a single list? Take one element from the first array, then one from the second, then another from the first, and so on:

```
sub blend (\@\@) {
  local ( *a, *b ) = @_; # faster than lots of derefs
  my $n = $#a > $#b ? $#a : $#b;
  my @res;
  for my $i ( 0 .. $n ) {
    push @res, $a[$i], $b[$i];
  }
  
  # could have written this:
  # map { $a[$_], $b[$_] } 0..$n;
  # but for and push end-up being faster
  @res;
}

# sample usage
blend @a, @b;
blend @{ [ 1 .. 10 ] }, @{ [ 11 .. 20 ] };
```

Along the same lines, you can write a subroutine that iterates through the elements of a list like `foreach`, but *n* at a time:

```
# for_n: iterate over a list n elements at a time
sub for_n (&$@) {
  my ( $sub, $n, @list ) = @_;
  my $i;
  while ( $i < $#list ) {
    &$sub( @list[ $i .. ( $i + $n -1 ) ] );
    $i += $n;
  }
}

# sample usage
@a = 1 .. 10;
for_n { print "$_[0], $_[1]\n" } 2, @a;
```

Be careful when using atoms like `\@` and `\%` in code that you are going to share with the world, since other programmers may not expect subroutines to take arguments by reference without an explicit backslash. Document such behavior thoroughly. By the way, you don’t need your own `blend` or `for_n`, since `List::MoreUtils`’ `mesh` and `natatime` can do those for you :)

### Things to remember
* Use prototypes to create your own array or hash operators.
* Use prototypes to create subroutines that take separate arrays as arguments.
* Avoid overusing prototypes, especially when they would confuse people.

## Create closures to lock in data

In Perl, **closures** are subroutines that refer to lexical variables that have gone out of scope. The data does not disappear, because the subroutines still have references to them. You can use closures to limit data to a named subroutine or new anonymous subroutines. 

### Private data for named subroutines

Sometimes your subroutines need some data that only they can see. As with any data, you want to limit their visibility to the smallest scope that you can compose. You could just put your data directly inside the subroutine:

```
sub some_sub {
  my $application_root = '/path/to/my/app';
  
  # do stuff with $application_root
}
```

If you do that, Perl has to recreate the scalar every time that you call the subroutine. If you don’t need to change the data, that’s a waste. Maybe it doesn’t affect performance that much, but it’s just philosophically ugly and needless. You can define `$application_root` outside of the subroutine, but you still want to limit its scope. You do that by wrapping a block around the definition of `$application_root` and the subroutine. You need to define `$application_root` before you define the subroutine so the subroutine can refer to it, so you need to wrap it in a `BEGIN` block:

```
BEGIN {
  my $application_root = '/path/to/my/app';
  
  sub some_sub {
    ...;
  }
}
```

In Perl 5.10 and later, you can get the same thing with a `state` variable. This is such a common pattern that it’s now a feature. The first time you run the subroutine, Perl defines the `state` variable and assigns its value. On subsequent calls, Perl ignores that line and the variable keeps the value it had from the previous run of the subroutine:

```
use 5.010;

sub some_sub {
  state $application_root = '/path/to/my/app';
  
  # do stuff with $application_root
}
```
The `state` variable is more useful for maintaining a variable’s value between calls to the subroutine:

```
use 5.010;

sub show_letter {
  state $letter = 'a';
  
  print "Letter is ",  $letter++, "\n";
}

foreach ( 0 .. 5 ) {
  show_letter();
}
```
The output shows the progression of `$letter`:

```
Letter is a
Letter is b
Letter is c
Letter is d
Letter is e
Letter is f
```

### Private data for subroutine references

Anonymous closures are almost the same thing as using `state` variables, but they can be much more useful because you can create as many closures as you like and you can set up each subroutine just the way you need it. If you wanted to make an anonymous closure doing the same as the previous example, you do mostly the same thing although it all happens at run time:

```
my $session = do {
  my $application_root = '/path/to/my/app';
  
  sub {
    ...;
  }
};
```

More useful, however, is something that creates the closure on demand.  Even though you have an anonymous subroutine, you still did all of the same work you did to set up the named subroutine. That’s not very flexible. Instead, you can use a **factory** that makes subroutines:

```
my $session = closure_factory('/path/to/my/app');

sub closure_factory {
  my $application_root = shift;
  
  sub {
    ...;
  }
}
```

You can create as many of these as you like. Consider a set of independent counters that you create from the same factory subroutine (a fancy name for a subroutine that creates other subroutines):

```
sub make_cycle {
  my ( $min, $max ) = @_;
  
  my @numbers= $min .. $max;
  my $cursor = 0;
  
  sub { $numbers[ $cursor++ % @numbers ] }
}

my $cycle_5_10 = make_cycle( 5, 9 );
my $cycle_f_m = make_cycle( 'f', 'm');
```

When you call one of your closures, it doesn’t affect any of the other closures that you created from the same factory:

```
foreach ( 0 .. 10 ) {
  print $cycle_5_10->(), $cycle_f_m->();
}
```

The output shows the closures operating independently as they interlace their output:
```
5f6g7h8i9j5k6l7m8f9g5h
```

### Closures can share data

You don’t have to limit your out-of-scope data to a single closure. As long as you create the subroutines while those data are in scope, they will share the references. Consider the `File::Find::Closures` module, which supplies convenience subroutines to work with `File::Find`. The `find` subroutine expects a reference to a subroutine to do its magic:

```
use File::Find qw(find);
use File::Find::Closures qw(find_by_regex);

my ( $wanted, $reporter ) = find_by_regex(qr/*.pl/);

find( $wanted, @search_dirs );

my @files = $reporter->();
```

The `find_by_regex` handles two important details for you, each handled by its own closure. First, it creates the callback function that `find` needs. In the same scope, it defines the `@files` array to store the list of files that it collects. To access that array, it creates a second closure:

```
# From File::Find::Closures
sub find_by_regex {
  require File::Spec::Functions;
  require Carp;
  require UNIVERSAL;
  
  my $regex = shift;
  
  unless ( UNIVERSAL::isa( $regex, ref qr// ) ) {
    Carp::croak "Argument must be a regular expression";
  }
  
  my @files = ();
  
  sub {
    push @files,
      File::Spec::Functions::canonpath($File::Find::name)
      if m/$regex/;
   }, sub { wantarray ? @files : [@files] }
 }
 ```
 
 ### Things to remember
 * Use lexical variables to make data private to subroutines.
 * In Perl 5.10 or later, use `state` variables for private data.
 * Make generator (“factory”) subroutines that create new subroutines for you.
 
 
# Links:
* O'reilly Perl Best Practices: http://archive.oreilly.com/pub/a/perl/excerpts/perl-best-practices/appendix-b.html#perlbp-APP-B-SECT-11
* PerlMonks thread on parallel execution of system commmands: http://www.perlmonks.org/?node_id=791996
* Really excellent summary of Perl regex, file IO & text processing: https://www.ntu.edu.sg/home/ehchua/programming/webprogramming/Perl2_Regexe.html
* Nice blog post featuring some Perl idioms to make life easier: https://www.xaprb.com/blog/2006/10/05/five-great-perl-programming-techniques-to-make-your-life-fun-again/
* Great site with some very straightforward and uncomplicated discussion on a number of topics related not only to Perl but programming in general: https://www.caveofprogramming.com/perl-tutorial/perl-stdin-reading-user-input-in-perl.html
* Perl and Docker: https://robn.io/docker-perl/
