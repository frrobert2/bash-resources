#pragma section-numbers 2
[[BashGuide/JobControl|<- Job Control]]

----
<<TableOfContents>> <<Anchor(StartOfContent)>>

= Practices =
== Choose Your Shell ==
The first thing you should do before starting a shell script, or any kind of script or program for that matter, is enumerate the requirements and the goal of that script.  Then evaluate what the best tool is to accomplish those goals.

[[BASH]] may be easy to learn and write in, but it isn't always fit for the job.

There are a lot of tools in the basic toolset that can help you.  If you just need AWK, then don't make a shell script that invokes AWK.  Just make an AWK script. If you need to retrieve data from an HTML or XML file in a reliable manner, Bash is also '''the wrong tool for the job'''.  You should consider XPath/XSLT instead, or a language that has a library available for parsing XML or HTML.

If you decide that a shell script is what you want, then first ask yourself these questions:

 * In the foreseeable future, might your script be needed in an environment where Bash is not available ''by default''?
  * If so, '''then consider sh instead.'''  `sh` is a POSIX shell and its features are available in any shell that complies with the POSIX standard.  You can rely on the fact that any POSIX system will be able to run your script.  You will have to balance the needs of portability against the Bash features you'd be unable to use.
  * ''Keep in mind that this guide does not apply to sh!''  The [[Bashism]] page gives some alternatives, but it is not complete.

 * Are you certain that in all environments you will run and might want to run this script in the future Bash 3.x (or 4.x) will be available to you?
  * '''If not, you should limit yourself to features of Bash 2.x ONLY.'''

If the above questions do not limit your choices, use all the Bash features you require, and then make note of which version of Bash is required to run your script.

Using Bash 3 or higher means you can avoid ancient scripting techniques.  They have been replaced with far better alternatives for ''very good reasons''.

 * Stay away from scripting examples you see on the Web unless you understand them completely.  Mostly any scripts you will find on the Web are broken in some way.  Do not copy/paste from them.

 * Always use the correct shebang.  If you're writing a Bash script, you '''need''' to put `#!/usr/bin/env bash` at the top of your script.  Omitting this header or using the `#!/bin/sh` header is '''wrong'''.  In the latter case you will no longer be able to use any Bash features.  You will be limited to the POSIX shell scripting standard (even if your `/bin/sh` links to Bash).

 * When writing Bash scripts, do '''not''' use the `[` command.  Bash has a far better alternative: `[[`.  Bash's `[[` is more reliable in many ways and there are no advantages whatsoever to sticking with the old relic.  You'll also miss out on a lot of features provided by `[[` that `[` does not have (like pattern matching).

 * It's also time you forgot about {{{`...`}}}.  It isn't consistent with the syntax of ''Expansion'' and is terribly hard to nest without a dose of painkillers.  Use `$(...)` instead.

 * And for heaven's sake, '''"Use more quotes!"'''  Protect your strings and parameter expansions from WordSplitting.  Word splitting will eat your babies if you don't quote things properly.

 * Learn to use [[BashGuide/Parameters#Parameter_Expansion|parameter expansions]] instead of `sed` or `cut` to manipulate simple strings in Bash.  If you want to remove the extension from a filename, use `${filename%.*}` instead of {{{`echo "$filename" | sed 's/\.[^.]*$//'`}}} or some other dinosaur.

 * Use built-in math instead of `expr` to do simple calculations, ''especially'' when just incrementing a variable.  If the script you're reading uses {{{x=`expr $x + 1`}}} then it's not something you want to mimic.

== Quoting ==
''Word Splitting'' is the demon inside Bash that is out to get unsuspecting newcomers or even veterans who let down their guard.

If you do not understand how ''Word Splitting'' works or when it is applied you should be very careful with your strings and your ''Parameter Expansion''s.  I suggest you read up on WordSplitting if you doubt your knowledge.

The best way to protect yourself from this beast is to quote all your strings.  Quotes keep your strings in one piece and prevent ''Word Splitting'' from tearing them open.  Allow me to illustrate:

{{{
$ echo Push that word             away from me.
Push that word away from me.
$ echo "Push that word             away from me."
Push that word             away from me.
}}}
Now, don't think ''Word Splitting'' is about collapsing spaces.  What really happened in this example is that the first command passed each word in our sentence as a separate argument to `echo`.  Bash split our sentence up in words using the whitespace between them to determine where each argument begins and ends.  In the second example Bash is forced to keep the whole quoted string together.  This means it's not split up into arguments and the whole string is passed to `echo` '''as one argument'''.  `echo` prints out each argument it gets with a space in between them.  You should understand the basics of ''Word Splitting'' now.

This is where it gets dangerous: ''Word Splitting'' does not just happen on literal strings.  It also happens '''after ''Parameter Expansion'''''!  As a result, if you missed a cup of coffee this morning you may find yourself making this mistake:

{{{
$ sentence="Push that word             away from me."
$ echo $sentence
Push that word away from me.
$ echo "$sentence"
Push that word             away from me.
}}}
As you can see, in the first `echo` command, we neglected to add quotes. Bash expanded our sentence and then used ''Word Splitting'' to split the resulting expansion up into arguments to use for `echo`, which had the result of destroying our carefully thought-out formatting.  In our second example, the quotes around the ''Parameter Expansion'' of `sentence` make sure Bash '''does not''' split it up into multiple arguments around the whitespace.

It's not just spaces you need to protect.  ''Word Splitting'' occurs on all whitespace, including tabs, newlines, and any other characters in the `IFS` variable.  Here's another example to show you just how badly you can break things if you neglect to quote when necessary:

{{{
$ echo "$(ls -al)"
total 8
drwxr-xr-x   4 lhunath users 1 2007-06-28 13:13 "."/
drwxr-xr-x 102 lhunath users 9 2007-06-28 13:13 ".."/
-rw-r--r--   1 lhunath users 0 2007-06-28 13:13 "a"
-rw-r--r--   1 lhunath users 0 2007-06-28 13:13 "b"
-rw-r--r--   1 lhunath users 0 2007-06-28 13:13 "c"
drwxr-xr-x   2 lhunath users 1 2007-06-28 13:13 "d"/
drwxr-xr-x   2 lhunath users 1 2007-06-28 13:13 "e"/
$ echo $(ls -al)
total 8 drwxr-xr-x 4 lhunath users 1 2007-06-28 13:13 "."/ drwxr-xr-x 102 lhunath users 9 2007-06-28 13:13 ".."/ -rw-r--r-- 1 lhunath users 0 2007-06-28 13:13 "a" -rw-r--r-- 1 lhunath users 0 2007-06-28 13:13 "b" -rw-r--r-- 1 lhunath users 0 2007-06-28 13:13 "c" drwxr-xr-x 2 lhunath users 1 2007-06-28 13:13 "d"/ drwxr-xr-x 2 lhunath users 1 2007-06-28 13:13 "e"/
}}}
In some very rare occasions it may be desired to leave out the quotes.  That's if you '''need''' ''Word Splitting'' to take place:

{{{
$ friends="Marcus JJ Thomas Michelangelo"
$ for friend in $friends
> do echo "$friend is my friend!"; done
Marcus is my friend!
JJ is my friend!
Thomas is my friend!
Michelangelo is my friend!
}}}
But, honestly?  You should use arrays for nearly all of these cases.  Arrays have the benefit that they separate strings without the need for an explicit delimiter.  That means your strings in the array can contain '''any''' valid (non-NUL) character, without the worry that it might be your string delimiter (like the space is in our example above).  Using arrays in our example above gives us the ability to add middle or last names of our friends:

{{{
$ friends=( "Marcus The Rich" "JJ The Short" "Timid Thomas" "Michelangelo The Mobster" )
$ for friend in "${friends[@]}"
> do echo "$friend is my friend"; done
}}}
Note that in our previous `for` we used an ''unquoted'' `$friends`.  This allowed Bash to split our `friends` string up into words.  In this last example, we quoted the `${friends[@]}` ''Parameter Expansion''.  Quoting an expansion of an array through the `@` index makes Bash expand that array into a sequence of its elements, '''where each element is wrapped in quotes'''.

== Readability ==
Almost as important as the result of your code is the '''readability of your code'''.

Chances are that you aren't just going to write a script once and then forget about it.  If so, you might as well delete it after using it.  If you plan to continue using it, you should also plan to continue maintaining it.  Unlike your house your code won't get dirty over time, but you will learn new techniques and new approaches constantly.  You will also gain insight about how your script is used.  All that new information you gather since the completion of your initial code should be used to maintain your code in such a way that it constantly improves.  Your code should keep growing more user-friendly and more stable.

 . Trust me when I say, no piece of code is ever 100% finished, with the exception of some very short and useless chunks of nothingness.

To make it easier for yourself to keep your code healthy and improve it regularly you should keep an eye on the readability of what you write.  When you return to a long loop after a year has passed since your last visit to it and you wish to improve it, add a feature, or debug something about it, consider whether you'd rather see this:

{{{#!highlight bash
friends=( "Marcus The Rich" "JJ The Short" "Timid Thomas" "Michelangelo The Mobster" )

# Say something significant about my friends.
for name in "${friends[@]}"; do

    # My first friend (in the list).
    if [[ $name = "${friends[0]}" ]]; then
        echo "$name was my first friend."

    # My friends whose names start with M.
    elif [[ $name = M* ]]; then
        echo "$name starts with an M"

    # My short friends.
    elif [[ " $name " = *" Short "* ]]; then
        echo "$name is a shorty."

    # Friends I kind of didn't bother to remember.
    else
        echo "I kind of forgot what $name is like."

    fi
done
}}}
Than something like this:

{{{#!highlight bash
x=(       Marcus\ The\ Rich JJ\ The\ Short
  Timid\ Thomas Michelangelo\ The\ Mobster)
for name in "${x[@]}"
  do if [ "$name" = "$x" ]; then echo $name was my first friend.
 elif
   echo $name    |   \
  grep -qw Short
    then echo $name is a shorty.
 elif [ "x${name:0:1}" = "xM" ]
     then echo $name starts   with an M; else
echo I kind of forgot what $name \
 is like.; fi; done
}}}
And yes, I know this is an exaggerated example, but I've seen some authentic code that actually has a lot in common with that last example.

For your own health, keep these few points in mind:

 * ''Healthy whitespace gives you breathing space''.  Indent your code properly and consistently.  Use blank lines to separate paragraphs or logic blocks.
 * ''Avoid backslash-escaping''.  It's counter-intuitive and boggles the mind when overused.  Even in small examples it takes your mind more effort to understand `a\ b\ c` than it takes to understand `'a b c'`.
 * ''Comment your way of thinking before you forget''.  You might find that even code that looks totally common sense right now could become the subject of ''"What the hell was I thinking when I wrote this?"'' or ''"What is '''this''' supposed to do?"''.
 * ''Consistency prevents mind boggles''.  Be consistent in your naming style.  Be consistent in your use of capitals.  Be consistent in your use of shell features.  In coding, unlike in the bedroom, it's ''good'' to be simple and predictable.

== Bash Tests ==
The `test` command, also known as `[`, is an application that usually resides somewhere in `/usr/bin` or `/bin` and is used a lot by shell programmers to perform certain tests on files and variables. In a number of shells, including Bash, `test` is also implemented as a shell builtin.

It can produce surprising results, especially for people starting shell scripting that think [ ] is part of the shell syntax.

If you use `sh`, you have little choice but to use `test` as it is the only way to do most of your testing.

If however you are using Bash to do your scripting (and I presume you are since you're reading this guide), then you can also use the `[[` keyword.  While it still behaves in many ways like a command, it presents several advantages over the traditional `test` command.

Let me illustrate how `[[` can be used to replace `test`, and how it can help you to avoid some of the common mistakes made by using `test`:

{{{
$ var=''
$ [ $var = '' ] && echo True
-bash: [: =: unary operator expected
$ [ "$var" = '' ] && echo True
True
$ [[ $var = '' ]] && echo True
True
}}}
`[ $var = '' ]` expands into `[ = '' ]`.  The first thing `test` does is count its arguments.  Since we're using the `[` form, we'll just strip off the mandatory `]` argument at the end.  In the first example, `test` sees two arguments: `=` and `''`.  It knows that if it has two arguments, the first one has to be a ''unary operator'' (an operator that takes one operand).  But `=` is not a unary operator (it's binary -- it requires ''two'' operands), so `test` blows up.

Yes, `test` did not see our empty `$var` because Bash expanded it into nothingness before `test` could even see it.  Moral of the story?  '''Use more quotes!'''  Using quotes, `[ "$var" = '' ]` expands into `[ "" = '' ]` and `test` has no problem.

Now, `[[` can see the whole command before it's being expanded.  It sees `$var`, and not the expansion of `$var`.  As a result, there is no need for the quotes at all!  '''`[[` is safer'''.

{{{
$ var=
$ [ "$var" < a ] && echo True
-bash: a: No such file or directory
$ [ "$var" \< a ] && echo True
True
$ [[ $var < a ]] && echo True
True
}}}
In this example we attempted a string comparison between an empty variable and '`a`'.  We're surprised to see the first attempt does not yield `True` even though we think it should.  Instead, we get some weird error that implies Bash is trying to open a file called '`a`'.

We've been bitten by ''File Redirection''.  Since `test` is just an application, the `<` character in our command is interpreted (as it should) as a ''File Redirection'' operator instead of the string comparison operator of `test`. Bash is instructed to open a file '`a`' and connect it to `stdin` for reading.  To prevent this, we need to escape `<` so that `test` receives the operator rather than Bash.  This makes our second attempt work.

Using `[[` we can avoid the mess altogether. `[[` sees the `<` operator before Bash gets to use it for ''Redirection'' -- problem fixed.  Once again, '''`[[` is safer'''.

Even more dangerous is using the `>` operator instead of our previous example with the `<` operator.  Since `>` triggers output ''Redirection'' it will create a file called '`a`'.  As a result, '''there will be no error message warning us that we've committed a sin'''! Instead, our script will just break. Even worse, we might overwrite some important file!  It's up to us to ''guess'' where the problem is:

