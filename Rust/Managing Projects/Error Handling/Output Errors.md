# Output Errors

In most terminals, there are two kinds of output:

- *standard output* (`stdout`) for general information
- *standard error* (`stderr`) for error messages

This distinction enables users to choose to direct the successful output of a program to a file but still print error messages to the screen

`println!` macro is only capable of printing to standard output

`eprintln!` macro that prints to the standard error stream