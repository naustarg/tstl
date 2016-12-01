TSTL: the Template Scripting Testing Language
===========================================

TSTL is a "little" language that makes it easy to test software.  This
implementation targets Python.  There is also a (beta) Java version,
at https://github.com/flipturnapps/TSTL-Java.

TSTL produces a simple, universal interface for test generators to use
-- it essentially turns a definition of valid tests into a graph with
transitions, backtrack, replay, automatic reduction, and code coverage support.

Some of you may be asking: "How does TSTL differ from Hypothesis
(https://hypothesis.readthedocs.io/en/latest/)?"  There are a few
answers.  First, TSTL is probably much less polished than Hypothesis,
right now!  More importantly, however, Hypothesis and TSTL both
generate tests, but they are primarily intended to generate different
kinds of tests.  Hypothesis is in what we consider the QuickCheck
family: if you have a function `f` that takes as input a list, a
string, or something more complex, Hypothesis is very likely what you
want to use.  If you have a set of functions, `f`, `g`, and `h`, and
they don't just return things, but modify invisible system state (but
also return things that may be inputs to other functions), you may
want TSTL.  You can do state-based sequence-of-method-calls testing
with Hypothesis, but it may be easier with TSTL, and it's what TSTL is
built for.  So, if you're testing a sorting implementation, Hypothesis
is almost certainly much better.  If you're testing something like a
file system, you might want to look into TSTL.  If you're testing a
parser that takes a string as input, both tools might be useful,
depending on your situation.

The similarity is that both TSTL and Hypothesis don't look like
traditional unit testing.  They instead let you define the idea of a
valid input (either some data values, or in TSTL a sequence of method
calls and assignments that more resembles a traditional
do-some-stuff-and-then-check-it unit test) and assert general
properties about the behavior of a system under valid input.

For more details on TSTL, the best starting point is a comprehensive
journal paper in STTT:
https://github.com/agroce/tstl/blob/master/doc/papers/STTTjournal/maintstl.pdf.
There are also NASA Formal Methods (NFM) and International Symposium
on Software Testing and Analsysis (ISSTA) 2015 papers at
http://www.cs.cmu.edu/~agroce/nfm15.pdf and
http://www.cs.cmu.edu/~agroce/issta15.pdf, with some implementation
details or concepts that are not present in the more up-to-date and
complete paper.

The NFM and ISSTA papers use an early version of TSTL syntax, which marks
pools and TSTL constructs with % signs.  "Modern" TSTL uses <> by
default, though if for some reason you need <> in your code (and to
prepare for a future C++ version) this can be turned off and only % supported.

Note that documentation below is preliminary.  A better guide to usage
might be to examine the examples directory and see real TSTL test
harnesses.  The generators directory includes some simple but useful
testing tools for finding bugs in systems defined in TSTL.  Only the
random tester is extensively tested and supports all TSTL features
well at this point.  Reading generators/randomtester.py provides
considerable guidance in how to (efficiently) use TSTL in a generic
testing tool, with TSTL providing an interface to the underlying
application/library to be tested.

Installation
------------


    git clone https://github.com/agroce/tstl.git
    cd tstl
    python setup.py install
    # Might need to do a sudo on the last step if not using virtualenv

The documentation below is preliminary, generated by some early
adapters of TSTL, and may not perfectly match the current version.
Use at your own risk.  Code in examples directories should be up to
date.

Using TSTL
------------

The simplest usage is to go to a directory with a .tstl file, and
type:

    tstl mytstlfile.tstl
    python <tstl-root>/generators/randomtester.py

The first line compiles mytstlfile.tstl and produces sut.py, which the
random tester will automatically import and test.  Both tstl and the
random tester take a variety of command line options, which --help
will show.

Example
-----

