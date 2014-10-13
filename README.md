A Little More Advanced Linux
============================

*Version 1.0, 20141013*

**Instructors**

Chris Collins, Jimmy Dorff, Drew Stinnett, et al

 
TOC HERE


<a name='lab0'></a>
## Lab 0: Creating a personal Linux VM

1. Using a web browser, go to *https://vm-manage.oit.duke.edu*
2. Login using your Duke NetId.
3. Create a new project for this class.
4. Select *Ubuntu 14 Basic* for the Server.

The vm-manage web page will tell you the name for your VM. The web site will also tell you the initial username and password. You should connect via ssh.

*Example:* `ssh bitnami@colab-sbx-87.oit.duke.edu`

5. Once logged in via ssh, enter the `passwd` command to set a unique password.

*Example:*

    passwd
    Changing password for bitnami.
    (current) UNIX password:
    Enter new UNIX password:
    Retype new UNIX password:


<a name='unit1'></a>
## Unit 1: Your Friend, the Shell

The unix shell is how you interact with the operating system - how you manipulate files and execute programs. You can automate shell commands. The shell can read a series of commands from a file rather than you typing it in. This is **scripting**.

You can add logic and flow control to scripts. Scripting is very powerful and should be used for all repetitive or complex tasks.

< GRUMPY CAT IMAGE > 

![grumpycat.jpg](images/grumpycat.jpg "Grumpy Cat")

To create a script, you must use a text editor. Shell scripts are plain text files. There are many text editors; nano, vi, emacs.

**nano**

Nano is a very simple, basic command line interface text editor. It is keyboard-based and controlled using control keys.

*Example:*

`CTRL-x` - exits the nano editor

Nano has a shortcut bar at the bottom of the terminal screen with the most-used CTRL commands.  On some systems, nano has syntax highlighting and limited mouse interaction.

**Shell Script Basics**
* Shell scripts must start with the line: `#!/bin/bash`
* Other than the first line, characters after a "#" are considered comments.
* The shell script file must have execute permissions. You can use chmod to add execute permissions:

*Example:*

`chmod +x myscript`

<a name='lab1'></a>
## Lab 1: Create and execute a shell script.

1. Recall the unix command `echo` and use it to print the string “Hello World”
2. Using a text editor, create a bash script that execute the above echo command
3. Make the script executable by adding execute permissions
4. Run the script. Note it is likely not in your $PATH, so you will need to call directly.

<a name='unit2'></a>
## Unit 2: Errors, they happen

In the most basic form, scripts are just a series of commands:

*Example:*

    #!/bin/bash
    mkdir /data/project3
    chgrp coolphds /data/project3
    chmod g+w /data/project3
 

This isn’t a very good shell script. Let’s make it better.  A **major** difference between a human entering commands and a shell script is error checking and error handling.

*Example:*

    #!/bin/bash
    if [[ -w /data ]]
    then
      mkdir /data/project3
    else
      echo “Count not write to /data”
      exit 1
    fi

**if statements**

"If statements take three forms:

Form 1:

    if condition
    then
      commands
    fi

Form 2: 

    if condition
    then
      commands
    else
      commands
    fi

Form 3:

    if condition
    then
      commands
    elif condition
    then
      commands
    fi

You can have multiple “else if” statements.

**conditions**

Conditions evaluate to true or false.

* `[[ “string1” = “string2” ]]` - true if the strings are equal
* `[[ “string1” != “string2” ]]` - true if the strings are not equal
* `[[ 3 -eq 3 ]]` - true if the numeric values are equal
* `[[ 3 -ne 3 ]]` - true if the numeric values are not equal
* `[[ -w /tmp ]]`-  true if I can write to the path /tmp
* `[[ -r /data/secret ]]` - true if I can read the file /data/secret
* `[[ -n "string" ]]` - true if the string is not empty
* `[[ -z "string" ]]` - true if the string is empty (z is for zero length)

Commands also return a true or false exit status. Commands that work return true; command that fail return false,

*Example:*

    $ if [[ -w $HOME ]]; then echo "worked"; else echo "failed"; fi
    worked
    $ if cp $HOME/.bashrc .bashrc-backup ; then echo "worked"; else echo "failed"; fi
    worked
    $ if [[ -w /home/bob/.bashrc ]]; then echo "worked"; else echo "failed"; fi
    failed
    $ if cp $HOME/.bashrc /home/bob/.bashrc ; then echo "worked"; else echo "failed"; fi
    cp: cannot create regular file ‘/home/bob/.bashrc’: Permission denied
    failed
 
**Alternative method for error checking**

The exit status of the most recent command is also available in the special value `$?`.  You may see scripts which use this for error checking:

    if [ $? -eq 0 ]

Since it is sometime difficult to know which commands exit status is in `$?`, I prefer the more direct method.

What is the deal with `if [ -w /tmp ]` versus `if [[ -w /tmp ]]`? The double bracket is a newer, faster form with more features. The single bracket still works for compatibility with older scripts.

Yet another way - conditions can use logical "and": `&&`

*Example:*

    if [[ -w /tmp && -n “$MYVAR” ]]

In an "and" statement, commands are executed left to right and stop if one fails. You might see:

    cp datafile $HOME && $HOME/bin/myprogram && cp $HOME/result /data/final

<a name='lab2'></a>
## Lab 2: Create a script with error checking

1. Using a text editor, create a new script.
2. Add error checking to all commands in the script
3. Try copying files to /tmp and to /home/bob
4. Run the commands true and false. To what value do they set `$?`

<a name='unit3'></a>
## Unit 3: Variables

