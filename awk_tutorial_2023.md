# Awk tutorial commands

## Pre-defined variables

Basic syntax: `pattern {action}` where the action is executed if **pattern** evaluates to a non-zero value

- **RS** is record separator, by default is new line so every line is considered a record
- **NR** is an internal variable that contains the record number (line number by default, **1 is header**)
- **FS** defines the input field separator. After a record is read, it is separated in fields by the given **FS** *[white space by default]*
  - After processing, the output is also separated in fields, this time using the **OFS** *(white space by default too)*
- The number of fields (**NF**) is defined according to the specified field separator (**FS**)

```sh
    # example file
CREDITS,EXPDATE,USER,GROUPS
99,01 jun 2018,sylvain,team:::admin
52,01    dec   2018,sonia,team
52,01    dec   2018,sonia,team
25,01    jan   2019,sonia,team
10,01 jan 2019,sylvain,team:::admin
8,12    jun   2018,öle,team:support
        


17,05 apr 2019,abhishek,guest
```

## Basic commands

```sh
    # print all, `[non-zero constant] {print}`
awk '2 {print}' example.txt

    # print used by default
awk 3 example.txt

    # prints nothing (zero value)
awk 0 example.txt

    # removes header
awk 'NR!=1 {print}' example.txt
awk 'NR>1 {print}' example.txt


    # removes white-space, no fields defined
awk 'NF' example.txt

    # weirdly 'RS=""' requires a 'newline + whitespace' as record separator
awk '1' RS="" example.txt
```

Logicals in `awk` are:

- **&& for and**
- **|| for or**
- **! for not**

```sh
    # print records above 1 and below 4
awk 'NR>1 && NR < 4' example.txt
    
    # print records that are > 1 or different from 4
awk 'NR>1 || NR != 4' example.txt
```

## Using fields

Printing files must be specified inside an action field `{action $X, $X`}

```sh
    # prints fields 1 and 3
awk '{print $1, $3}' example.txt
awk '{print $1,$3}' example.txt
    # define ',' as the separator
awk '{print $1, $3}' FS=, OFS=, example.txt
    # output with tab '\t'
awk '{print $1, $3}' FS=, OFS='\t' example.txt
    # $0 represents whole record
awk '{print $0, $3}' FS=, OFS='\t' example.txt
    # remove blank line without fields
awk ' NF {print $1, $3}' FS=, OFS='\t' example.txt
```

The option `BEGIN {awk-command}` specifies commands to run before the whole script. In the previous examples the field separators where specified in the command line, outside of ' ', we can also do it inside `awk` using `BEGIN`.

The `END {awk-command}` rule is similar, although it is executed after the main `awk` script.

```sh
awk 'BEGIN {FS=OFS=","} NF {print $1, $3}' example.txt
awk 'BEGIN {FS=OFS=","} NF {print $1, $3}' example.txt

awk '{print $1, $3} END {print "done"}' example.txt
```

### Field separator magic

Re-specifying a field, makes `awk` apply any changes to the output field separator before printing.

- `$1=$1` also evaluates an expression, so if the value is false for an empty line, it does not return anything and empty lines are not printed.

```sh
awk '{ print }' FS=, OFS=';' example.txt
awk '$1=$1 { print }' FS=, OFS=';' example.txt

    # print empty lines, interpreted as $1 = ($1 || 1)
awk '$1=$1 || 1 { print }' FS=, OFS=';' example.txt
    # correct form
awk '($1=$1) || 1 { print }' FS=, OFS=';' example.txt

```

Tweaking the output by defining OFS and ORS (record separator). **Might get really confusing, with better options available for formatting** (see printf).

```sh
    # OFS is " " by default, so it eliminates extra spaces
awk '1' example.txt
awk '$1=$1 {print}' example.txt

# ----- changing record separator

awk 'BEGIN {FS=","} NR>1 {print $3}' ORS=" " example.txt
    # has white space included, eliminates header
awk 'BEGIN {FS=","} NR>1 {print $3}' ORS="+" example.txt
    # ignores when white space follows beginning of record
awk 'BEGIN {FS=","} /[^[:space:]]/ && NR>1 {print $3}' ORS="+" example.txt
    # gets rid of separator at the end of line
awk 'BEGIN {FS=","} /[^[:space:]]/ && NR>1 {print SEP $3; SEP="+"}' ORS="" example.txt
```

## Operations using awk

1. We define a variable 'SUM' and assign a value, starts in 0 and adds the value of field #1 per record.
2. To count the number of non-empty lines, I match the RegEx `/./` or use `NF` and add 1 to the 'COUNT' for every record meeting the condition.

