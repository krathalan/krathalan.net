### [🠔 back to Home](https://krathalan.net/)

# Krathalan's Bash Guidebook: Chapter 2
While parameter substitution can help you get what you need, sometimes you need an external program. You should try to use Bash's built-in features as much as you can, since opening external programs is the #1 way to slow down your Bash script -- especially if you are opening a lot of external programs in a loop.

## grep
`grep` is a tool almost every terminal user is familiar with, so I won't spend too long going over its features. Some essential flags include:
- `-i` : ignore case when searching input for a match
- `-R` : search all files in a directory for a match, following symbolic links (`-r` does not follow symbolic links)
- `-q` : useful to test if input contains a pattern without needing the output of the `grep` command
- `-n` : display a line number for matching lines
- `-v` : invert the search, printing lines which do not match the pattern

You can use `^` at the start of your pattern to only match patterns which are at the beginning of a line. For example, given the input:

```
apples are good
this is an apple
```

And the command `grep '^apple'`, only the first line "**apple**s are good" will match. Conversely, you can use `$` at the end of your pattern to only match patterns which are at the end of a line. Given the command `grep 'apple$'` on the same input, only the second line "this is an **apple**" will match.

## sed
Sed is useful for replacing 

## sort and uniq