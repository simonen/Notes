prints the 4th field of passwd file. F : - delimiter field 

``` bash
awk -F : '{ print $4}' /etc/passwd
```

# SED

Streaming Editor

displays the 5th line of a file
**$ sed -n 5p**  /etc/passwd

replaces all occurrences of old_name with new_name in a text file
**$ sed s/old_name/new_name/g** text_file

-i - saves changes to the file

deletes a line with a specific number
**$ sed -e '2d'** file

deletes a range of lines (1 to 3)
**$ sed -e '1, 3d'** file

deletes specific lines, separated with ';'
**$ sed -e '1d; 3d'** *file*

deletes line 2 and lines 4 to 10
**$ sed -e '2d; 4,10d'** file

