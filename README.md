Download Link: https://assignmentchef.com/product/solved-cs2106-lab-4-building-your-own-malloc
<br>
Heap memory region is used for dynamically allocated data. In C, library calls like <strong>malloc()</strong>, <strong>free()</strong>, <strong>realloc()</strong> etc manipulate the heap region. Behind the scene, the heap region can be managed using <strong>contiguous memory allocation scheme </strong>as discussed in lecture 7. For this lab, we are going to implement our very own <strong>malloc()</strong> and <strong>free()</strong> functions. For “mysterious reasons” (ask Uncle Soo), we are going to call our own version <strong><em>mymalloc</em></strong><strong>()</strong> and <strong><em>myfree</em></strong><strong>()</strong> respectively. Don’t panic! Substantial amount of code has already been written, your tasks are to understand, expand and improve the current implementation.




There are three tasks in this lab

<ul>

 <li>Exercise 1: Understand the basic implementation and print simple usage statistics.</li>

 <li>Exercise 2: Change the <strong>first-fit </strong>algorithm to <strong>worst-fit </strong>algorithm in <strong>mymalloc()</strong>.</li>

 <li>Exercise 3: Improve <strong>myfree()</strong> by automatic merging of adjacent free partitions and provide <strong>compaction</strong></li>

</ul>




<h1>Section 2. Heap Region Basics</h1>

<strong> </strong>

On *nix systems, the heap region is defined by three important parameters:







<ol>

 <li><strong>Start:</strong> Starting address of the heap region.</li>

 <li><strong>Break:</strong> The boundary of currently <strong>usable</strong> heap region.</li>

 <li><strong>rLimit:</strong> The maximum boundary of heap region, i.e. <strong>break </strong>can only grows up to <strong>rLimit</strong>. Once <strong>break == rLimit</strong>, we have run out of heap memory.</li>

</ol>

In C, we can use the “set break” <strong>sbrk(<em>size</em>)</strong> system call to increase the <strong>break </strong>boundary by <strong><em>size</em></strong> bytes. The <strong>sbrk(0)</strong> is a special usage which returns the <strong>current address of the break</strong> boundary.




For our usage, we will acquire a contiguous piece of heap region at the program start via <strong>setupHeap()</strong> function. This piece of memory is then used to fulfil all user <strong>mymalloc()</strong> requests. So, there is <strong>no need to use </strong><strong>sbrk() </strong>for your own code.




In the dynamic allocation scheme, we try to provide partition of the <strong>exact size </strong>requested by the users. Instead of using a <em>separate</em> linked list to maintain the partition information, we will use an alternative design in this lab. At the head of each partition, there is a “hidden” <strong>meta data</strong>, which stores:

<ul>

 <li>Size of the partition</li>

 <li>Pointer to the next partition</li>

 <li>Status of the partition</li>

</ul>




Graphically, the layout of our “heap” looks like:










The above show three sample partitions: <strong>2 occupied partitions</strong> (i.e. allocated to user <strong>mymalloc()</strong> requests) and <strong>1 free partition</strong> (possibly allocated and then deallocated by user <strong>myfree()</strong> request). Essentially, you can imagine the linked list of the partition information is <strong>embedded </strong>in the partitions themselves. Also, note that the pointer returned by <strong>mymalloc()</strong> points to <strong>the start of the user data portion</strong>, i.e. <strong>[Starting address of partition] + offset [Size of the meta data]</strong>. The partition meta data is defined as a <strong>partMetaInfo </strong>structure in the given code.




In addition, we have a single <strong>heapMetaInfo </strong>structure <strong>hmi</strong>, which keep tracks of a) the address of the first partition; b) the total size of the whole “heap” and c) the size (in bytes) of the meta data information. (c) is frequently used in calculation, setup and teardown of partitions. The above information should help to shed some lights on the given sample code.

<h1>Section 3. Exercises in Lab 4</h1>

<strong> </strong>