When you login, you get a standard set of variables defined for you, called your environment.

* $RANDOM
* $SHELL
* $USER
* $HOME
* $PATH
* $PWD
* $TERM

You can define your own variables and use them both in scripts and at the command line. Note you do not use the "$" when assigning a value. Variables are case sensitive; upper case is not required but is common.

*Example:*

    #!/bin/bash
    mkdir /data/project3
    chgrp coolphds /data/project3
    chmod g+w /data/project3

...becomes:

    #!/bin/bash
    DATA="/data/project3"
    
    if mkdir $DATA
    then
     chgrp coolphds $DATA
     chmod g+w $DATA
    fi

You can enclose a variable in curly braces to clearly define the variable when using it. Some consider it good form to always use curly braces.

*Example:*

    $ FOO="Red"
    $ echo "${FOO}Bike ${FOO}Car"
    RedBike RedCar

You can store the results of a command in a variable:

*Example:*

    $ FOURDAYS=$( date -d "+4 days" +'%A' )
    $ echo ${FOURDAYS}
    Friday

You can pass values into your script via the environment or as arguments

*Example:*

    #!/bin/bash
    DATA="$1"
    if [[ -n "$DATA" ]] && mkdir $DATA
    then
     chmod g+w $DATA
    fi

...invoke as `myscript /data/project3`.

* The first argument is `$1` the second is `$2` and so on.
* The name of the script is `$0` and all the arguments are `$*`
* For complex arguments, use `getopts`

<a name='lab3'></a>
## Lab 3: Working with variables

1. Create a script that prints the day of the week 33 hours from now.
2. Change the script such that the number of hours is a variable passed into the program.
3. Add error checking if the user forgets to specify a value for hours

<a name='unit4'></a>
## Unit 4: Looping with for and while

Loops allow code to be repeatedly executed:

* for loops repeat over a set of things.
* while loops repeat based on a condition.

*Example:*

    #!/bin/bash
    for COUNT in 0 1 2 3 4 5 6 7 8 9
    do
     touch ${COUNT}.txt
    done
    # BASH can auto generate a sequence
    for COUNT in {10..20}
    do
     touch ${COUNT}.txt
    done
 

You may also come across the old fashioned `for COUNT in $( seq 1 10 )`.

**for loops**

`for` can loop over files:

*Example:*

    #!/bin/bash
    for TEXTFILE in *.txt
    do
     cp ${TEXTFILE} /backup
    done

`for` can loop over the output of a command:

*Example:*

    #!/bin/bash
    for TEXTFILE in $( find . -type f -name '*.txt' )
    do
     cp ${TEXTFILE} /backup
    done

What is the difference between the two above scripts?

**while loops**

While loops are a Common way to make an infinite loop to repeat a process until you kill it.

*Example:*

    #!/bin/bash
    while true
    do
     date >> logfile.txt
     sleep 5
    done

## Lab 4: Working with loops

1. Create a script that creates 1000 files, each containing a random number.

## Unit 5: Parsing text data

A core tenet of Unix design is storing data in plain text. Unix has very powerful tools to parse data. View each program *as a filter*: text data comes in, is processed in some way, then leaves as text data.

Modern formats expand “text” to include markup languages like html, LaTeX and XML and JSON

**Cut**

 - removes fields from lines of text

*Example:* 

Input file:

    $ cat input
    Bob 2947 1973
    Alice 8653 1989
    Jose 4325 2004
    Eric 2186 1999
    Adam 4310 2001
    Ernest 4309 2003

    $ cut -d " " -f 2 input
    2947
    8653
    4325
    2186
    4310
    4309

    $ cut -d " " -f 1,3 input
    Bob 1973
    Alice 1989
    Jose 2004
    Eric 1999
    Adam 2001
    Ernest 2003

    $ cut -d " " -f 1,3 input | sort
    Adam 2001
    Alice 1989
    Bob 1973
    Eric 1999
    Ernest 2003
    Jose 2004
 

We can add additional filters with pipes ("|"). 

*Example:*

* Match every line that begins with “E” in the file input
* Cut out the first and third fields where the fields are delimited by spaces
* Sort the remaining lines


    $ grep "^E" input | cut -d " " -f 1,3 | sort
    Eric 1999
    Ernest 2003
 

We can translate characters from one set to another set.

*Example:*

    $ cut -d " " -f 1,3 input | tr [:lower:] [:upper:]
    BOB 1973
    ALICE 1989
    JOSE 2004
    ERIC 1999
    ADAM 2001
    ERNEST 2003

Another important Unix principle is “there is always more than one way to do it”

*Example:*

    $ awk '{print toupper($1)" "$3}' input
    BOB 1973
    ALICE 1989
    JOSE 2004
    ERIC 1999
    ADAM 2001
    ERNEST 2003

    $ cut -d " " -f 1,3 input | sed 's/.*/U&/'
    BOB 1973
    ALICE 1989
    JOSE 2004
    ERIC 1999
    ADAM 2001
    ERNEST 2003

**Other Powerful Text Processing Tools**

* `sed` is the stream editor and uses much of the vi syntax 
* `awk` is a full programing language, but is often used for simple one line programs inside of shell scripts.
* `perl` and `python` are even more powerful programing languages and should be used for more advanced tasks than shell scripts.

---

![Creative Commons CC0 1.0 License](http://i.creativecommons.org/p/zero/1.0/88x31.png)

To the extent possible under law, [Linux@Duke](https://github.com/LinuxAtDuke) has waived all copyright and related or neighboring rights to *A Little More Advanced Linux*.  This work published from: United States.

