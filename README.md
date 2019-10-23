# bdl-lib.sh

## NAME
bdl-lib.sh -- a Bash Debug library scriptlet

## USAGE
Source the scriptlet into a script
```
. bdl-lib.sh
```

## ENVIRONMENT VARIABLES
 bdl_dst
 bdl_call_depth
 bdl_call_stack_view

### bdl_dst
Defines the default destination which maybe a
file descriptor 1..9 or a file name. If bdl_dst=0
then output is disabled when no destination
parameter is passed.

#### Examples
 bdl_dst=0            # Default output disabled
 bdl_dst=1            # Default output to file descriptor 1, i.e. STDOUT
 bdl_dst=~/git/o      # Default output to file ~/git/o

### bdl_call_depth
Defines the call depth for retriving the line number
information from the Bash call stack. Set with the
first parameter to bdl_push and restored with bdl_pop.

### bdl_call_stack_view
Internal environment and when 't' displays the Bash
call stack. Useful for determining what an appropriate
value for the first parameter to bdl_push.

## FUNCTIONS
 bdl
 bdl_push
 bdl_pop

### bdl
Outputs its parameters to a file descriptor or a
or a file destination. The parameters are preceeded
with the filename and line number of bdl statement.

If the number of parameters == 0 then just the file
name and line number are printed to $bdl_dst.

If the number of parameters == 1 then the file
name and line number are printed followed by a space
and then the parameter to $bdl_dst.

If number of parameters > 1 then the first parameter
is the destination and the other parameters are written
to the destination. The destination rules are the same
as those for bdl_dst.

### bdl_push
Pushes the current bdl state (bdl_dst, bdl_call_depth,
bdl_call_view_state) onto an internal stack.

If the number of parameters == 0 the state is pushed.

If the number of parameters == 1 the state is pushed and
then the bdl_call_depth set to $1.

If the number of parameters == 2 the state is pushed and
then the bdl_call_depth is set to $1 and $2 is a series
of text lines, typically the content of a test. These
lines are processed so line number information is available
when bdl is used in a test_expect_xxx. See t/test-bdl-lib.sh
for examples.

### bdl_pop
Pops the previous saved state from a bdl_push, no parameters.

## Testing
**Currently doesn't work**

There are some tests in `t/` but those rely on code in `git` which
is where I originally developed this scriptlet.

## Examples
```
$ cat -n examples.sh
     1	#!/usr/bin/env bash
     2	# Examples using bdl-lib.sh
     3
     4	# Source bdl-lib.sh
     5	. bdl-lib.sh
     6
     7	# These output to the default bdl_dst=1
     8	bdl
     9	bdl "hi"
    10	bdl 1 "hi"
    11
    12	echo -n >bdl_out.txt # Empty bdl_out.txt
    13
    14	# Output to a file as parameter
    15	bdl bdl_out.txt "hi to bdl_out.txt"
    16	cat bdl_out.txt
    17
    18	echo -n >bdl_out.txt # Empty bdl_out.txt
    19
    20	# Output to a file using bdl_dst
    21	bdl_dst=bdl_out.txt
    22	bdl "hi to bdl_out.txt"
    23	cat bdl_out.txt
    24	bdl_dst=1
    25
    26	echo -n >bdl_out.txt # Empty bdl_out.txt
    27
    28	# Push the current state and change bdl_dst to FD 5
    29	bdl_push
    30	bdl_dst=5
    31	exec 5>bdl_out.txt # Redirect 5 to bdl_out.txt
    32	bdl
    33	bdl "This and previous line via bdl_dst=5"
    34	bdl 5 "hi via 5 directly"
    35	cat bdl_out.txt
    36	bdl_pop
    37
    38	echo -n >bdl_out.txt # Empty bdl_out.txt
    39
    40	# No printing when parameter it 0
    41	bdl_dst=1
    42	bdl 0 "not printed"
    43	
    44	echo -n >bdl_out.txt # Empty bdl_out.txt
    45
    46	# No printing when bdl_dst=0 but is printed if direct
    47	bdl_dst=0
    48	bdl
    49	bdl bdl_out.txt "printed to" "bdl_out.txt"
    50	bdl 1 "printed to 1 directly"
    51	bdl "not printed"
    52	cat bdl_out.txt
    53
    54	echo -n >bdl_out.txt # Empty bdl_out.txt
    55
    56	# This prints a "0" since there is only one parameter
    57	bdl_dst=1
    58	bdl 0
    59
    60	# Cleanup
    61	rm bdl_out.txt
```

### This is the output of ./examples.sh
```
$ ./examples.sh
examples.sh:8:
examples.sh:9: hi
examples.sh:10: hi
examples.sh:15: hi to bdl_out.txt
examples.sh:22: hi to bdl_out.txt
examples.sh:32:
examples.sh:33: This and previous line via bdl_dst=5
examples.sh:34: hi via 5 directly
examples.sh:50: printed to 1 directly
examples.sh:49: printed to bdl_out.txt
examples.sh:58: 0
```
