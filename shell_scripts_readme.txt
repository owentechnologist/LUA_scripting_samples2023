Three shell scripts are described below:
lualoader1.sh
timedroutlooploader.sh <number-of-slots> <number-of-entries>
randomread.sh <number-of-repetitions>

lualoader.sh should be executed one time before routlooploader.sh is ever executed.

The timedroutlooploader.sh takes 2 arguments
1)- the number of different redis slots into which you wish to write data
     (for each slot, it will write 1 large SortedSet and many small SortedSets)
2)- the number of entries to be written in each pass
     (this should not exceed 25000 as it will block redis for too long)

Running timedroutlooploader.sh 32 20000 against a redis database with 8 primary shards takes about 3 minutes and
results in the memory growing from ~98Mb to ~1.1GB (so about 1Gb of data is written)
Additionally - roughly 627 thousand keys are written into redis.

Running: routlooploader.sh 64 10000
takes about 3 minutes and writes about 500MB of data into 627 thousand keys
Executing routlooploader.sh 128 20000 took roughly 11 minutes and resulted in 2.5M keys and 4.17GB data
routlooploader.sh 1024 1000 took a while... and resulted in a very balanced 1 million keys and 800MB used

# The shell script lualoader.sh loads 4 LUA scripts and stores their SHA values in keys in redis
# The shell script timedroutlooploader.sh uses the SHA values to repeatedly load SortedSet data into redis
# timedroutlooploader also performs an aggregation against one of the SortedSets (it outputs the average value stored)
# The shell script randomread.sh loops for as many times as you tell it to - grabbing a random key and
 (if it is a SortedSet) -reporting the value of one member of that SortedSet

NB: THESE SCRIPTS ARE NOT DESIGNED FOR PERFORMANCE TESTING
In fact: The routlooploader makes several calls using redis-cli - among them is a call where a pause of 1 second occurs.

# The following are variables used in the two scripts:

bigzs <-- sha for script that loads many entries into a single SortedSet
varyzs <-- sha for script that loads varying numbers of entries into many SortedSets
avgzsv <-- sha for script that returns the average score for entries contained in a SortedSet
calcrouts <-- sha for script that returns the routing values (for entries used by that run of a LUA script)

Why have these two shell Scripts?

LUA scripts are convenient, but work best when combined with some kind of controller program/script.

Here it is suggested to use a shell script as a wrapper to multiple calls of a LUA script.

As demonstrated by the two sample shell scripts:
It makes sense to first load the interesting LUA scripts into redis so they can be more simply invoked:

SCRIPT LOAD "SCRIPT GOES HERE"
loadLUA.sh loads 4 scripts into redis and stores their SHA values as Strings in redis for later use

Once SHA values are stored, you can run the scripts using the SHA value returned:
EVALSHA "2ac113c2cb2197012e5b21129f9cc2ba1104c714"

-- remember to include any arguments to the script as part of the invocation:
  EVALSHA "2ac113c2cb2197012e5b21129f9cc2ba1104c714" 1 z:routvalues 8