In lab 4, we are going to improve / modify the library calls <strong>mymalloc()</strong>, <strong>myfree()</strong> and a few utility functions. The given skeleton code has the same structure for all exercises:

<ol>

 <li><strong>c</strong>: A sample user program that utilize our <strong>mymalloc()</strong>and <strong>myfree()</strong>. It is used as a test driver for your exercises. <strong>No need to modify</strong>.</li>

 <li><strong>h</strong>: A header file for function declarations. User program should include this header file to use our malloc functionalities. <strong>No need to modify.</strong></li>

 <li><strong>ex</strong><strong><em>X</em></strong><strong>c</strong>: <strong><em>X</em></strong> refers to the exercise number. An implementation of the <strong>mymalloc()</strong> functionalities and your main focus in each of the exercises.</li>

</ol>




For your convenience, we have provided a <strong>makefile </strong>for each exercise. You can simply type “<strong>make</strong>” to build the <strong>a.out</strong> executable.




<h1>3.1 Exercise 1 – Simple Usage Statistic [Demo Exercise]</h1>




For this exercise, calculate the following statistics in the <strong>printHeapStatistic</strong>() function:

<ul>

 <li>Number of occupied partition and the total size of occupied partition in bytes.</li>

</ul>

Note that the meta data of each partition is not counted.

<ul>

 <li>Number of free partition (aka hole) and the total size of holes in bytes.</li>

 <li>Total size of all meta information block in bytes.</li>

</ul>




<table width="501">

 <tbody>

  <tr>

   <td width="501">Sample Output (using <strong>test1.in</strong>, information in <strong>bold </strong>needs to be calculated):</td>

  </tr>

  <tr>

   <td width="501">Heap Meta Info:===============Total Size = 1024 bytesStart Address = 0x1dd5000 Partition Meta Size = 24 bytesPartition list:[+    0 |    40 bytes | 1][+   64 |    80 bytes | 0][+  168 |   120 bytes | 0][+  312 |   160 bytes | 1][+  496 |   504 bytes | 0] Heap Usage Statistics: ======================Total Space: 1024 bytesTotal Occupied Partitions: <strong>2</strong>     Total Occupied Size: <strong>200</strong> bytesTotal Number of Holes: <strong>3</strong>Total Hole Size: <strong>704</strong> bytesTotal Meta Information Size: <strong>120</strong> bytes</td>

  </tr>

 </tbody>

</table>

You can see that the total occupied size, total hole size and total meta information size should add up to the total size of the heap (<strong>1024</strong> bytes in this test).




Again, don’t panic J. You will spend more time to understand the given code rather than coding yourself. (hint: you need about 10 lines of code for this exercise).







<h1>3.2 Exercise 2 – Worst-fit Algorithm</h1>

<strong> </strong>

The given <strong>mymalloc()</strong> implements the <strong>first-fit </strong>algorithm, i.e. we allocate the first large enough free partition for user requests. Change the <strong>mymalloc()</strong> function in <strong>ex2_mmalloc.c</strong> so that we use <strong>worst-fit </strong>algorithm, i.e. the largest free partition is chosen instead.




You are not allowed to modify the existing functions and declarations, but can implement helper function(s) if needed. <strong>There is also no need to copy your solution for ex1 over</strong>.




<table width="492">

 <tbody>

  <tr>

   <td width="492">Sample Output (using <strong>test1.in</strong>, just before the last <strong>mymalloc(88)</strong> request)</td>

  </tr>

  <tr>

   <td width="492"><strong>Heap Meta Info: </strong><strong>=============== </strong><strong>Total Size = 1024 bytes </strong><strong>Start Address = 0xa30000 Partition Meta Size = 24 bytes </strong><strong>Partition list: </strong><strong>[+    0 |    40 bytes | 1] </strong><strong>[+   64 |    80 bytes | 0] </strong><strong>[+  168 |   120 bytes | 1] </strong><strong>[+  312 |   160 bytes | 0] </strong><strong>[+  496 |   200 bytes | 1] </strong><strong>[+  720 |   240 bytes | 0] </strong><strong>[+  984 |    16 bytes | 0] </strong></td>

  </tr>

 </tbody>

