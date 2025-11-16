---
layout: default
title: lzrdtrak - Cool interface
---

# Background

We all love vintage obnoxious interfaces and we do love complaining how the internet was better when there was no Javascript (wink at my good friend F  ;) ), but what do we do about it? NOTHING.

I am here to break this pattern and wake us up from a React nightmare (S please don't unfriend me). 

We're going ASCII baby (⌐■_■) 

# The plan

Here's a general idea:

1) Write a C program that queries SQLite and generates HTML with ASCII graphs and content (no components, no CSS, no pictures, NOTHING )

2) Serve it with a simple web server

3) Add the meta refresh tag for auto-reload

4) Profit? 

# Implementation

The code structure would be something like this

```
lzrdtrak/
├── main.c           # Entry point
├── database.c       # Database operations
├── database.h       # Database function declarations
├── display.c        # HTML/ASCII generation
├── display.h        # Display function declarations
└── gecko.db
```


So we need header files for all the intermediary files like `database.c` or `display.c`. 

let's look first at the `database.h` file

We will start it with:
```
#ifndef DATABASE_H
#define DATABASE_H
//.... some code
#endif
```
Why like this? There is a huge chance that we will include these definitions in multiple files in our code. In effect, we would be defining DATABASE_H multiple times in the code.
And that would throw a compiler error.
Therefore, we need to say something like "define it ONLY if that definition doesn't exist yet, otherwise skip".

Ok good. So what is going inside this definition?


First we are going to define a custom data type called *Reading*:
```
typedef struct {
    float temperature;
    float humidity;
} Reading;
```
We can think of `struct` as container that groups related variables together. 
and `typedef` is creating a type alias, so letting us use just `Reading` instead of `struct Reading` every time. 

And finally, the last part of the header file is:

```
Reading get_latest_reading();
```

Which is a **function prototype** - some sort of saying that the funciton exists, returns type Reading and doesn't take any parameters. 


Now display.h will contain similar stuff, we won't dwell on it. 

Let's go to the code. Let's start with the easiest bit, `main.c`

```
#include "database.h"
#include "display.h"

int main() {
    Reading r = get_latest_reading();
    generate_html(r);
    return 0;
}
```

ahh we do love short files don't we. 

Lets move on to the database reading file

In that file we load stuff we need:



```
#include <sqlite3.h>
#include <stdio.h>
#include "database.h"
```

Then we define our prefiously promised function:

```
Reading get_latest_reading(){
\\... something
}
```

Declare variables:

```
Reading r = {0, 0};
sqlite3 *db;
sqlite3_stmt *stmt;
int rc;
```
Hold on, what's that stmt doing there? type `sqlite3_stmt`?? 

Well, it's a STATEMENT. An SQL STATEMENT type, a SQL query ready to be executed. 

We do the usual checks if the database wants to talk to us:

```
rc = sqlite3_open("gecko.db", &db);
if (rc != SQLITE_OK) {        
        fprintf(stderr, "Cannot open database\n");
        return r;
}
```

And then we define the string that will store the SQL query:

```
char *sql = "SELECT temperature, humidity FROM readings ORDER BY timestamp DESC LIMIT 1;";
```

And then we make this string into a SQL statement by pasing it into a `sqlite3_prepare_v2` function:

```
rc = sqlite3_prepare_v2(db, sql, -1, &stmt, 0);
```

BTW, the explanation of the function signature explanation is below:
- *db* - which database
- *sql* - which string we will be passing to be our SQL query
- *-1* - length of the SQL string we're passing; in this case we say it's null-terminated - not sure what that means, probably something like "ends with zeros"
- *&stmt* pointerto where to store the statement - we used this pointer defined at the begining
- *0* - we don't need the tail, so NULL

OK, almost there. Now we have some weird if statement:

```
if (sqlite3_step(stmt) == SQLITE_ROW) {
        \\ some code
       
    }
```

I think this executes the query on each row, but if the row is something else that SQLITE_ROW then we pass. 
What else can it be, you'd ask?

These are the options:

- SQLITE_ROW : normal row of data
- SQLITE_DONE : no more rows :(
- SQLITE_ERROR : something went wrong

Now once we get the row, we get it's data:

```
r.temperature = sqlite3_column_double(stmt,0);
r.humidity = sqlite3_column_double(stmt,1);
```

0th column is temperature, 1rst column is humidity. Simple enough

And finally...

We need to prevent memory leaks by freeing up the memory. 

First, we tell C that we are done with that SQL statement `sqlite3_finalise(stmt);`

so it can clean up that part, and then we say that we are done with database connection: `sqlite3_close(db);`
   
And finally, we return our struct with data `return r;`

# Interface

Ok ok but we were promised a cool interface, where is that??

Look, I am a newbie, I can't keep up with all the cool ideas while I am also learning C, have some mercy. 

Your cool interface is coming right now.

We have previously defined `display.h` header files so we can now define actual C file for it. 

This file will generate the html.

```
#include <stdio.h>
#include "display.h"

void generate_html(Reading r) {

    printf("Content-Type: text/html\n\n")
    printf("<!DOCTYPE html>\n");
    printf("<html>\n<head>\n");
    printf("<meta http-equiv=\"refresh\" content\"1\">\n");
    printf("<title>LZRDTRAK.EXE</title>\n");
    printf("<style>body { background: black; color: lime; font-family: monospace; padding: 20px; }</style>\n");
    printf("</head>\n<body>\n");
    printf("<pre>\n");
    
    printf("╔════════════════════════════════════════╗\n");
    printf("║         LZRDTRAK.EXE v1.0             ║\n");
    printf("║    Leopard Gecko Monitoring System    ║\n");
    printf("╚════════════════════════════════════════╝\n\n");
    
    printf("Temperature:  ");
    int temp_bars = (int)((r.temperature - 20) / 10.0 * 20);
    for (int i = 0; i < temp_bars; i++) printf("█");
    for (int i = temp_bars; i < 20; i++) printf("░");
    printf(" %.1f°C\n", r.temperature);
    
    // Humidity bar
    printf("Humidity:     ");
    int hum_bars = (int)(r.humidity / 100.0 * 20);
    for (int i = 0; i < hum_bars; i++) printf("█");
    for (int i = hum_bars; i < 20; i++) printf("░");
    printf(" %.1f%%\n\n", r.humidity);
    
    printf("</pre>\n");
    printf("</body>\n</html>\n");

}

```

This ispretty self explanatory if you understand basic HTML, maybe I'll get to some more detail later.

Now lets serve this baby

# Server

Because we are going with a simplest setup possible, we will be using netcat









































[back](./)