{{{
$ var=a
$ [ "$var" > b ] && echo True || echo False
True
$ [[ "$var" > b ]] && echo True || echo False
False
}}}
Two different results, not good.  The lesson is to trust `[[` more than `[`.  `[ "$var" > b ]` is expanded into `[ "a" ]` and the output of that is being redirected into a new file called '`b`'.  Since `[ "a" ]` is the same as `[ -n "a" ]` and that basically tests whether the `"a"` string is non-empty, the result is a success and the `echo True` is executed.

Using `[[` we get our expected scenario where `"a"` is tested against `"b"` and since we all know `"a"` sorts before `"b"` this triggers the `echo False` statement.  And this is how you can break your script without realizing it.  You will however have a suspiciously empty file called '`b`' in your current directory.

Yes it adds a few characters, but '''`[[` is far safer than `[`'''.  Everybody inevitably makes programming errors. Even if you try to use `[` safely and carefully avoid these mistakes, I can assure you that you '''will''' make them. And if other people are reading your code, you can be sure that they'll absolutely mess things up.

Besides `[[` provides the following features over `[`:

 * '''`[[` can do glob pattern matching:'''
 {{{
[[ abc = a* ]]
}}}
 * '''`[[` can do regex pattern matching (Bash 3.1+):'''
 {{{
[[ abb =~ ab+ ]]
}}}

