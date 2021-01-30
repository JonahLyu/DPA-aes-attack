# Applied Cryptography 2019-2020

-------------------------------------------------------------------------------
Author: Jonah Lyu

**Citation**

[

1. The main part of my AES implementation was borrowed from the codes in the given slide of Week 14 with the title "Implementation (AES)" on the unit page.

2. The DPA implementation has imported the following codes which compute Pearson's correlation fast:
https://github.com/ikizhvatov/efficient-columnwise-correlation

3. The implementation of stage 2 was strongly supported by the idea provided in the slide of Week 19 with the title "DPA on AES" on the unit page and also the Matlab demo on www.dpabook.org:
http://dpabook.iaik.tugraz.at/onlinematerial/matlabscripts/demo_dpa.txt

4. The masking countermeasure was strongly supported by the idea provided in the slide of Week 20 with the title "Countermeasures for AES" on the unit page.

5. The S-box constant lookup table was linked with the data in the wiki:
https://en.wikipedia.org/wiki/Rijndael_S-box


]

**Documentation**

[

Stage 1

I have tried both AES implementations with S-box and T-table. T-table AES is able to work efficiently with lower latency but higher footprint compared with the general S-box AES. But then, when I reached the stage 3, I found the masking countermeasure is not preferably suitable for the T-table version because it is more costly due to re-computation of the t-tables. So I finally chose the S-box AES implementation.

To run the s-box version in the emulator:

	> make emulator-target
	> make putty-emulator

note: input COMMAND_CHECK (0X02) in the emulator to run my own AES test:
	> 00:02


There are some techniques used to improve the compute efficiency:
* Replace the s-box function with a lookup table. (256 bytes)
* Replace multiplication function with the lookup tables as we only need to compute the multiplication with constant 2 and 3. (256 bytes * 2)
* Remove some repetitive operations.

To demonstrate my effort in researching both ways of AES implementation, the t-table implementation files were also submitted with the name "target_t_table.c/h". 

To run the t-table version in the emulator:
	
	> make emulator-target_t_table
	> make putty-emulator-target_t_table  



Stage 2

The attack.py implements DPA successfully with assisted data traces. The default setting takes 200 traces into account to do the statistical analysis and it tries to locate the first s-box in the samples. With the correct input of trace data, the default setting can finish the key recovery in several seconds (ignoring the time for data read in). 

Python 3 was used so it needs to run the command like this (if python 3 is not the default one):
	> python3 attack.py traces.dat

It is easy to edit the global variables in the main to change the default settings:

	numTraces = 200
	startSample = 3000
	numSamples = 5000

These indicate: reading 200 data traces and considering sample points from 3000 to 8000 (therefore, 5000 sample in total). Since there is only one given data trace to test, I prefer not to use any argument flags to keep the implementation neat. But the code can be extended easily to have more self-contain features with physical board hopefully. 



Stage 3

There are two countermeasures used:

1. Masking
	* In each round, all masks are updated. S-box masks will pick two different values from the random source. And the mix column mask will take the last output masks as input.
	* Masks are stored in two arrays: mask[5] and mask_p[5], the first 4 data are corresponding to four unique column masks. The fifth data is corresponding to the S-box mask.
	* The masked S-box and mix column lookup tables will be computed each round.
	* This countermeasure is able to remove the dependency between intermediate values and power consumption. Therefore, the first-order DPA can not succeed theoretically. 


2. Hiding by inserting extra operations
	* Some operations such as re-masking procedures and re-computing lookup tables were inserted into the general AES round loop.
	* This prevents the attacker relating the trace with particular operations in a normal way since all operations are shifted in time.




Other Techniques

Beyond the general requirement, there are more techniques used but not mentioned previously, I have:

1. used Jupiter notebook and NumPy library to plot the correlation peak diagrams among all guessed key. This is helpful to evaluate the results visually.

2. added timer implementations in both c and python codes to test the efficiency of computation as there is no way to acquire power analysis physically. Even though the time may differ due to the platform, the progress in latency can reflect the benefit of algorithms.


3. also tried to use a parallel programming library (MPI for python) to compute the correlation. I finally decide to abandon this because it is highly dependent on the platform and requiring a special binary (mpiexec). But theoretically, it should bring a faster performance overall (as the 16 bytes guess in the key are totally independent with each other).



]

-------------------------------------------------------------------------------
