# "Saint John": what is writing to this log file?

## Problem description

**Scenario:** "Saint John": what is writing to this log file?

**Level:** Easy

**Type:** Fix

**Tags:**

**Description:** A developer created a testing program that is continuously writing to a log file ```/var/log/bad.log``` and filling up disk. You can check for example with ```tail -f /var/log/bad.log```.
This program is no longer needed. Find it and terminate it. Do not delete the log file.

Root (sudo) Access: True

**Test:** The log file size doesn't change (within a time interval bigger than the rate of change of the log file).

The "Check My Solution" button runs the script /home/admin/agent/check.sh, which you can see and execute.

**Time to Solve:** 10 minutes.

## Solution

When we connect to the VM, if we use the ```ls``` command we can see that there is a program named ```badlog.py``` containing the following python code:

```python
#! /usr/bin/python3
# test logging

import random
import time
from datetime import datetime

with open('/var/log/bad.log', 'w') as f:
    while True:
        r = random.randrange(2147483647)
        print(str(datetime.now()) + ' token: ' + str(r), file=f)
        f.flush()
        time.sleep(0.3)
```

The python code is writing logs in the file ```/var/log/bad.log```. We can see the logs by using the command ```tail -f /var/log/bad.log```:

```bash
2025-10-07 19:48:08.823177 token: 227570320
2025-10-07 19:48:09.123640 token: 273358610
2025-10-07 19:48:09.424121 token: 953510084
2025-10-07 19:48:09.724592 token: 525796999
2025-10-07 19:48:10.025069 token: 340903265
```

It is the program that runs as a process we want to shutdown. To find the process, we can use the command ```ps -aux | grep 'badlog'```. 

* ```ps``` means 'process status'
* ```a``` means that we want to select all the processes for all users
* ```u``` is used to display the list in a user-oriented format
* ```x``` show processes that are not attached to a terminal (like daemons or background services like ours) 

We are using ```grep``` to filter with the name of the python program.

In the output, we can find the process we want to shutdown :

```bash
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
admin        584  0.0  1.7  12508  8192 ?        S    19:47   0:00 /usr/bin/python3 /home/admin/badlog.py
```
Now we need to shutdown the process. The linux foundation has done a wonderful [article](https://www.linuxfoundation.org/blog/blog/classic-sysadmin-how-to-kill-a-process-from-the-command-line) on it to understand how to shutdown a process.

To shutdown the process, we use the command :

```bash
kill -9 584
```

Let's break down the command :

* ```kill```: Linux command to send a signal. In Linux, a signal is like a message sent to a process to tell it that something has happened or that it needs to do something. It’s a way for the kernel or another process to communicate with a running program. So the ```kill``` command is not necessarily used to kill a process technically;
* ```9```: It's the signal number for ```SIGKILL```. We tell the ```kill``` command to send the signal ```SIGKILL``` to immediately stop a process.

* ```584```: It's the PID or the Process ID of the process we want to shutdown