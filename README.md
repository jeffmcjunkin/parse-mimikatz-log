# Overview

This started out as a quick and dirty tool to parse and organise large amounts of mimikatz output and I wrote it to be able to parse the usual format even when interleaved with other random output.

```
*     Username :
*     Domain :
*     Password :
```

Sometimes mimikatz will generate a huge amount of output; the classic example is running ```sekurlsa::logonpasswords``` on a lsass process dump from an Exchange server. Although there are a few tools to parse this, I wanted something that I could just ```cat`` or pipe in all files in any order and expect it to parse them, ignore any malformed output and offer enough flexibility to allow the credentials to be presented in different forms.

# Compatibility

It works with mimikatz 2.0

# Further work

I shall update it as needed and as new versions of mimikatz come out. It was not a significant piece of work but I hope it is useful to someone.

# Populating the database
One option is to use a command to cat the entire log file to screen and read from STDIN:

```bash
find ./ -name \*.log -exec cat {} \; | pml.py -d /tmp/passwords.db -i -

       .mMMMMMm.             MMm    M   WW   W   WW   RRRRR
      mMMMMMMMMMMM.           MM   MM    W   W   W    R   R
     /MMMM-    -MM.           MM   MM    W   W   W    R   R
    /MMM.    _  \/  ^         M M M M     W W W W     RRRR
    |M.    aRRr    /W|        M M M M     W W W W     R  R
    \/  .. ^^^   wWWW|        M  M  M      W   W      R   R
       /WW\.  .wWWWW/         M  M  M      W   W      R    R
       |WWWWWWWWWWW/
         .WWWWWW.           Quick & Dirty Mimikatz Log Parser
                        stuart.morgan@mwrinfosecurity.com | @ukstufus

Opening database: /tmp/passwords.db
Reading from STDIN
  Processing line 236676/236676 (100%)

        Sets of credentials: 949
    Unique 'user' usernames: 355
    Unique 'user' passwords: 278
```

An alternative is to find all log files and run it individually for each one:

```
find ./ -name \*.log -exec ./pml.py -d /tmp/passwords.db -i {} \;
```

# Extracting credentials
The database needs to be loaded into SQLite:

```
sqlite3 /tmp/tmpyIJeAn.20161011103852.mimikatz.db
SQLite version 3.14.1 2016-08-11 18:53:32
Enter ".help" for usage hints.
sqlite> 
```

All user credentials can be listed:

```
sqlite> select * from view_usercreds;
```

The credentials could be exported in a username:password format ('||' is the concatenation operator in SQLite and '.once' writes the output to a file):

```
sqlite> .once /tmp/credentials.txt
sqlite> select distinct username||':'||password from view_usercreds;
```

The credentials could also be exported in a DOMAIN\username:password format using the same technique as above:

```
sqlite> select distinct domain||'\'||username||':'||password from view_usercreds;
```

The number of users sharing the same password could be found by a query such as the below, which would display each password and the number of users with that password:

```
sqlite> select password,count(username) from view_usercreds group by password;
```
