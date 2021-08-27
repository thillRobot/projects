# Using Bash 
this file contains various useful bash (born again shell) commands

## change file extensions to all lower case

```

find . -name '*.*' -exec sh -c '
  a=$(echo "$0" | sed -r "s/([^.]*)\$/\L\1/");
  [ "$a" != "$0" ] && mv "$0" "$a" ' {} \;

```