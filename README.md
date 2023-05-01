Download Link: https://assignmentchef.com/product/solved-assignment-5-y86-64-system-emulator-cs429
<br>
<h1>1        Introduction</h1>

In this lab, you will be implementing two simulators:

<ul>

 <li>psim, a standalone simulator for the PIPE implementation of the Y86-64 instruction set architecture (ISA), assuming an ideal memory system; and</li>

 <li>pcsim, an integrated “PIPE-with-CACHE” simulator, by implementing a cache simulator csim, enhancing psim to handle variable delays in the memory stage, and connecting them to each other, producing a simulator for the PIPE implementation of the Y86-64 ISA with a realistic memory hierarchy.</li>

</ul>

For reference, we are also providing you with the source code for ssim, a standalone simulator for the SEQ implementation of the Y86-64 ISA.

Outcomes you will gain from this lab include the following:

<ul>

 <li>You will understand how the SEQ implementation of Y86-64 works. You will understand the utilities of each stage and how they are connected to each other.</li>

 <li>You will understand how the PIPE implementation of Y86-64 works. You will understand how stalling, squashing, and forwarding help handle different hazard conditions.</li>

 <li>You will understand the impact that cache memories can have on the performance of programs.</li>

 <li>You will understand the additional changes that need to be made to the PIPE implementation to accommodate a realistic memory hierarchy.</li>

</ul>

<h1>2        Logistics</h1>

This assignment lasts for three weeks and consists of two interrelated parts. You are required to perform two submissions:

<ul>

 <li>The checkpoint, due by Tuesday, 30 November 2021 23:59 CT.</li>

 <li>Final hand-in, due by Monday, 06 December 2021 23:59 CT.</li>

</ul>

The purpose of the checkpoint is to make sure that you have started and made some demonstrable progress on the assignment.

You may use up your remaining late (slip) days for the checkpoints and final uploads of this assignment. <em>Note that using slip days for a checkpoint does not adjust any future due dates. </em>Start early enough to get the assignment done before the due date. Assume things will not go according to plan, and so you must allow extra time for heavily loaded systems, dropped internet connections, corrupted files, traffic delays, minor health problems, etc.

<em>This is an individual or partner project. </em>If you choose to work in pairs, the team may use as many slip days as the partner with the most available slip days. That is, if you have 2 slip days and your partner has 3, the team gets 3 slip days to use for the entire assignment. All hand-ins are electronic. You may do your coding on any machine you choose, <em>but it is your responsibility to test this assignment for correct build/execution on an UTCS 64-bit x86-64 Linux machine before your final hand-in. </em>You may not share your work on lab assignments with other students outside your team, but feel free to ask instructors for help (e.g., during office hours or discussion sections). Unless it’s an implementation-specific question (i.e., private to instructors), please post it on Piazza publicly so that students with similar questions can benefit as well.

Before you begin, please take the time to review the course policy on academic integrity at: <a href="https://www.cs.utexas.edu/academics/conduct">https:</a>

<a href="https://www.cs.utexas.edu/academics/conduct">//www.cs.utexas.edu/academics/conduct</a><a href="https://www.cs.utexas.edu/academics/conduct">.</a> Don’t copy code from anywhere; do it yourself. This discipline is very important for this class and next classes.

Any updates for this lab will be posted on Canvas. Any clarifications or corrections for this lab will be posted on Piazza.

<h1>3        Download and Setup</h1>

<ol>

 <li>On your local machine, download the file SE-handout.tar from the Assignment 5: SE + WU-SE folder in the Files tab of the Canvas page for the class. Then use Linux command scp to copy the tarball over to your account on a lab machine.</li>

 <li>Use tar xvf SE-handout.tar on the lab machine to extract the contents of the tarball into the directory SE-handout. Then navigate to the root directory and create a new git repository with command git init. Use git add to add at least these files required for submission: pipe/psim.c, cache/cache.c, pipe-cache/pcsim.c, y86-code/myprog.ys.</li>

 <li>Create a new private repository in your GitHub account named SE-Lab and link it to your local repository. You should know how to do this from previous assignments.</li>

 <li>From the project root directory, run make to compile all the y86 modules and utility programs (see details below). Alternatively, you can use bash all.sh to run tests along with the compilation, but a lot of failure messages will pop up, since the simulators haven’t been implemented yet.</li>

</ol>

<h1>4        Assignment Details</h1>

<h2>4.1       Repository Structure</h2>

Now that you have your private repository of the code base, confirm that you have the following subdirectories:

