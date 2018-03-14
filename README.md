# perl-stuff
This is currently where I've decided to dump miscelaneous notes and example code related to perl.



As a start, the following is some useful general info excerpted from <ins>Effective Perl Programming: Ways to Write Better, More Idiomatic Perl</ins>, Second Edition
by brian d foy; Joseph N. Hall; Joshua A. McAdams
Published by Addison-Wesley Professional, 2010:

## Regular expressions are made up of atoms and operators. Atoms are generally single-character matches. For example:

```
a      # matches the letter a
\$     # mathces the character
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

Although Perl provides no method of automatically naming parameters in the function to which you pass them (in other words, no “formal parameters”), there are a variety of ways that you can call functions with an argument list that provides both names and values. All of these mechanisms require that the function you call do some extra work while processing the argument list. In other words, this feature isn’t built into Perl either, but it’s a blessing in disguise. Different implementations of named parameters are appropriate at different times. Perl makes it easy to write and use almost any implementation you want. A simple approach to named parameters constructs a hash out of the argument list:

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

The below table shows the characers you can use in a prototype.

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
