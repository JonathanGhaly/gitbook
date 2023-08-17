# milli cpu

It's best to see _millicores_ as a way to express fractions, _x_ millicores correspond to the fraction x/1000 (e.g. 250millicores = 250/1000 = 1/4).\
The value 1 represent the complete usage of 1 core (or hardware thread if hyperthreading or any other SMT is enabled).

So 100mcpu means the process is using 1/10th of a single CPU time. This means that it is using 1 second out of 10, or 100ms out of a second or 10us out of 100.\
Just take any unit of time, divide it into ten parts, the process is running only for one of them.\
Of course, if you take a too short interval (say, 1us), the overhead of the scheduler becomes non-negligeable but that's not important.

If the value is above 1, then the process is using more than one CPU. A value of 2300mcpu means that out of, say, 10 seconds, the process is running for... 23!\
This is used to mean that the process is using 2 whole CPUs and a 3/10 of a third one.\
This may sound weird but it's no different to saying: "I work out 3.5 times a week" to mean that "I work out 7 days every 2 weeks".

Remember: millicores represent a _fraction of CPU time_ not of CPU number. So 2300mcpu is 230% the time of a single CPU.
