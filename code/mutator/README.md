# Master thesis
Tool name: Cyber-Physical Mutation tool
Author names: Balaji Balasubramaniam, Prof. Justin Bradley, Prof. Sebastian Elbaum
University of Nebraska-Lincoln

# Configure
1) Go to `Makefile` and change path, the below line, in headers to point to llvm installation in your computer.

HEADERS := -isystem /usr/lib/llvm-4.0/include/

2) Optional - Go to `compilemutator.bash` file and change the container name if you are using different docker container.`compilemutator.bash` file is used to automate two steps: make and then run. It is recommended to first follow the step by step tutorial before modifying or running the `compilemutator.bash` file.

3) Optional - change the configuration paths for compiler of target_file in `config_<project name>.txt` acccording to your software environment.

4) Optional - change the path in `compilemutator.bash`, `compilerun.bash`, and `runsave.bash` based on the location where you have unzipped and placed the mutationdocker for the project directory of the target_file. `compilerun.bash`, and `runsave.bash` file is used to automate the compilation and execution of target_file, it varies depending on the project. 

5) Optional - change `spacevariables.txt` file. Provide names of the variables that needs to be mutated. For example: if the file has line "location,*,50" then the tool will find the occurence of variable name "location" in the target program and chage it to "location * 50" in all the places.

6) Optional - The program attempts to automatically detect the target platform for abstract syntax tree parsing templates. If you want to manually configure the platform, then you can provide the information in the `platform.txt` file. First line is either true or false. True enables the program to automatically detect the platform. Second line is the platform name. Currently, the tool supports only MATLAB and C. MATLAB represent simulink generated C code with filname extension as `.c`.

# Dependecies
1) Ubuntu 16.04

2) docker version
--------------
Client:
 Version:      1.13.1
 API version:  1.26
 Go version:   go1.7.5
 Git commit:   092cba3
 Built:        Wed Feb  8 06:50:14 2017
 OS/Arch:      linux/amd64

Server:
 Version:      1.13.1
 API version:  1.26 (minimum version 1.12)
 Go version:   go1.7.5
 Git commit:   092cba3
 Built:        Wed Feb  8 06:50:14 2017
 OS/Arch:      linux/amd64
 Experimental: false

3) clang --version
---------------
clang version 3.8.0-2ubuntu4 (tags/RELEASE_380/final)
Target: x86_64-pc-linux-gnu
Thread model: posix
InstalledDir: /usr/bin

4) llvm-4.0

5) cloc -version
--------------
1.60

# To build and run
To run a code sample, you need to build the docker container:

```sh
$ docker build -t clang .
```

If you then `cd` into mutator directory, like `code/mutator`, you will find a
`Makefile` specifically for this tool. To compile or use the tool, enter the
docker container and mount the folder under `/home`:

```sh
$ docker run -it -v $PWD:/home clang
```

Then just use the code executable already there, or re-build it with `make`. Please note that if you make changes to `config_<project name>.txt` file or `spacevariables.txt` file then you have to execute `./runconfiguration config_<project name>.txt` first. Similarly, if you make chages to `Makefile` then you have to re-build it with `make` before running the tool. To summarize, the order of execution is first `./runconfiguration`, followed by `make`, and then the tool execution as below:

```sh
$ ./mutator -seed=7 -number=0 -category=PET b747cl.c
```
(or)

```sh
$ ./mutator -s 7 -n 0 -c PET b747cl.c
```

#Tool run - step by step tutorial

Normal Steps: 
1) Unzip mutationdocker.zip file 
2) Run in shell - cd to enter the directory, "cd mutationdocker" 
3) Run in shell - "docker build -t clang ." 
4) Run in shell - "cd code/mutator" 
5) Run in shell - "docker run -it -v $PWD:/home clang" 
6) Run in docker shell- "make" 
7) Run in docker shell - "exit" 
8) Run in shell - "./mutator -s 7 -n 0 -c PET b747cl.c" 
9) After step 6 you can run in docker shell - "./mutator -s 7 -n 0 -c PET b747cl.c", provided the docker has configured to access the system resources

Steps for naming the container (this should be carried out only after performing all the normal steps mentioned above): 
1) Run in shell - "cd code/mutator" 
2) Run in shell - "docker run -it --name "mutationcontainer" -v $PWD:/home clang" 
3) Run in docker shell - "exit" 
4) Run in shell - "docker start mutationcontainer" 
5) Run in shell - "./compilemutator.bash" (make and execute) 
6) Run in shell - "docker stop mutationcontainer" 

# Tool options
```sh
$ ./mutator <seed> <number> <categories> <target_file>
```

seed = random seed 

number = number of mutants, please note that number 0 represents the maximum possible mutants to be generated from the target_file.

categories = PET (Precision&Accuracy, Exception Handling, Time and Space)

# Output details
The generated mutants are present in a folder called 'mutants'. This folder contains subfolders for each template as mentioned below:
-----------------------------------------------------------------------------------------------------------------------------
Category    	|  Mutation template  |        MATLAB platform		   			|        C/ CPP platform      				|
-----------------------------------------------------------------------------------------------------------------------------
Precision   	|         P0          |  int_T -> uint32_T 		  				| int -> float 								|
& accuracy    	|         P1          |  int_T -> real_T 		  				| int -> double								|
	    		|         P2          |  uint32_T -> int_T 		  				| float -> int								|
	    		|         P3          |  real_T -> int_T 		  				| double -> int 							|
	    		|         P4          |  d*-> d*.0f 			  				| d*-> d*.0f 								|
	    		|         P5          |  d* -> d*.0 			  				| d* -> d*.0 								|
	    		|         P6          |  d*.0f or d*.0 -> d* 		  			| d*.0f or d*.0 -> d* 						|
	    		|         P7          |  float F() -> (double) F() 	  			| float F() -> (double) F() 				|
	    		|         P8          |  double F() -> (float) F() 	  			| double F() -> (float) F()					|
-----------------------------------------------------------------------------------------------------------------------------
Exception   	|         E0          |  if(rtIsNaN(X)) -> if(!rtIsNaN(X)) 		| if(isnan(X)) -> if(!isnan(X))				|
handling    	|         E1          |  if(rtIsInf(X)) -> if(!rtIsInf(X))		| if(isinf(X)) -> if(!isinf(X))				|
				|         E2          |  if(!rtIsNaN(X)) -> if(rtIsNaN(X)) 		| if(!isnan(X)) -> if(isnan(X))				|
				|         E3          |  if(!rtIsInf(X)) -> if(rtIsInf(X)) 		| if(!isinf(X)) -> if(isinf(X)) 			|
				|         E4          |  insert if statment - check divide by 0	| insert if statment - check divide by 0	|
				|         E5          |  remove if statment - check divide by 0	| remove if statment - check divide by 0	|
				|         E6          |  miultiply denominator by zero			| miultiply denominator by zero				|
-----------------------------------------------------------------------------------------------------------------------------	
Time and Space	|         T0          |  dataype of time is multiplied by 1000	| dataype of time is multiplied by 1000		|
				|         T1          |  dataype of time in ifstmt() is negated	| dataype of time in ifstmt() is negated	|
				|         T2          |  variable of time is multiplied by 1000	| variable of time is multiplied by 1000	|
				|         T3          |  variable of time in ifstmt() is negated| variable of time in ifstmt() is negated	|
				|         T4          |  for all space variables present in 	| for all space variables present in 		|
				|		  ..	      |	 spacevariables.txt						| spacevariables.txt						|
				|		  Tn	      |											|											|
-----------------------------------------------------------------------------------------------------------------------------		


		

