# folg
Folg is a one-line interpretation language build specifically for adding email manipulation commands in email address. Used specifically in folgen app

### Email address available symbols

Because we limit language to a one line that follows end of email address, we're limited to next list of symbols:

1. lowercase Latin letters: `abcdefghijklmnopqrstuvwxyz`,
1. uppercase Latin letters: `ABCDEFGHIJKLMNOPQRSTUVWXYZ`,
1. digits: `0123456789`,
1. special characters: !#$%&'*+-/=?^_`{|}~,
1. dot: . (not first or last character or repeated unless quoted),
1. comments: () (are allowed within parentheses, e.g. (comment)john.smith@example.com).

### Predefined receiver

Folgen server should contain finite set of receiver addresses. You should be able to see them in control panel.
However, each Folgen server expected to receive next set of receiver addresses:

1. `followup`@domain - default followup email, registers email in control panel but doesn't affect process. 
There won't be any followup until you specify additional parameters
1. `default`@domain - executes default followup scenario specified in your control panel
1. `[1-48]h`@domain or `[1-48]hour`@domain- followup in x hours if no reply received
1. `[1-365]d`@domain - followup in x days if no reply received
1. `[1-12]m`@domain - followup in x month if no reply received
1. `every[1-x][h|d|m]`@domain - followup every x hours|days|minutes until reply received
1. `eval`@domain - evaluate folg expression after `+` sign and return result in email, useful for debugging without available parser

# LWFL - Lightweight followup language

Main purpose of LWFL is to tag some predefined conditions for email. It doesn't have any expressions or computations, 
rather it's a instructions to lookup specific routines and execute following scripts.

### 2.1 Operators

#### 2.1.1 `+` operator

`+` operator marks start of LWFL logic, it's logic changing depending on next variable or operator

##### 2.1.1.1 `+tag` operator

`+tag` operator specifies that this email should be eligible to specified condition. 
e.g `+workhours` will enforce email to be followed up only during working hours, so it won't be sent at 1A.M, 
instead it'll wait for start of working hours(e.g 8A.M, specified in settings)