The only advantage of `test` is its portability.

== Don't Ever Do These ==
The Bash shell allows you to do quite a lot of things, offering you considerable flexibility.  Unfortunately, it does very little to discourage [[BashPitfalls|misuse]] and other ill-advised behavior.  It hopes people will find out for themselves that certain things should be avoided at all costs.

Unfortunately many people don't care enough to want to find out for themselves.  They write without thinking things through and many awful and [[ShellHallOfShame|dangerous scripts]] end up in production environments or in Linux distributions.  The result of these, and even your very own scripts written in a time of neglect, can often be '''DISASTROUS'''.

That said, for the good of your scripts and for the rest of mankind, '''Never Ever Do Anything Along These Lines''':

 * '''`ls -l | awk '{ print $8 }'`'''
  . '''DON'T EVER [[ParsingLs|parse the output]] of `ls`!'''  The `ls` command's output cannot be trusted for a number of reasons.
   1. For one, `ls` will mangle the filenames of files if they contain characters unsupported by your locale.  As a result, parsing filenames out of `ls` is NEVER guaranteed to actually give you a file that you will be able to find.  `ls` might have replaced certain characters in the filename by question marks.
   1. Secondly, `ls` separates lines of data by newlines.  This way, every bit of information on a file is on a new line.  '''UNFORTUNATELY filenames themselves can ALSO contain newlines'''.  This means that if you have a filename that contains a newline in the current directory, it will completely break your parsing and as a result, break your script!
   1. Last but not least, the output format of `ls -l` is NOT guaranteed consistent across platforms.  Some systems omit the group ID by default, for instance, and reverse the effect of the `-g` switch.  Some systems use two fields for the modification time, and some use three.  On the systems that use three, the third field can be either a year, or an HH:MM concatenation, depending on how old the file is.
  There are [[BashFAQ/087|alternatives to using ls]] in many cases.  If you need to work with file modification times, you can typically use [[BashGuide/TestsAndConditionals|Bash Tests]].  If none of those things is possible, ''I recommend you switch to a different language, like python or perl''.

 * '''`if echo "$file" | fgrep .txt; then`'''<<BR>> '''`ls *.txt | grep story`'''
  . '''DON'T EVER test or filter filenames with `grep`!'''  Unless your `grep` pattern is really smart it will probably not be trustworthy.
  . In the first example above, the test would match both `story.txt` AND `story.txt.exe`.  If you make `grep` patterns that are smart enough, they'll probably be so ugly, massive and unreadable that you shouldn't use them anyway.
  . The alternative is called [[glob|globbing]].  Bash has a feature called ''Pathname Expansion''.  This will help you enumerate all files that match a certain pattern.  Also, you can use globs to test whether a filename matches a certain glob pattern (in a `case` or `[[` command).

 * '''`cat file | grep pattern`'''
  . Don't use `cat` to feed a single file's content to a filter.  `cat` is a utility used to concatenate the contents of several files together.
  . To feed the contents of a file to a process you will probably be able to pass the filename as an argument to the program (`grep 'pattern' /my/file`, `sed 'expression' /my/file`, ...).
  . If the manual of the program does not specify any way to do this, you should use redirection (`read column1 column2 < /my/file`, `tr ' ' '\n' < /my/file`, ...).

 * '''`for line in $(<file); do`'''
  . [[DontReadLinesWithFor|Don't use a for loop]] to read the lines of a file.  Use a [[BashFAQ/001|while read]] loop instead.

 * '''`for number in $(seq 1 10); do`'''
  . For the love of god and all that is holy, do not use `seq` to count.
  . Bash is able enough to do the counting for you.  You do not need to spawn an external application (especially a single-platform one) to do some counting and then pass that application's output to Bash for word splitting.  Learn the syntax of `for` already!
  . In general, C-style `for` loops are the best method for implementing a counter `for ((i=1; i<=10; i++))`.
  . If you actually wanted a stream of numbers separated by newlines as test input, consider `printf '%d\n' {1..10}`. You can also loop over the result of a sequence expansion if the range is constant, but the shell needs to expand the full list into memory before processing the loop body. This method can be wasteful and is less versatile than other arithmetic loops.

 * '''`i=`{{{`}}}`expr $i + 1`{{{`}}}'''
  . `expr` is a relic of ancient Rome.  Do not wield it.
  . It was used in scripts written for shells with very limited capabilities.  You're basically spawning a new process, calling another C program to do some math for you and return the result as a string to bash.  Bash can do all this itself and so much faster, more reliably (no numbers->string->number conversions) and in all, better.
  . You should use this in Bash: `let i++` or `((i++))`
  . Even POSIX sh can do arithmetic: `i=$(($i+1))`.  It only lacks the `++` operator and the `((...))` command (it has only the `$((...))` substitution).

== Debugging ==
Very often you will find yourself clueless as to why your script isn't acting the way you want it to.  Resolving this problem is always just a matter of common sense and debugging techniques.

=== Diagnose the Problem ===
Unless you know what exactly the '''problem''' is, you most likely won't come up with a ''solution'' anytime soon.  So make sure you understand what exactly goes wrong.  Evaluate the symptoms and/or error messages.

Try to formulate the problem as a sentence.  This will also be vital if you're going to ask other people for help with your problem.  You don't want them to have to go through your whole script or run it so that they understand what's going on.  No; '''you''' need to make the problem perfectly clear to '''yourself''' and to anybody trying to help you.  ''This requirement stands until the day the human race invents means of telepathy''.

=== Minimize the Codebase ===
If staring at your code doesn't give you a divine inspiration, the next thing you should do is try to minimize your codebase to isolate the problem.

Don't worry about preserving the functionality of your script.  The only thing you want to preserve is the logic of the code block that seems buggy.

Often, the best way to do this is to copy your script to a new file and start deleting everything that seems irrelevant from it.  Alternatively, you can make a new script that does something similar in the same code fashion, and keep adding structure until you duplicate the problem.

As soon as you delete something that makes the problem go away (or add something that makes it appear), you'll have found where the problem lies.  Even if you haven't precisely pinpointed the issue, at least you're not staring at a massive script anymore, but hopefully at a stub of no more than 3-7 lines.

For example, if you have a script that lets you browse images in your image folder by date, and for some reason you can't manage to iterate over your images in the folder properly, it suffices to reduce the script to this part:

{{{
for image in $(ls -R "$imgFolder"); do
    echo "$image"
done
}}}
Your actual script will be far more complex, and the inside of the `for` loop will also be far longer.  But the essence of the problem is this code.  Once you've reduced your problem to this it may be easier to see the problem you're facing.  Your `echo` spits out parts of image names; it looks like all whitespace is replaced by newlines.  That must be because `echo` is run once for each chunk terminated by whitespace, not for every image name (as a result, it seems the output has split open image names with whitespace in them).  With this reduced code, it's easier to see that the cause is actually your `for` statement that splits up the output of `ls` into words.  That's because `ls` is '''UNPARSABLE''' in a bugless manner (do not ever use `ls` in scripts, unless if you want to show its output to a user).

We can't use a recursive glob (unless we're in bash 4), so we have to use `find` to retrieve the filenames.  One fix would be:

{{{
find "$imgFolder" -print0 | while IFS= read -r -d '' image; do
    echo "$image"
done
}}}
Now that you've fixed the problem in this tiny example, it's easy to merge it back into the original script.

=== Activate Bash's Debug Mode ===
If you still don't see the error of your ways, Bash's debugging mode might help you see the problem through the code.

When Bash runs with the `x` option turned on, it prints out every command it executes before executing it (to standard error).  That is, '''after any expansions have been applied'''.  As a result, you can see exactly what's happening as each line in your code is executed.  Pay very close attention to the quoting used.  Bash uses quotes to show you exactly which strings are passed as a single argument.

There are three ways of turning on this mode.

 * Run the script with `bash -x`:
 {{{
$ bash -x ./mybrokenscript
}}}
 * Modify your script's header:
 {{{#!text
#!/bin/bash -x
[.. script ..]
}}}
 . Or:
 {{{#!text
#!/usr/bin/env bash
set -x
}}}
 * Or add `set -x` somewhere in your code to turn on this mode for only a specific block of your code:
 {{{#!text
#!/usr/bin/env bash
[..irrelevant code..]
set -x
[..relevant code..]
set +x
[..irrelevant code..]
}}}

Because the debugging output goes to stderr, you will generally see it on the screen, if you are running the script in a terminal.  If you would like to log it to a file, you can tell Bash to send all stderr to a file:

{{{
exec 2>> /path/to/my.logfile
set -x
}}}
A nice feature of bash version >= 4.1 is the variable BASH_XTRACEFD. This allows you to specify the file descriptor to write the `set -x` debugging output to. In older versions of bash, this output always goes to stderr, and it is difficult if not impossible to keep it separate from normal output (especially if you are logging stderr to a file, but you need to see it on the screen to operate the program).  Here's a nice way to use it:

{{{#!highlight bash
# dump set -x data to a file
# turns on with a filename as $1
# turns off with no params
setx_output()
{
    if [[ $1 ]]; then
        exec {BASH_XTRACEFD}>>"$1"
        set -x
    else
        set +x
        unset -v BASH_XTRACEFD
    fi
}
}}}
If you have a complicated mess of scripts, you might find it helpful to change PS4 before setting -x. If the value assigned to PS4 is surrounded by double quotes it will be expanded during variable assignment, which is probably not what you want; with single quotes, the value will be expanded when the PS4 prompt is displayed.

{{{
PS4='+$BASH_SOURCE:$LINENO:$FUNCNAME: '
}}}
=== Step Your Code ===
If the script goes too fast for you, you can enable code-stepping.  The following code uses the `DEBUG` trap to inform the user about what command is about to be executed and wait for his confirmation do to so. Put this code in your script, at the location you wish to begin stepping:

{{{
debug_prompt () { read -p "[$BASH_SOURCE:$LINENO] $BASH_COMMAND?" _ ;}
trap 'debug_prompt "$_"' DEBUG
}}}
=== The Bash Debugger ===
The Bash Debugger Project is a gdb-style debugger for bash, available from http://bashdb.sourceforge.net/

The Bash Debugger will allow you to walk through your code and help you track down bugs.

=== Reread the Manual ===
If your script still doesn't seem to agree with you, maybe your perception of the way things work is wrong.  Try going back to the manual (or this guide) to re-evaluate whether commands do exactly what you think they do, or the syntax is what you think it is.  Very often people misunderstand what `for` does, how ''Word Splitting'' works, or how often they should use quotes.

Keep the tips and good practice guidelines in this guide in mind as well.  They often help you avoid bugs and problems with scripts.

I mentioned this in the ''Scripts'' section of this guide too, but it's worth repeating it here.  First of all, make sure your script's '''header''' is actually '''`#! /bin/bash`'''.  If it is '''missing''' or if it's something like '''`#! /bin/sh`''' then you deserve the problems you're having.  That means you're probably not even using Bash to run your code.  Obviously that'll cause issues.  Also, make sure you have no ''Carriage Return'' characters at the ends of your lines.  This is the cause of scripts written in ''Microsoft Windows(tm)''.  You can get rid of these fairly easily like this:

{{{
$ tr -d '\r' < myscript > tmp && mv tmp myscript
}}}
=== Read the FAQ / Pitfalls ===
The [[BashFAQ]] and BashPitfalls pages explain common misconceptions and issues encountered by other Bash scripters.  It's very likely that your problem will be described there in some shape or form.

To be able to find your problem in there, you'll obviously need to have ''Diagnosed'' it properly.  You'll need to know what you're looking for.

=== Ask Us on IRC ===
There are people in the `#bash` channel almost 24/7.  This channel resides on the [[https://libera.chat|Libera Chat]] IRC network.  To reach us, you need an IRC client.  Connect it to `irc.libera.chat`, and `/join #bash`.

Make '''sure''' that you know what your real problem is and have stepped through it on paper, so you can  explain it well.  We don't like having to guess at things.  Start by explaining what you're trying to do with your script.

Either way, please have a look at this page before entering `#bash`: XyProblem.

<<Anchor(EndOfContent)>>

----
[[BashGuide/JobControl|<- Job Control]]