</table>




<table width="493">

 <tbody>

  <tr>

   <td width="493">Sample Output (using <strong>test1.in</strong>, after the last <strong>mymalloc(88)</strong> request)</td>

  </tr>

  <tr>

   <td width="493"><strong>Heap Meta Info: =============== </strong><strong>Total Size = 1024 bytes </strong><strong>Start Address = 0xbc6000 Partition Meta Size = 24 bytes </strong><strong>Partition list: </strong><strong>[+    0 |    40 bytes | 1] </strong><strong>[+   64 |    80 bytes | 0] </strong><strong>[+  168 |   120 bytes | 1] </strong><strong>[+  312 |   160 bytes | 0] </strong><strong>[+  496 |   200 bytes | 1] </strong><strong>[+  720 |    88 bytes | 1] </strong><strong>[+  832 |   128 bytes | 0] </strong><strong>[+  984 |    16 bytes | 0] </strong></td>

  </tr>

 </tbody>

</table>




The row in red shows that worst fit algorithm is used because the largest free partition at offset 720, size 240 bytes was used to satisfy the request <strong>mymalloc(88)</strong>. If you need a step by step print out of the heap layout for debugging purpose, you can add a <strong>“-DDEBUG</strong>” flag during compilation.




Please note that the skeleton code for this exercise will result in a segmentation fault before your code is added as the <strong>mymalloc()</strong> functionfrom exercise 1 has been removed.

<strong> </strong>

<h1>3.3 Exercise 3 – Merging and Compaction</h1>

<strong> </strong>

The given implementation of <strong>myfree()</strong> does not perform merging.  Even if the newly freed partition has free neighbouring partition(s), no coalescing is performed which leave them as separate free holes. One example can be seen from the sample output in exercise 2, the last two partitions are both free, but not merged.




Modify the <strong>myfree() </strong>function in <strong>ex3_mmalloc.c</strong> so that merging is performed after a partition is freed. The given test case <strong><em>test1.in</em></strong> test the following scenario:




1            2             3             4             5             6             7             8            9

<table width="514">

 <tbody>

  <tr>

   <td width="47">Occ</td>

   <td width="59">Occ</td>

   <td width="59">Occ</td>

   <td width="59">Occ</td>

   <td width="59">Occ</td>

   <td width="59">Occ</td>

   <td width="59">Occ</td>

   <td width="59">Occ</td>

   <td width="51">Free</td>

  </tr>

 </tbody>

</table>




8 partitions (40 bytes each) were allocated <strong>initially</strong>. We then procced to free:

<ol>

 <li>Partition 1:      nothing special, just deallocate.</li>

 <li>Partition 3:      nothing special, just deallocate.</li>

 <li><strong>Partition 4: should merge with the freed partition 3 </strong></li>

 <li>Partition 7:      nothing special, just deallocate.</li>

 <li><strong>Partition 6: should merge with the freed partition 7 </strong></li>

 <li><strong>Partition 2: should merge with the free partition 1 and the merged free partition 3 and 4. </strong></li>

</ol>




The end result is shown in the test output <strong>test1.out</strong>:




<table width="493">

 <tbody>

  <tr>

   <td width="493">Sample Output (using <strong>test1.in</strong>)</td>

  </tr>

  <tr>

   <td width="493"><strong>Heap Meta Info: =============== </strong><strong>Total Size = 1024 bytes </strong><strong>Start Address = 0x111a000 Partition Meta Size = 24 bytes Partition list: </strong><strong>[+    0 |   232 bytes | 0]  </strong>//merged free partitions 1, 2, 3 and 4<strong>[+  256 |    40 bytes | 1] </strong><strong>[+  320 |   104 bytes | 0]  </strong>//merged free partitions 6 and 7<strong>  </strong><strong>[+  448 |    40 bytes | 1] </strong><strong>[+  512 |   488 bytes | 0]  </strong>//the initial free partition 9</td>

  </tr>

 </tbody>