The easiest way to understand TSTL may be to examine
examples/AVL/avlnew.tstl (https://github.com/agroce/tstl/blob/master/examples/AVL/avlnew.tstl), which is a simple example file in the latest
language format (easier to read) (the file avlblocks.tstl has this
same harness in a different format).

These are pretty full-featured testers for an AVL tree class.  You can
write something very quick and fairly effective with just a few lines
of code, however:

    @import avl
    pool: <int> 3
	pool: <avl> 2
	
	property: <avl>.check_balanced()

	<int> := <[1..20]>
    <avl> := avl.AVLTree()
	
	<avl>.insert(<int>)
	<avl>.delete(<int>)
	<avl>.find(<int>)
    <avl>.display()	

This says that there are two kinds of "things" involved in our
AVL tree implementation testing:  `int` and `avl`.   We define (in
Python, almost) how to create these things, and what we can do with
these things, and then TSTL produces sequences that match our
definition.  It also checks that all AVL trees, at all times, are
properly balanced.  If we wanted, as in avlnew.tstl, we could also
make sure that our AVL tree "acts like" a set --- when we insert
something, we can find that thing, and when we delete something, we
can no longer find it.

If we test this (or avlnew.tstl) for 30 seconds, something like this will appear:

    ~/tstl/examples/AVL$ python ~/tstl/generators/randomtester.py --timeout 30
    Random testing using config=Config(swarmSwitch=None, verbose=False, fastQuickAnalysis=False, failedLogging=None, maxtests=-1, greedyStutter=False, exploit=None, seed=None, generalize=False, localize=False, uncaught=False, speed='FAST', internal=False, normalize=False, highLowSwarm=None, replayable=False, essentials=False, quickTests=False, coverfile='coverage.out', uniqueValuesAnalysis=False, swarm=False, ignoreprops=False, total=False, swarmLength=None, noreassign=False, profile=False, full=False, multiple=False, relax=False, swarmP=0.5, stutter=None, running=False, compareFails=False, nocover=False, swarmProbs=None, gendepth=None, quickAnalysis=False, exploitCeiling=0.1, logging=None, html=None, keep=False, depth=100, throughput=False, timeout=30, output=None, markov=None, startExploit=0)
      12 [2:0]
    -- < 2 [1:0]
    ---- < 1 [0:0] L
    ---- > 5 [0:0] L
    -- > 13 [1:-1]
    ---- > 14 [0:0] L
    set([1, 2, 5, 12, 13, 14])
    ...
      11 [2:0]
    -- < 5 [1:0]
    ---- < 1 [0:0] L
    ---- > 9 [0:0] L
    -- > 14 [1:-1]
    ---- > 18 [0:0] L
    set([1, 5, 9, 11, 14, 18])
    STOPPING TEST DUE TO TIMEOUT, TERMINATED AT LENGTH 17
    STOPPING TESTING DUE TO TIMEOUT
    80.8306709265 PERCENT COVERED
    30.0417540073 TOTAL RUNTIME
    236 EXECUTED
    23517 TOTAL TEST OPERATIONS
    10.3524413109 TIME SPENT EXECUTING TEST OPERATIONS
    0.751145362854 TIME SPENT EVALUATING GUARDS AND CHOOSING ACTIONS
    18.4323685169 TIME SPENT CHECKING PROPERTIES
    28.7848098278 TOTAL TIME SPENT RUNNING SUT
    0.179262161255 TIME SPENT RESTARTING
    0.0 TIME SPENT REDUCING TEST CASES
    224 BRANCHES COVERED
    166 STATEMENTS COVERED

For many (but not all!) programs, a more powerful alternative to
simple random testing is to use swarm testing, which restricts the
actions in each individual test (e.g., insert but no delete, or find
but no inorder traversals) (see
http://www.cs.cmu.edu/~agroce/issta12.pdf).

    ~/tstl/examples/AVL$ python ~/tstl/generators/randomtester.py --timeout 30 --swarm
    Random testing using config=Config(swarmSwitch=None, verbose=False, fastQuickAnalysis=False, failedLogging=None, maxtests=-1, greedyStutter=False, exploit=None, seed=None, generalize=False, localize=False, uncaught=False, speed='FAST', internal=False, normalize=False, highLowSwarm=None, replayable=False, essentials=False, quickTests=False, coverfile='coverage.out', uniqueValuesAnalysis=False, swarm=True, ignoreprops=False, total=False, swarmLength=None, noreassign=False, profile=False, full=False, multiple=False, relax=False, swarmP=0.5, stutter=None, running=False, compareFails=False, nocover=False, swarmProbs=None, gendepth=None, quickAnalysis=False, exploitCeiling=0.1, logging=None, html=None, keep=False, depth=100, throughput=False, timeout=30, output=None, markov=None, startExploit=0)
      11 [2:0]
    -- < 7 [1:0]
    ...
    STOPPING TEST DUE TO TIMEOUT, TERMINATED AT LENGTH 94
    224 BRANCHES COVERED
    166 STATEMENTS COVERED

Here, the method is not very important; simple random testing does a
decent job covering the AVL tree code in just 60 seconds.  If we
introduce a bug by removing the `self.rebalance()` call on line 205 of
avl.py, either method will quickly report a failing test case,
automatically reduced:
        
    ~/tstl/examples/AVL$ python ~/tstl/generators/randomtester.py --timeout 30
    Random testing using config=Config(swarmSwitch=None, verbose=False, fastQuickAnalysis=False, failedLogging=None, maxtests=-1, greedyStutter=False, exploit=None, seed=None, generalize=False, localize=False, uncaught=False, speed='FAST', internal=False, normalize=False, highLowSwarm=None, replayable=False, essentials=False, quickTests=False, coverfile='coverage.out', uniqueValuesAnalysis=False, swarm=False, ignoreprops=False, total=False, swarmLength=None, noreassign=False, profile=False, full=False, multiple=False, relax=False, swarmP=0.5, stutter=None, running=False, compareFails=False, nocover=False, swarmProbs=None, gendepth=None, quickAnalysis=False, exploitCeiling=0.1, logging=None, html=None, keep=False, depth=100, throughput=False, timeout=30, output=None, markov=None, startExploit=0)
    PROPERLY VIOLATION
    ERROR: (<type 'exceptions.AssertionError'>, AssertionError(), <traceback object at 0x10ca2ce60>)
    TRACEBACK:
      File "/Users/alexgroce/tstl/examples/avl/sut.py", line 5998, in check
        assert self.p_avl[2].check_balanced()
    Original test has 32 steps
    REDUCING...
    Reduced test has 11 steps
    REDUCED IN 1.51964116096 SECONDS
    int0 = 7                                                                 # STEP 0
    int3 = 16                                                                # STEP 1
    int2 = 2                                                                 # STEP 2
    avl2 = avl.AVLTree()                                                     # STEP 3
    avl2.insert(int0)                                                        # STEP 4
    avl2.insert(int3)                                                        # STEP 5
    int3 = 2                                                                 # STEP 6
    avl2.insert(int3)                                                        # STEP 7
    int3 = 14                                                                # STEP 8
    avl2.insert(int3)                                                        # STEP 9
    avl2.delete(int2)                                                       # STEP 10

    FINAL VERSION OF TEST, WITH LOGGED REPLAY:
    int0 = 7                                                                 # STEP 0
    int3 = 16                                                                # STEP 1
    int2 = 2                                                                 # STEP 2
    avl2 = avl.AVLTree()                                                     # STEP 3
    avl2.insert(int0)                                                        # STEP 4
    avl2.insert(int3)                                                        # STEP 5
    int3 = 2                                                                 # STEP 6
    avl2.insert(int3)                                                        # STEP 7
    int3 = 14                                                                # STEP 8
    avl2.insert(int3)                                                        # STEP 9
    avl2.delete(int2)                                                       # STEP 10
    ERROR: (<type 'exceptions.AssertionError'>, AssertionError(), <traceback object at 0x10cc28290>)
    TRACEBACK:
      File "/Users/alexgroce/tstl/examples/avl/sut.py", line 5998, in check
        assert self.p_avl[2].check_balanced()
    STOPPING TESTING DUE TO FAILED TEST
    70.1923076923 PERCENT COVERED
    1.69284701347 TOTAL RUNTIME
    2 EXECUTED
    132 TOTAL TEST OPERATIONS
    0.0442249774933 TIME SPENT EXECUTING TEST OPERATIONS
    0.00354051589966 TIME SPENT EVALUATING GUARDS AND CHOOSING ACTIONS
    0.0838589668274 TIME SPENT CHECKING PROPERTIES
    0.128083944321 TOTAL TIME SPENT RUNNING SUT
    0.00327777862549 TIME SPENT RESTARTING
    1.51979422569 TIME SPENT REDUCING TEST CASES
    192 BRANCHES COVERED
    146 STATEMENTS COVERED

Using `--output`, the failing test can be saved to a file, and with the `standalone.py`
utility, converted into a completely standalone test case that does
not require TSTL itself.

    ~/tstl/examples/AVL$ python ~/tstl/generators/randomtester.py --timeout 30 --output failure.test
    Random testing using config=Config(swarmSwitch=None, verbose=False, fastQuickAnalysis=False, failedLogging=None, maxtests=-1, greedyStutter=False, exploit=None, seed=None, generalize=False, localize=False, uncaught=False, speed='FAST', internal=False, normalize=False, highLowSwarm=None, replayable=False, essentials=False, quickTests=False, coverfile='coverage.out', uniqueValuesAnalysis=False, swarm=False, ignoreprops=False, total=False, swarmLength=None, noreassign=False, profile=False, full=False, multiple=False, relax=False, swarmP=0.5, stutter=None, running=False, compareFails=False, nocover=False, swarmProbs=None, gendepth=None, quickAnalysis=False, exploitCeiling=0.1, logging=None, html=None, keep=False, depth=100, throughput=False, timeout=30, output=None, markov=None, startExploit=0)
    ...
    ~/tstl/examples/AVL$ python ~/tstl/utilities/standalone.py failure.test sut.py failure.py --check
	~/tstl/examples/AVL$ python failure.py
    Traceback (most recent call last):
      File "failure.py", line 98, in <module>
        check()
      File "failure.py", line 45, in check
        assert avl2.check_balanced()
    AssertionError


Developer Info
--------------

There are no developer docs yet, which will hopefully change in the future.
The best shakedown test for tstl is to compile and run (using
generators/randomtester.py) the AVL
example.  Removing any call to the balancing function in the avl.py
code should cause TSTL to produce a failing test case.