<ul>

 <li>misc: Source code files for utilities such as yas (the Y86-64 assembler) and yis (the Y86-64 instruction set simulator). It also contains the isa.c source file that is used by all of the processor simulators.</li>

 <li>seq: Source code for the SEQ simulator. This is solely for your reference. See file README for instructions on compiling and using the simulator.</li>

 <li>pipe: Source code for the PIPE simulator. You will modify psim.c to complete this simulator in Part A. See file README for instructions on compiling and using the simulator.</li>

 <li>cache: Source code for the CACHE simulator. You will modify cache.c to complete this simulator in Part B. csim simulates a cache controller, and is tested using the test-csim executable.</li>

 <li>pipe-cache: Source code for the “PIPE-with-CACHE” simulator. You will modify cache.c and pcsim.c to complete this simulator in Part B. See file README for instructions on compiling and using the simulator.</li>

 <li>y86-code: Y86-64 assembly code for some of the example programs shown in CS:APP3e Chapter 4. You can automatically test your SEQ and PIPE simulators on these benchmark programs. See file README for instructions on how to run these tests.</li>

 <li>ptest: Scripts that generate systematic regression tests of the different instructions, the different jump possibilities, and the different hazard possibilities. These scripts are very good at finding bugs in your implementation. See file README for instructions on how to run these tests.</li>

</ul>

<h2>4.2       Utility Programs</h2>

The misc directory contains two useful programs:

<ul>

 <li>yas: The Y86-64 assembler. This takes a Y86-64 assembly code module (a text file with extension .ys) and generates a Y86-64 object module (a “binary” file <a href="#_ftn1" name="_ftnref1"><sup>[1]</sup></a> with extension .yo). The easiest way to invoke the assembler is to use or create assembly code files in the y86-code subdirectory. For example, to assemble the program in file prog1.ys in this directory, use the command:</li>

</ul>

linux&gt; make prog1.yo

Note: The yas binary is pre-compiled and is only guaranteed to work on the UTCS x86-64 Linux machines.

<ul>

 <li>yis: The Y86-64 ISA simulator. This program interprets the instructions in a Y86-64 machine-level program according to the instruction set definition, and is the “gold standard” IaaT definition of instruction semantics that your simulators must replicate accurately. To run the program prog.yo from within the subdirectory y86-code, simply run:</li>

</ul>

linux&gt; ../misc/yis prog.yo

yis simulates the execution of the program and then prints changes to any registers or memory locations on the terminal, as described in CS:APP3e §4.1. For your convenience, yis also prints out any intermediate changes for each step.

<h2>4.3       Helper Functions</h2>

There are several helper functions provided to help you with your psim.c implementation:

<ul>

 <li>HPACK() is a macro defined that takes two 4-bit quantities and combines them into a single byte.</li>

</ul>

HPACK(0x5, 0xB) returns 0x5B.

<ul>

 <li>HI4() is a macro defined that takes a byte and returns the upper 4 bits of it. Similarly, LO4 is a macro that takes a byte and returns the lower 4 bits of it. HI4(0x5B) and LO4(0x5B) return 0x5 and 0xB respectively.</li>

 <li>getbyteval(), getwordval(), and getregval() are functions for accessing memory or the register file. They are fully declared in isa.c, but they essentially take a byte array (simulating the machine’s memory) and an address or register id to look up. Accessing memory also requires providing a pointer to the read memory’s destination.</li>

 <li>condholds() is a method provided to check whether a conditional jump or move should be taken, that takes the current condition codes and the condition to be evaluated.</li>

 <li>computealu() and computecc() are methods provided for computing the result of an arthmetic function and its resulting condition codes when an OPq instruction is being executed.</li>

</ul>

<h1>5        Implementing Simulators</h1>

For the PIPE processor implementation, we have provided a dummy implementation in psim.c that you can compile and run before you start filling in the missing details.

<h2>5.1       Simulator Command Line Options</h2>

You can specify several options from the command line of the simulator:

<ul>

 <li>-h: Prints a summary of all of the command line options.</li>

 <li>-l <em>m</em>: Sets the instruction limit, executing at most <em>m </em>instructions before halting. The default limit is 10,000 instructions.</li>

 <li>-v <em>n</em>: Sets the verbosity level to <em>n</em>, which must be between 0 and 3 (for SEQ) or between 0 and 2 (for PIPE), with a default value of 2.</li>

 <li>-i: Runs the simulator in interactive mode.</li>

</ul>

The final command-line argument specifies the object filename.

Here are some typical invocations of the simulators from within the seq subdirectory, for example:

linux&gt; ./ssim -h

