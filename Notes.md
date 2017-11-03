---
## 11/03/17: Input? fgets about it!

* In [Work 08](https://github.com/iwang2/08_stat/blob/master/stat.c), we used `st_mode` in `struct stat`, which returned a 6-digit octal as opposed to the 3-digit one we were expecting.
* To remove the first three digits, use the bitwise & operator:

```
Mode
_ _ _ _  _ _ _  _ _ _  _ _ _
         r w x  r w x  r w x
     &0b 1 1 1  1 1 1  1 1 1
```

**Command Line Arguments**

* `int main(int argc, char *argv[])`
    * `argc` - number of command line arguments
    * `argv` - array of command line arguments
    * Program name is considered the first command line argument, e.g., `./a.out`
* `scanf(<FORMAT STRING>, <VAR 1>, <VAR 2>, ...)` - `<stdio.h>`
    * Reads in data from stdin using the format string to determine types and puts the data in each variable.
    * Example:
    ```C
        int x; float f; double d;
        scanf("%d %f %lf", &x, &f, *d);
    ```
