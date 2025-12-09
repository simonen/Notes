
**cat** - dumps the contents of a file on screen
**tac** - same as cat, in reverse

#### less - pager format
**$ ps aux | less** - passes the processes output to less
**$ /text** - searches for **text** within the less page
**$ less** *text_file*

**more** - similar to less, present in all Linux Distributions

#### head

**$ head** *text_file* - shows the first 10 lines of a file by default
$ **head -n 10** *text_file* - shows the first 10 lines of a file specifically
**$ head -10** *text_file* - same
**$ head -n 5** /etc/passwd | **tail -n 1** to show only line number 5 of the /etc/passwd file.

#### tail 

**$ tail** *text_file* - displays the last 10 line of a line by default
$ **tail -n 10** *text_file* - shows the last 10 lines of a file specifically
**$ tail -10** *text_file* - same
**$ tail -f** *filename* - refreshes the output for real time data

#### cut

Displays a specified column (field)

-d - delimiter
-f - field (column)

**$ cut -d : -f 1** /etc/passwd

#### sort

Sorts text in byte order (using the ASCII table)

**-n** - sorts in ascending numeric order
**-rn** - descending numeric order
**-t** - specifies field delimiter
**-k** # - sorts by column

**$ sort -k 3 -t :** /etc/passwd

#### wc 

word count - displays:
number of lines    number of words   number of characters

**$ps aux | wc**

Read a text from a .gz archive
$ **zcat** *ACHIVE.GZ*