```sh
# ------- 1. SUM
awk '{ SUM=SUM+$1 } END { print SUM }' FS=, OFS=, example.txt
    # alternative syntax
awk '{ SUM+=$1 } END { print SUM }' FS=, OFS=, example.txt

# ------- 2. Count
awk '/./ { COUNT+=1 } END { print COUNT }' example.txt
    # ignores whitespace only lines
awk 'NF { n+=1 } END { print n }' example.txt
    # omits first line or header, variable syntax
awk 'NF && NR>1 { n+=1 } END { print n }' example.txt
awk '+$1 { COUNT+=1 } END { print COUNT }' example.txt
```

## Arrays

In the form `array_name[index] = value` where **array_name** is the name of the declared array, **index** is the array index, and **value** is any value declared to the given element of the array

In the following examples, the name of the array is 'CREDITS|ARRAY', the index are the strings contained in field #3 and the value the contents of field #1

```sh
awk '+$1 { CREDITS[$3]+=$1 }
     END { for (NAME in CREDITS) print NAME, CREDITS[NAME] }' FS=, example.txt
    # change name of array, print credits for 'sylvain'
awk '+$1 { ARRAY[$3]+=$1 }
     END { print ARRAY["sylvain"]}' FS=, example.txt
```

Declare an array whose index is the complete record ($0), the operator `++` adds 1 only after evaluation of the englobing expression. `awk` evaluates the expression `array[$0]` which is zero by default, executes the command, and then adds 1 to the value for the given index.

Thus, only a repeated line has a non-zero value when evaluated (before the operator) and is printed.

```sh
awk 'array[$0]++' example.txt

    # prints non-zero values, so non-repeated lines
awk '!array[$0]++' example.txt
    # remove only whitespace line too
awk 'NF && !array[$0]++' example.txt
    # no header
awk 'NF && !array[$0]++ && NR>1' example.txt
```

## Field formatting using `printf`

The `printf` function takes a format as the first argument, containing both plain text that will be output verbatim and wildcards used to format different section of the output. The wildcards are identified by the % character. The most common being `%s` (for string formatting), `%d` (for integer numbers formatting) and `%f` (for floating point number formatting).

The `printf` command does not use the ORS or OFS variables for output, the separators or such must be explicitly called in the arguments

In the example below, we explicitly add a " " after the string (%s), which is later replaced by the specified fields after the comma.

```sh
    # single replacement, no header (+$1)
awk '+$1 { printf("%s ",  $3) }' FS=, example.txt; echo
    # two replacement of strings by fields
awk '+$1 { printf("%s %s ",  $3, $4) }' FS=, example.txt; echo
    # add newline (\n)
awk '+$1 { printf("%s\n",  $4) }' FS=, example.txt; echo
```

The function also allows tabular output if you specify the width of the string/double as `%[width_num]s` or `%[width_num]d` as its shown below

```sh
    # no format
awk '+$1 { printf("%s | %s",  $3, $1) }' FS=, example.txt; echo
    # define width 
awk '+$1 { printf("%10s | %4s",  $3, $1) }' FS=, example.txt; echo
    # add newline
awk '+$1 { printf("%10s | %4s\n",  $3, $1) }' FS=, example.txt; echo
    # pad on the right instead of left for strings (negative width)
awk '+$1 { printf("%-10s | %4s\n",  $3, $1) }' FS=, example.txt; echo
```

For floating numbers (decimals) you may want to specify always the width and the precision after the decimal point. Below, AVG is defined as a floating number, that is latter replaced by the result of the operation SUM/NUM.

```sh
awk '+$1 { SUM+=$1; NUM+=1 } END { printf("AVG=%f\n",SUM/NUM); }' FS=, example.txt
    # add width
awk '+$1 { SUM+=$1; NUM+=1 } END { printf("AVG = %30f\n",SUM/NUM); }' FS=, example.txt
    # add precision too
awk '+$1 { SUM+=$1; NUM+=1 } END { printf("AVG = %12.2f\n",SUM/NUM); }' FS=, example.txt
awk 'BEGIN {FS=","} +$1 { SUM+=$1; NUM+=1 } END { printf("AVG = %8.3f\n",SUM/NUM); }' example.txt

```

## Quick operations to format strings

Quickly reformat to UPPER or lower case

```sh
awk '1' example.txt
awk 'BEGIN {FS=","} +$2 {print toupper($0); }' example.txt
awk 'BEGIN {FS=","} {print tolower($0); }' example.txt
```

The `substr(s, a, b)` returns **b** number of chars from string **s**, starting at position **a**. The parameter **s** is optional, in which case it means up to the end of the string.

So, `substr($3,1,1)` will evaluate to the first character of `$3`, and `substr($3,2)` to the remaining ones.

```sh
awk '{ $3 = toupper(substr($3,1,1)) substr($3,2) } $3' FS=, OFS=, example.txt
```

We can also apply regular expression replacement in a specific field.

```sh
awk 'BEGIN {FS=","} +$1 { gsub(/ +/, "-", $2); print }' example.txt
```
