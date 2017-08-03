# System-Call-in-Linux-kernel
This repository contains the information about a custom System Call, BUT it does not have the code. One can email me at - arrajan@cs.stonybrook.edu, if you would like to explore the source code in C language.

Introduction :
A system call, sometimes referred to as a kernel call, is a request in a Unix-like operating system made via a software interrup by an active process for a service performed by the kernel. In this project, we created a Loadable Linux kernel Module (LKM) that when added into linux 4.6 that perform the sorting operation. 

Files :
1. sys_my_sort.c (Kernel Code)
2. xhw1.c (user space code)
3. Makefile 
4. install_module.sh (Script to install kernel module)

System call name : my_sort       

How does it work : 
'my_sort' system call is invoked from a user space program 'xhw1.c'. It takes input 1, input 2 and target file name. A user has to provide 'flags' too. It takes records from both the input files, sort them and store in the target file based on what 'flag' options you provide. 
 
How to execute ?
./xhw1 [-uitd] outputfile input1 input2

Flags :
-u 0x01: output sorted records; if duplicates found, output only one copy
	 (e.g., ala "sort -u")
-a 0x02: output all records, even if there are duplicates
-i 0x04: compare records case-insensitive (case sensitive by default)
-t 0x10: if any input file is found NOT to be sorted, stop and return an
	 error (EINVAL); otherwise continue to output records ONLY if any
	 are found that are in ascending order to what you've found so far.
-d 0x20: return the number of sorted records written out in "data"

Description:
1. User Space :
a) The user program performs basic checks on the input arguments, for example, a combination of 'u' and 'a' can NOT be provided. 
   Following struct is used to process the arguments. 
   input_info{
           char *src1;
           char *src2;
           char *dest;
           int flags;
           int data;
   };

b) After processing which all 'flags' are provided, system call is invoked.
c) Based on the result of the system call, 'my_sort', information is provided to the user. For example, if 'd' flag is set, the total number of records written to the output file is shown.

2. Kernel Space
a) Validation of System call arguments : This is one of the most crucial step, where a lot of checks are performed on the user input considering the philosophy 'Never trust the user' in mind. 
For example, i) 'copy_from_user()' is used to validate the address (virtual to physical)  being passed.
             ii) Input file names are checked, if the files don't exist, return an error.
             iii) If the target file exist, it gets deleted. A fresh file with the same name exist. Please note that, if the system call fails at any point of time, output file gets deleted.
             
If any check fails, appropriate error number is returned.
b) Memory is allocated for processing the records from input file. If any 'kmalloc' fails, return -ENOMEM
c) Input files are read with the help of 'vfs_read' in the chunk of 4096 KB, the page size.
d) Based on the flags given in the input, output is written to the target file in the chunk of 4096 KB, the page size.

e) If everything goes OK, free all the resources and close all the file descriptors.
f) return success to the user program.



Good Things and Bad Things:
 
1) Minimal use of kernel memory, only 3 buffers are being used.

2) This implementation does not work for 't' flag.

References Used:
1. http://www.linfo.org/system_call.html
2. http://lxr.fsl.cs.sunysb.edu/linux/source/fs/


