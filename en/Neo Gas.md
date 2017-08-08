# Neo Gas

### Introduction of Neo Gas

1. Brief Introduction  
  Neo Gas represents the right to use the Neo Blockchain. There will be 100 million Neo Gas in total. Neo Gas are generated along with every new block. The issuance will slow down according to a set slowly-decreasing pace, while Neo Gas will go through a generating process to grow from zero to 100 million. Once Neo is acquired, Neo Gas will be generated in the system following the algorithms.

2. Basic Concept  
  Every Neo has two states, unspent and spent. Each Gas has two states too, which are available and unavailable. A life cycle of a Neo starts from being transferred into an address and ends with being transferred out of that address. When it is transferred in, its state becomes unspent while when it is transferred out, its state becomes spent. When Neo is at the unspent state, its generated Gas will be in unavailable state, in another word, undrawable; when Neo is at the spent state, its generated Gas will be in available state, so the users can draw them.

3. Mathematical Model  
  t_start = the moment that Neo goes into the state of unspent  
  t_end = the moment that Neo goes into the state of spent  
  Δt_const = t_end - t_start  
  Δt_var = t - t_start  
  Available Gas = f(neo_amount, Δt_const)  
  Unavailable Gas = f(neo_amount,  Δt_var)  
  Please note that:  
  Δt_const is fixed, thus the available Gas is of a fixed amount too. And this amount is a function of the amount of Neo held by the user and the duration between the moments that he or she transferred this amount of Neo into and out of his or her address. The current time is a variable, so the amount of the unavailable Gas also grows through time, which means it is a variable.

4. Usage  
  - To pay to the Neo blockchain for recording 
  - To pay to the Neo blockchain for additional services



Edited by Peter Lin (https://github.com/PeterLinX/Introduction-to-Neo)
