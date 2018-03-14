# perl-stuff
This is currently where I've decided to dump my notes/code related to perl.

As a start, here's some stuff related to regular expressions in perl:

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