</table>

Finally, you need to provide <strong>compaction </strong>functionality via the <strong>compact() </strong>function call. Compaction moves all allocated partitions to the start of the heap region and consolidate all holes into one free partition at the end of the heap region.




In the given test case <strong>test2.in</strong>, we have the following layout before compaction is performed:




1            2             3             4             5             6             7             8            9

<table width="514">

 <tbody>

  <tr>

   <td width="47">Free</td>

   <td width="59">Occ</td>

   <td width="59">Free</td>

   <td width="59">Occ</td>

   <td width="60">Free</td>

   <td width="59">Occ</td>

   <td width="60">Free</td>

   <td width="59">Occ</td>

   <td width="51">Free</td>

  </tr>

 </tbody>

</table>




After the compaction, the end result is shown in the test output:




<table width="493">

 <tbody>

  <tr>

   <td width="493">Sample Output (using <strong>test1.in</strong>)</td>

  </tr>

  <tr>

   <td width="493"><strong>Heap Meta Info: =============== </strong><strong>Total Size = 1024 bytes </strong><strong>Start Address = 0x1c93000 Partition Meta Size = 24 bytes </strong><strong>Partition list: </strong><strong>[+    0 |    40 bytes | 1] </strong><strong>[+   64 |    40 bytes | 1] </strong><strong>[+  128 |    40 bytes | 1] </strong><strong>[+  192 |    40 bytes | 1] </strong><strong>[+  256 |   744 bytes | 0] </strong></td>

  </tr>

 </tbody>

</table>




You can see that all four occupied partitions were moved to the start of the region and the holes were merged into a big free partition at the end. In the given code, we will perform a “compact verification” process as a check on your <strong>compact()</strong> implementation.




Several <strong>key criteria </strong>for a correct implementation:

<ul>

 <li>The relative ordering of all occupied partitions should be maintained after compaction.</li>

</ul>




<ul>

 <li>The <strong>user data </strong>of each partition needs to be copied to new location. Remember: There are actual user data in the occupied partitions! So, you need to relocate the user data instead of just changing the partition meta information. We simulate user data by placing <strong>magic numbers </strong>at the start and end of each of the occupied partitions (you can find out more from the <strong>c</strong>).</li>

</ul>




<ul>

 <li>There should be only 1 free partition after compaction. This hole should contains all remaining free space in the heap region.</li>

</ul>




<strong>           </strong>

<h1>Section 4. Level UP! [Just for your pondering, no need to submit]</h1>

<strong> </strong>

This exercise attempts to shed lights on some “weird” behaviours you may have encountered before with the standard <strong>malloc()</strong>, <strong>myfree()</strong>. For example:

<ol>

 <li>Why data in recently freed memory space seems intact (for a while)?</li>

</ol>




<ol start="2">

 <li>Why exceeding the range of allocated memory space <strong>sometimes</strong> does not cause segmentation fault?</li>

</ol>




<ol start="3">

 <li>Why dynamically allocated memory space contains “random” garbage values?</li>

</ol>




You should be able to give a pretty good guess / answer to the above questions after this lab.




Now, one last surprise. Once you have the complete code, you can try to link any code that uses dynamic memory with the <strong>mmalloc.c</strong>, e.g. linked list code (e.g. from lab 1) would be a good showcase. The linked list code should work perfectly with your very own <strong>mymalloc()</strong> and <strong>myfree()</strong>, fun eh? J

[Note: you need to remove the “<strong>&lt;stdlib.h&gt;</strong>” library, as it provides the standard malloc/free and can conflict with your own implementations.] [Note2: Of course, you need to rename all <strong>malloc()</strong> to <strong>mymalloc()</strong> and <strong>free()</strong> to <strong>myfree()</strong> in the user code to utilize your own version.]




<strong> </strong>


