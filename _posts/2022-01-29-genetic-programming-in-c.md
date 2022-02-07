`pkg-config --list-all` to see the available C packages
`pkg-config --cflags *package name*` 
`pkg-config --libs *package name*` 

SDL and MinGW need to have same architecture compilation target, I've made a mistake of using x32 files while my compiler is x64.

If you run your binary and don't see a window try using prompt in administrator mode, worked for me.

https://youtu.be/iL--xqGgd0g?t=3167