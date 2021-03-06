LibPaxos2
============

Intro
---------

LibPaxos2 is a complete rewrite of [LibPaxos](http://libpaxos.sourceforge.net) on top of [LibEvent 1.4](http://monkey.org/~provos/libevent/)

Although this code has been extensively tested and benchmarked, it's comes as-it-is, it may contain bugs and should not be used where true reliability/fault tolerance is required.

For more details and informations on the performances of this library, refer to:

    @mastersthesis{paxos-made-code,
		Title = {Paxos Made Code: Implementing a high throughput Atomic Broadcast},
		Year = {2009}
		Author = {Marco Primi},
		Keywords = {paxos, atomic-broadcast},
		School = {University of Lugano},
	}


Paxos Essentials
-------------------

LibPaxos2 supplies the Atomic Broadcast primitive to a set of application processes.

A Paxos network is composed of the following actors:

 - Clients: application processes that want to send (ABroadcast) and/or receive (ADeliver) the Atomic Broadcast messages.
   A client only interested in sending can do so by initializing a submit_handle.
   A client also interested in receiving values must start a Paxos Learner internally.
   Refer to tests/benchmark_client.c for an example client that sends/receives.

 - Paxos Proposers: at least one proposer must be active in the network. Refer to tests/example_proposer.c for an example.
   Each proposer must start with a different identification number.
   Among the Proposers, a single one is chosen by the Leader Election service to be Coordinator.

 - Paxos Acceptors: number of acceptors is pre-determined and cannot change during execution (some may temporarily fail of course). 
   Like Proposers, each Acceptor must have it's unique identification number between 0 and N-1 (where N is the number of acceptors).
   How acceptors use their stable storage is probably the single most influential factor on performances. See the config and [3] for more details.
   Refer to tests/example_acceptor.c for an example.

 - Leader Election Oracle: this component is responsible of choosing a Leader among the Proposer, replace it in case of failure and so on.
   All proposers periodically send (UDP) heartbeats to the Oracle but nothing prevents from using more sophisticated failure detection strategies.
   The oracle in tests/example_oracle.c is not a really good example since it requires human intervention to pick a new leader (it ignores heartbeats) but it can be easily extended.


**Some practical details**

 - Each client process should initialize a single submit_handle.
 
 - Submitted values are (for the moment) sent to the proposer through UDP. Therefore they may be lost. The client must timeout on 
   it's own if the case and retry to submit them.
 
 - Because (i) submit is unreliable and (ii) proposer-leader may crash, the broadcast is NOT FIFO, not even respect to a single client.


Setup
---------

Before compiling LibPaxos, you need to have compiled both Berkeley DB and Libevent.

Download BDB source code from http://www.oracle.com/technology/software/products/berkeley-db/db/index.html

Extract it to some folder (i.e. ~/bdb) and follow the instructions included to compile, i.e., on unix-like: 

    cd ~/bdb && ../dist/configure && make

`~/bdb/build_unix/` should now contain the file `libdb.a`

Download Libevent source from http://monkey.org/~provos/libevent-1.4.12-stable.tar.gz

Extract it to some folder (i.e. ~/libevent) and follow the instructions included to compile, i.e., on unix-like: 

    cd ~/libevent && ./configure && make

`~/libevent/` should now contain the file `libevent.a`

Update Makefile.inc in the main libpaxos directory to reflect the location of the library and header files for the two libraries.
For example:

    BDB_DIR		= $(HOME)/bdb/build_unix
    LEV_DIR		= $(HOME)/libevent

Take a look at paxos_config.h to see if the default configuration fits your needs.
You can now compile libpaxos with:
    
    cd libpaxos && make

To see LibPaxos in action, you can launch an example script that starts all the required actors plus a simple client. (Requires the `xterm` command to open different windows!)

Edit the first variable of `scripts/local/run_example.sh` to reflect the placement of the LibPaxos tests/ directory on your filesystem. Example:

    PROJ_DIR="/Users/bridge/Desktop/libpaxos/trunk/libpaxos2/tests" 

To link your own program with LibPaxos, you can copy the compiler flags used for the test programs (i.e. look at `tests/Makefile`)


Feedback
-------------

Any suggestion, comment, or question is welcome via SourceForge [mailing list](https://lists.sourceforge.net/lists/listinfo/libpaxos-general)
or by contacting us directly.