linux&gt; ./ssim ../y86-code/prog1.yo linux&gt; ./ssim -i ../y86-code/prog1.yo The first case prints a summary of the command line options for ssim. The second case runs ssim on object file prog1.yo until the simulator halts. The resulting register and memory values are compared with those from the ISA simulator (yis). The third case launches ssim in interactive mode, reads object file prog1.yo, and waits for further commands.

In interactive mode, you can use more commands to step through the program:

<ul>

 <li>help : print help message (or use the first letter h for short, similar for below)</li>

 <li>go : run program to completion</li>

 <li>next n : advance n instructions</li>

 <li>cycle n : advance n cycles</li>

 <li>memory : display differences in memory</li>

 <li>registers : display differences in registers</li>

 <li>arch : display processor state</li>

 <li>undo n : step back n instructions (or use u n for short)</li>

 <li>back n : step back n cycles</li>

 <li>pipe X : display pipeline info for stage X (f, d, e, m, w)</li>

 <li>quit : exit the program</li>

</ul>

The same invocations work for the PIPE simulator psim from within the pipe subdirectory, and with pipe-cache/pcsim as well.

The dummy implementation we provide doesn’t update any state and never halts, so you will see the simulator falling into an infinite loop and stopping only after reaching the 10,000-instruction limit.

<h1>6        Programming Tasks</h1>

This lab is a sequence of two programming parts. In Part A, you will implement psim, a PIPE simulator. In Part B, you will implement pcsim, a “PIPE-with-CACHE” simulator, by implementing a cache simulator csim, enhancing psim to handle variable delays in the memory stage, and connecting them to each other.

The assignment carries nine points: one point for the checkpoint, and four points each for Parts A and B. There is an additional three-point bonus for the extra-credit section.

<h2>6.1       Part A: Implementing <strong>psim</strong>, A Simulator for the PIPE Implementation</h2>

The goal for Part A is to implement a PIPE simulator as described in CS:APP3e §4.5. You are going to complete the code in pipe/psim.c. The only functions you will modify are simsteppipe() and several helper functions in it. The detailed instructions are provided in the comment blocks marked “TODO”.

<h3>6.1.1      Basic Pipelined Implementation</h3>

You are required to implement five functions in simsteppipe() that emulate five stages for your PIPE simulator:

<ul>

 <li>dofetchstage(): Fetch stage.</li>

 <li>dodecodestage(): Decode stage.</li>

 <li>doexecutestage(): Execute stage.</li>

 <li>domemorystage(): Memory stage.</li>

 <li>dowritebackstage(): Write-back stage.</li>

</ul>

In hardware, all the stages would execute concurrently for multiple instructions in a single cycle of a PIPE implementation. Since C code is executed sequentially, however, you need to process the stages in reverse order (decode stage after execute and memory stages, and memory stage before execute) in order to propagate forwarding values properly among these multiple executions in flight through the pipeline. <em>This ordering is already handled in the code skeleton; you do not need to worry about it.</em>

The pipeline register contents for PIPE are declared as structs in stages.h. The contents of an input pipeline register into the next stage are pointed by XXinput (which you should update), and the contents of an output pipeline register from the last stage are pointed by XXoutput. Thus, dodecodestage() will read from Doutput and write to Einput. See function siminit() in psim.c to get an idea of how it works.

<h3>6.1.2      Hazard Control</h3>

You are also required to implement stalling, squashing, and forwarding to deal with data hazards and control hazards, as described in CS:APP3e §4.5.5. For stalling, you are provided with a helper function dostallcheck(). Implement this function to make function updatepipes() handle stalling correctly. For forwarding, add your own implementation in the functions for stage updating mentioned above.

<h3>6.1.3      Testing</h3>

Use the following command to run your PIPE simulator against the standard YIS simulator on a Y86-64 machine-level program (prog1.yo, for example):

linux&gt; make clean; make linux&gt; ./psim ../y86-code/prog1.yo

There is also a provided psim-ref executable that you can use (with the same arguments as psim) to check the correct output of a test case. Or you can run an exhaustive test with ptest:

linux&gt; make clean; make linux&gt; cd ../ptest linux&gt; make SIM=../pipe/psim

When the ptest program detects an erroneous simulation, it leaves the corresponding .ys file in the directory so that you can debug on it. When this occurs, you can use the provied compareref.py to find the line where your solution and the reference differ:

linux&gt; cd pipe

linux&gt; python3 compare_ref.py -f &lt;failing .yo file&gt;

<h3>6.1.4      Submission</h3>

