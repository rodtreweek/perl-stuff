# perl-stuff
This is currently where I've decided to dump various notes and example code related to perl.



As a start, the following is some useful general info excerpted from <ins>Effective Perl Programming: Ways to Write Better, More Idiomatic Perl</ins>, Second Edition
by brian d foy; Joseph N. Hall; Joshua A. McAdams
Published by Addison-Wesley Professional, 2010

Regular expressions are made up of atoms and operators. Atoms are generally single-character matches. For example:

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

Although Perl provides no method of automatically naming parameters in the function to which you pass them (in other words, no “formal parameters”), there’s a variety of ways that you can call functions with an argument list that provides both names and values. All of these mechanisms require that the function you call do some extra work while processing the argument list. In other words, this feature isn’t built into Perl either, but it’s a blessing in disguise. Different implementations of named parameters are appropriate at different times. Perl makes it easy to write and use almost any implementation you want.A simple approach to named parameters constructs a hash out of the argument list:
