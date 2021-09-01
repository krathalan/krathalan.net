### [🠔 back to Home](https://krathalan.net/)

# Krathalan's Bash Guidebook
# Chapter 1: Basics and Variables

Welcome to Krathalan's Bash guidebook. In the first chapter, we are going to go over expectations and some basic features of the Bash scripting language.

You should have at least some cursory experience with programming. You should understand what a variable and a function is, and you should have an editor like vim, nano, or VS Code set up and ready to go. You should know how to use Bash interactively as a shell, at least at a basic level of navigating your files and knowing a couple commands.

This guidebook is not intended to be a "full" guide to Bash scripting. It is intended to be a guide that shows you the most useful features and tricks of Bash to help you get started with writing safer and more maintainable Bash scripts as fast as possible.

This first chapter will look at suggested reading and Shellcheck, the shebang, setting options, printf, variables, and parameter substitution.

## Suggested reading
An amazing, long, well-written list of "how do I do X" in Bash can be found here: https://github.com/dylanaraps/pure-bash-bible

This guidebook will cover some features and methods contained in the Pure Bash Bible, but hopefully they will be a bit more explained and detailed here.



## shebang
The first element of any Bash script is line 1: the shebang. Its purpose is to instruct the computer what kind of script it is. You may find some people that say it should definitely be one instead of the others, but generally any one of these are acceptable:

```
#!/bin/bash

#!/usr/bin/bash

#!/usr/bin/env bash
```

While they all work on Arch Linux, some distros which do not symlink `/bin` to `/usr/bin` or which keep the `bash` binary in a strange place might work better with `#!/usr/bin/env bash`.

❗ Warning! Some scripts use `#!/bin/sh` as their shebang. `sh` is generally thought of as strictly a POSIX-compliant shell, which is different from Bash and doesn't include some Bash features, like tab completion or arrays for example. While most distros symlink `/bin/sh` to `bash`, some do not and it is not expected behavior. On Arch Linux, a fair amount of users symlink `/bin/sh` to `dash`, an implementation of `sh`; around 10% of Arch users have `dash` installed [\[↗\]](https://pkgstats.archlinux.de/packages#query=dash), and it serves as `/bin/sh` on Debian [\[↗\]](https://wiki.debian.org/Shell).

## set
By default, Bash is missing a lot of features that make more robust programming languages safe, like exiting immediately when an error occurs. By default, if an error occurs in a Bash script, the script will continue unless explicitly told to exit, since that's how Bash behaves as an interactive shell. You can use the `set` Bash built-in command to turn on some safety features.

With `set -Ee`, you can tell your script to exit immediately upon any command failing. `set -u` will cause your script to exit whenever you attempt to reference a variable that doesn't exist. `set -o pipefail` with `set -Ee` will cause your script to exit if any command in a pipeline (e.g. `grep pattern input | cut -d' ' -f3 | ...`) fails.

You can use all these together by putting `set -Eeuo pipefail` at the top of your script, just after the shebang. For some commands, you may wish to have your script continue if the command fails. Before that command, you can "unset" with `set +Eeo pipefail`, run the command, and then re-set with `set -Eeo pipefail` after the command.

To assist with debugging, you can `set -x`, which will print every line executed by the script after the `set -x`. Similarly, you can `set +x` to stop printing every line executed.

## echo vs. printf
You should use `printf` to print to the terminal. `printf` gives you stronger control over the output format, and gives you extra options for number output, such as specifying the number of decimals to print. `printf` uses the same "format string" style as C strings, as well as some other programming languages. 

To use `printf`, first specify a format string, then specify the values to substitute. For example:

```
1  #!/bin/bash
2  printf "Today is %s, %s\n" "Monday" "April 21"

Output:
Today is Monday, April 21
```

`%s` tells printf to look at the arguments passed and replace `%s` with the argument in the actual output. The first `%s` will be replaced with the first argument `"Monday"`, and the second `%s` will be replaced with the second argument `"April 21"`.

The `\n` tells printf to print a newline.

## Variables
Variables in Bash are fairly simple. You never need to specify a variable type, since all variables are technically strings (but Bash is still smart enough to be able to perform integer math). Variable declaration looks like this:

```
1   #!/bin/bash
2   readonly IMPORTANT_NUMBER=420
3
4   someVariable="This is a string"
5   todaysDate="$(date)"
6 
7   some_function()
8   {
9     local scopedVariable="You can only use me in this function"
10  }
```