Submit your checkpoint version using Gradescope, by providing a pointer to the private GitHub repository where you have done your work. <em>Clearing checkpoint 1 corresponds to your correctly running test programs </em>y86-code/prog1.yo <em>through </em>y86-code/prog9.yo.

<h3>6.1.5      Evaluation</h3>

Part A of the assignment counts for four points. We will run your PIPE simulator using <em>ptest </em>(a regression suite of 743 tests) to check the correctness of your implementation.

<h2>6.2       Part B: Implementing <strong>pcsim</strong>, A Simulator for the PIPE Implementation with A Realistic Memory Hierarchy</h2>

In this part, you will first write a standalone cache controller simulator csim and test it against a number of memory traces. Correctness will be determined by matching the cache events generated by your simulator against a reference. You will then augment psim and connect it to csim to produce pcsim.

<h3>6.2.1      Implementation and Testing</h3>

<ul>

 <li>Start your work in the cache directory.</li>

 <li>Implement the get_line() and select_line() helper functions in the file cache.c. Implement the check_hit() and handle_miss() routines in the file cache.c. This will give you a skeletal cache simulator that implements the control actions (the three-state finite-state machine cache controller discussed in class) of a write-back cache with LRU replacement and write-allocate policies, for arbitrary numbers of sets, associativity values, and block sizes.</li>

 <li>You can assume that each cache read/write only accesses one single cache line.</li>

 <li>Test your code by running make test-csim and running test-csim. Your implementation is correct when the test score printed out is 40/40. <em>You do not have to turn in this code.</em></li>

 <li>Next, copy the contents of your psim/psim.c into pipe-cache/pcsim.c. This will give you a baseline code to start from. Copy stage by stage, and make sure to take note of the existing code provided in the memory stage of pcsim.c to see how the new memory functions are used.</li>

 <li>Implement the four functions get_byte_cache, get_word_cache, set_byte_cache, and set_word_cache in the file cache/cache.c. After completing this task, you will have a fully functional cache simulator that implements both the control and the data portions of the cache.</li>

 <li>Integrate the memory hierarchy simulator into the PIPE simulator by making the appropriate changes in the file pcsim.c. This should involve no more than updating the memory routines to the corresponding cache routines and handling stalls resulting from cache misses. In the fetch stage, you might need the functions get_byte_val_I and get_word_val_I. In the memory stage, you might need the functions get_word_val_D and set_word_val_D.</li>

 <li>Check the correctness of the combined simulator using make test-pipe-cache in the ptest directory. There are a total of 64 tests run against the simulator: eight programs, each tested against eight different cache configurations.</li>

</ul>

<h3>6.2.2      Submission</h3>

Submit your final version using Gradescope, by providing a pointer to the private GitHub repository where you have done your work.

<h3>6.2.3      Evaluation</h3>

Part B of the assignment counts for four points. We will run your pcsim simulator using the command make test-pipe-cache in the ptest directory against eight programs, using eight different cache configurations for each program.

<h1>7        Extra Credit (Optional)</h1>

There are three optional extra-credit opportunities, each worth one point <em>towards your course total</em>.

<h2>7.1       Custom Y86 tests</h2>

Create an original test case in the file y86-code/myprog.ys. In order to earn the bonus, two conditions need to be satisfied:

<ul>

 <li>This test case needs to be non-trivial and demonstrate some interesting feature or quirk of the architecture.</li>

 <li>Both your psim and pcsim simulators must correctly execute this test case.</li>

</ul>

<h2>7.2       Additional Y86 Instructions</h2>

Implement all of the following instructions as an extension to the Y86 ISA in your PIPE simulator:

<ul>

 <li>leaq D(rB), rA: Load the effective address into rA following the Y86 Base + Displacement addressing mode.</li>

 <li>vecadd rA, rB: Add the corresponding bytes in rA and rB together and save the result to rB.</li>

 <li>shlq | shrq | sarq rA, rB: The x86-like left shift, right logical shift, and right arithmetic shift instructions.</li>

</ul>

The yas and yis programs have been implemented to handle these custom instructions already. You can use them as the reference. Modify your PIPE simulator code in-place, and make sure it still works for all previous tests in ptest.

<h2>7.3       Improved Cache Implementation</h2>

For the cache in Part B, we assumed that each cache read/write only accesses a single cache line. Improve your cache implementation in cache.c so that it can handle read/write requests spanning multiple cache lines. Run make cache-bonus in the ptest directory to check your implementation. Your improved cache must still work correctly for the original tests.

<a href="#_ftnref1" name="_ftn1">[1]</a> The generated file actually contains an ASCII version of the object code, and is therefore not truly a binary file.