List of available tags for Folgen can be found [here](http://todo)

List of basic tags:

* `+containsDate` check email if it contains any date, identified by Machine Learning
* `+containsLink` check if email any link(except email)(may work poorly if email footer contains website link),
  to filter out links you can use script with same name
* `+notify` notify if reply received via all available notification channels
* `+workhours` will enforce email to be followed up only during working hours, so it won't be sent at 1A.M, 
               instead it'll wait for start of working hours(e.g 8A.M, specified in settings)
* TODO: many more will arrive in second revision
    
##### 2.1.1.2 `+script{var1|var2}`
`+script{var1|var2|..}` specifies that this email should execute specific script, general or predefined at control panel.
e.g `+toggle{'Project1'|20h}`. **   Warning: whitespaces are not allowed, use _ in strings instead**

List of available scripts for Folgen can be found [here](http://todo)

List o basic scripts:

* `+days{[day|..]` specify by which days email should be followed up
* `+!days{[day|..]}` specify by which days email should not be sent, send it in all the other days
* `+macro{'Macro1'[|var..]}` execute script that should return successful result before following up
* `+Macro1{[var|..]}` this is possible but may have issues in a long run, more keywords would be introduced in later versions
* `+#Macro1{[var|..]}` it'll enforce to use local script
* `+contains{['string'|..]}` check email if it replied message contains specific `string`. 
   It'll continue following up unless required value found. e.g `+contains{'about_trip'}`
   will send followup until `about_trip` string encountered. Dangerous function, better to consider next option
* `+containsAny{['string'|..]` same as above, but will stop after encountering any of strings
* (WIP, ver2)`+topic{'string'}` similar to `+contains`, but uses Machine learning to check if reply contains specific topic
* `+containsLink{'string'} checks if email contains link and `string` is in this link
* `+notify{'channel'}` notify via notification channel, should be set up before using or won't work otherwise
* TODO: many more will arrive in second revision


#### 2.2 Simplified interval system

LWFL uses simplified interval system. You can use only limited amount of interval metrics, 
but it should fulfill general requirements for followup.

##### 2.2.1 Available interval values

* `m` as minutes, e.g `30m` marks as a 30 minute interval
* `h` as hours, e.g `1h`, `24h`
* `d` as days, e.g `30d`, `7d`
* `w` as weeks, e.g `3w`, `15w`
* `p` or (WIP, ver2)`mon` as months, e.g `1p`, `6p`
* (?)`q` as quartal
* (WIP, ver2)`year` for years, e.g `1year`

You can combine all of them in scripts e.g `+toggle{'Project1'|1mon` 

### Examples

* Send followup every 6 hours - `6h@folgen.email`
* Send followup every 6 hours, but only on Wednesday `onlywed@folgen.email` or `6h+days{'wed'}@folgen.email`
* Send followup every 24 hours if logged less than 20 hours in toggle Project1 `24h+toggle{'Project1'|20h}@folgen.email`
* Send followup every day if no money arrived at Paypal from sender that has *Deutsche Bahn* within last 30 days `1d+paypal{'Deutsche_Bahn'|30d}@folgen.email`
* Same as above but we add amount of payment, it should be more than 100 eur `1d+paypal{'DeutscheBahn'|30d|'100EUR'}@folgen.email`

# Folg (WIP, ver2)

## Folg rules

### Command position

folg interpreter expects beginning of code after `+` symbol. `+` symbol indicates variation of email, receiver is still the same.

Following `+` symbols indicate new command(in case you need to finish one expression and start next one)

## Available language functions

### 3.1 Features
#### 3.1.1 Literal expressions
The types of literal expressions supported are strings, numeric values (int, real, hex), boolean and null. Strings are delimited by single quotes. To put a single quote itself in a string, use two single quote characters.

The following listing shows simple usage of literals. Typically they would not be used in isolation like this but rather as part of a more complex expression, for example using a literal on one side of a logical comparison operator.

* eval+`#{'Hello_world'}`@domain evaluates to `Hello world`
* eval+`#{6.0221415E+23}`@domain evaluates to `6.0221415E+23`
* eval+`#{true}`@domain evaluates to `true` value

#### 3.1.2 Boolean and relational operators

##### 3.1.2.1 `+=` as `>=` greater or equal to operator replacement
##### 3.1.2.2 `-=` as `<=` greater or equal to operator replacement

Because `<` and `>` are restricted symbols, we have to use analogous operators `-` and `+`


### 3.2 Operators

#### 3.2.1 `#` operator
##### 3.2.1.1 `#{}` expression execution

`#{}` operator serves as a pre-processing symbol, it computes everything that's inside of brackets and replaces it within command 

followup+`#{2+2}`@domain

**Why is it helpful?** In control panel you can specify variables that can be used later on in such computations e.g `#{Math.pi+2}` or,

##### 3.2.1.2 `#function{}` execute functions

##### 3.2.1.3 `#variable` insert variable


#### 3.2.2 `?` operator
##### 3.2.2.1 `#list?{filter}` filter operator


#### 3.2.2 `_` operator
##### 3.2.2.1 Replacement for spaces


### Examples:

`followup+stopif=#{$toggle{'Development'}.hours&gt20}}`

### TODO:
* [x] Literal expressions
* [ ] Boolean and relational operators
* [ ] Regular expressions
* [ ] Class expressions
* [ ] Accessing properties, arrays, lists, maps
* [ ] Method invocation
* [ ] Relational operators
* [ ] Assignment
* [ ] Calling constructors
* [ ] Bean references
* [ ] Array construction
* [ ] Inline lists
* [ ] Inline maps
* [ ] Ternary operator
* [ ] Variables
* [ ] User defined functions
* [ ] Collection projection
* [ ] Collection selection
* [ ] Templated expressions
