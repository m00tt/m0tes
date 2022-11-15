# Some basic command using pipeline

1. Write a command pipeline that identifies only the process with higher PPID
```bash
ps -eo uname,pid,ppid,args | sort -nk3 | tail -1
```

2. Get the total number of files in the directory /usr/bin and /var
```bash
expr $(ls -Rl /usr/bin | grep -v ^d | wc -l) + $(ls -Rl /var | grep -v ^d | wc -l)
```

3. Getting the sum of file occupations in the directory /usr/bin and /var
```bash
expr $(sudo du -sb /usr/bin | cut -f1) + $(sudo du -sb /var | cut -f1)
```

4. Move all files starting by "m" from one directory (and subdirectories) to another
```bash
find /home/user/directory1 -type f -name m* | xargs -I {} mv {} /home/user/directory2
```

5. Copy files from one directory to another without changing permissions
```bash
cp -p file1 directory1
```

6. Finding the "biggest" file on the system
```bash
find / -type f | xargs du | sort -nr | head -1
```
or
```bash
find / -type f | xargs du | sort -n | tail -1
```

7. Archive files larger than 5k and smaller than 100k
```bash
find / -type f -size +5k -size -100k | xargs tar -cvf archive.tar
```

8. Find all executables over 5k and show only the 5 largest.
```bash
find / -type f -size +5k -executable 2>/dev/null | xargs du -k | sort -nk1 | tail -5 | xargs du -kh 2>/dev/null
```

9. Archive files that have been changed in the last week
```bash
find / -type f -mtime -7 | xargs tar -cvf LastWeek.tar
```

10. Find the process that uses more ram memory
```bash
ps -aux | sort -nk4 | tail -1
```

11. Calculate for each user the number of files changed in the last month
```bash
#!/bin/sh

item=0;
for user in $(cut /etc/passwd -d : -f 3 | sort -n | uniq);
do
item=$(find / -user $user -type f -mtime -30 2>/dev/null | wc -l);
echo"User $user modified $item files in last month";
done
```

12. Find all files with odd number of lines (so they are text files)
```bash
find /home/user/es/ -type f | xargs wc -l | tr -s ' ' | awk {'if(($1%2)!=0) print $2'}
```

13. Find all symbolic links in the system
```bash
find / 2>/dev/null | xargs ls -l 2>/dev/null | grep ^l
```

14. Count the occurrence of words within a file without considering punctuation and put them in
descending order
```bash
grep -oE '[[:alpha:]]+' occurrence | sort | uniq -c | sort -nr
```

15. Take the greatest occurrence within a file
```bash
grep -oE '[[:alpha:]]+' occurrence | sort | uniq -c | sort -nr
```
or
```bash
grep -oE '[[:alpha:]]+' occurrence | sort | uniq -c | sort -nr | sed '2d'
```
or
```bash
grep -oE '[[:alpha:]]+' occorrenze | sort | uniq -c | sort -n | tail -1
```

16. Print words with higher occurrence of files that start with "s" and have extension ".h" that are found in home/user/include
```bash
find /home/user/include -type f -name s*.h | xargs grep -oE '[[:alpha:]]+' | cut -d : -f 2 | sort | uniq -c | sort -nr
```

17. Print files contained in home/user/include which start for "s" and have extension ".h" and wich containing the word "the" inside them
```bash
find /home/user/include -type f -name s*.h | xargs grep -l 'the' | cut -d / -f 5
```

18. Calculate how many times the words DEFINE and INCLUDE appear in the total of files ".c" and ".h"
```bash
find / -name *.[ch] 2>/dev/null | xargs cat | grep -oe define -oe include | sort | uniq -c
```

19. Find the user who has the most files in the system
```bash
for user in $(cut -d: -f1 /etc/passwd); do find / -type f -user $user 2>/dev/null | echo -e $user:'\t' $(wc -l); done | sort -rn -k 2 | head -n 1
```

20. For each user print the sum of file size (made with xargs du)
```bash
for u in $(cut /etc/passwd -d ':' -f 1); do size=$(find / -user $u -type f 2>/dev/null | xargs du -bc 2>/dev/null | tail -1 | awk '{print $1}'); echo "$u, $size"; done
```
