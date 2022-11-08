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