Notice that there are no spaces between the variable name, equals sign, and value. You must quote string values with more than one word. Variables are global by default, meaning they can be accessed anywhere in the script after declaration. To scope a variable to one function only, you can use the `local` keyword before declaring the variable (line 8). Trying to use local variables as much as is reasonable, especially in larger Bash scripts, can help you keep your script cleaner.

Command substitution is an important tool in Bash scripting. It allows you to assign or manipulate output from commands in unique ways. For example, line 4, `todaysDate="$(date)"`, allows us to assign the output of the `date` command to the variable `todaysDate`. You can combine command substitution with other things in a variable assignment, for example `todaysDate="Today is $(date)"`.

You can reference a variable with the `$` symbol before its name. You should always contain variables in quotes, and you should use curly braces `{}` to enclose your variable name to prevent confusion when you surround your variable with other text, unless it is a special argument variable, which we'll talk about later. For example:

```
1  #!/bin/bash
2  someNumber=8
3  printf "I have %s apples\n" "${someNumber}"

Output:
I have 8 apples
```

To prevent a variable from being modified, you can add the `readonly` keyword before declaring a variable. If you want the variable to be `local` AND `readonly`, put `local -r` before declaring. Crucially, you can declare a global or local variable, and then make it readonly later. For example:

```
1  #!/bin/bash
2  someNumber=8
3
4  printf "I have %s apples\n" "${someNumber}"
5
6  readonly someNumber=9
7  someNumber=8
8
9  printf "%s\n" "${someNumber}"

Output:
I have 8 apples
bash: someNumber: readonly variable
```

This example will fail with the error `"bash: someNumber: readonly variable"` because you cannot assign new values to a readonly variable.

## Parameter substitution
Parameter substitution is a fantastic way to quickly get the values you need from complex strings, among other things. There are several different types of parameter substitution with unique syntax; I will show my most used parameter substitution techniques here, but there are quite a few more.

- `${var#pattern}` : Remove **shortest** match of 'pattern' from the **beginning** (left) of `${var}`
- `${var##pattern}` : Remove **longest** match of 'pattern' from the **beginning** (left) of `${var}`
- `${var%pattern}` : Remove **shortest** match of 'pattern' from the **end** (right) of `${var}`
- `${var%%pattern}` : Remove **longest** match of 'pattern' from the **end** (right) of `${var}`
- `${var/pattern/replacement}` : Replace all instances of 'pattern' with 'replacement' in `${var}`

You can remember which is which because `#` furthest to to the left on the keyboard, and `%` is furthest right.

```
1   #!/bin/bash
2  
3   someDate="5/9/1943" # d/m/y
4
5   # Remove the shortest match of "/*" from the end of the string, printing only d/m (5/9)
6   printf "%s\n" "${someDate%/*}"
7
8   # Remove the longest match of "/*" from the end of the string, printing only d (5)
9   printf "%s\n" "${someDate%%/*}"
8
9   # Remove the shortest match of "*/" from the beginning of the string, printing only m/y (9/1943)
10  printf "%s\n" "${someDate#*/}
11
12  # Remove the longest match of "*/" from the beginning of the string, printing only y (1943)
13  printf "%s\n" "${someDate##*/}"
14
15  # Remove the shortest match of "/*" from the end of the string and
16  # the shortest match of "*/" from the beginning of the string, printing only m (9)
17  tmpDate="${someDate%/*}"
18  printf "%s\n" "${tmpDate#*/}"
19
20  # Print the filename without the full path
21  somePath="/home/user/documents/school/semester-01/cs/project-one.c"
22  printf "%s\n" "${somePath##*/}"
23
24  # Replace all instances of "apple" with "lemon"
25  someString="I love apples. My mom gave me an apple. Apples are great."
26  printf "%s\n" "${someString//apple/lemon}"

Output:
5/9
5
9/1943
1943
9
project-one.c
I love lemons. My mom gave me an lemon. Apples are great.
```

Parameter substitution is case sensitive, which is why the capitalized "Apples" in the last sentence was not replaced. Additionally, you should be aware of grammar errors when using parameter substitution -- notice the "an" in the second sentence was not replaced, since we only replaced "apple" with "lemon", leaving "an lemon". You could correct this issue by replacing "an apple" with "a lemon", THEN replacing the remaining "apple" with "lemon".

Corrected code:

```
1   #!/bin/bash
  ...
24  # Replace all instances of "apple" with "lemon" correctly
25  someString="I love apples. My mom gave me an apple. Apples are great."
26  someString="${someString//an apple/a lemon}"
27  someString="${someString//Apple/Lemon}"
28  printf "%s\n" "${someString//apple/lemon}"
 
Output:
...
I love lemons. My mom gave me a lemon. Lemons are great.
```