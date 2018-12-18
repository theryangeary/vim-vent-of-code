# This one was actually quite a doozy!
I separated it into two parts. Step 1: Find the guard that sleeps most. Step 2:
Look at just that guard's data and find when they sleep most.

## Step 1

First I sorted all the input so it was chronological.

```:sort```

Then, separate each shift into a paragraph

```:g/Guard/norm O<Esc>```

Turn every paragraph into a line. This will allow me to sort again to get each
Guard's shifts all together. I used the @ as a separator because it isn't ever
used as part of the text.

```:g/Guard/norm vip:s/\n/@/g<cr>o<Esc>```

Now sort by Guard

```:sort /Guard/```

At this point it was getting tough to read so I turned off linewrap. You don't
have to do this to solve the problem but it made it easier for me to visualize.

```:set nowrap```

Now each Guard's data is all grouped together, and all the data from each
individual shift is on a line. Keep in mind the result we're looking for: the
total amount of time slept by each guard.

Here's how I approached this.

Each [fall asleep/wakes up] pair needs to have the minutes subtracted to find
sleeping time (hour doesn't matter since it's all between 00:00 and 00:59).
Then for each guard we must sum all of the subtraction results. So I'm going
to separate each Guard into a paragraph again, get rid of the unnecessary "
begins shift" text, split the falls asleep/wakes up data all to new lines, and
perform our arithmetic.

To split Guards into paragraphs I used a simple macro (stored in q) and
repeated it more times than necessary (`50@q` did the trick)

```f#l*NNokj+```

Now let's start pairing down the verbosity. I used a global command on every
line with "Guard" to take the whole [<time stamp> Guard starts shift] and turn
it into just the ID number. Whatever matches inside of \( and \) in the search
is accessible as \1 in the replace.

```:g/Guard/s@^.*\(#\d\+\).*shift@\1@```

Now I'll do something similar for the [falls asleep/wakes up] pairings. This
one is ugly because I left several pairs on one line and vim seems to be
inclined to find the largest possible selection matching a regex whereas my
brain is inclined to find the smallest.

```:g/./s#@.\{,15}:\(\d\+\). falls asleep@.\{,15}:\(\d\+\). wakes up# \2-\1#g```

Unfortunately I was sloppy and left on trailing @ signs so let's clean those
up.

```:g/@$/norm $x```

Easy enough! Now let's do some arithmetic.

```vip:s/ /\r/g<cr>{j"addvip:g/#/norm dd<cr>vip:g/./norm cc="<cr><cr>vip:s/\n/+<cr>cc="0<cr>kj"aPjojk+```

This fairly complicated macro will split every "word" on a line, save the
guard number in register a, delete every line with a # in it (the lines with
the guard id #), leaving only the arithmetic. It will then calculated the
result of each line and replace the expression with the result. Following that
, it puts everything on one line, adds it all together, replaces the
expression with the result, and puts the guard number (from register a) above
it. Now we just need to put the guard numbers on the same lines as the sleep
times and sort them numerically to find the guard that sleeps the most.

```:g/#/norm ddpkJ```

This will find the lines with guard ID numbers, delete the line, paste it
below the line that was below it originally, go up a line, and then merge the
lines (see :help J).

Let's sort!

```:sort n```

This leaves several anomolies. First, there are a bunch of empty lines at the
top, and second, each guard that did not sleep wasn't formatted as expect, so
they got sorted at the bottom. However, this does sort all the guards that did
sleep, and that's all we need. The reason we're doing this in Vim instead of
writing code is because it's disposable, it needs to be done one time, it doesn
't need to be the perfect solution, just a working one.

Ignoring the anomolies, this shows that the guard that slept the most was #
\<REDACTED\> with \<REDACTED\> minutes.
