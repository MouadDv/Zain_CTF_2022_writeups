# Zain_CTF_2022_writeups

Challenges covered in this writeup are: equat1onz, Email Address, flipflap, camera, Duck, Fish

## equat1onz

In this challenge we were given a zip containing a python script used to encrypt the flag.

```
#!python3 
from math import *
with open('flag.txt','r') as fg:
	flag=fg.read()

def calc(x,val):
	return {
		0: lambda x: (x+1)/(x-1),
		1: lambda x: 984512/(x-69964)+69964
	}[val](x)

def get_val(x,i):
	if ( i % 2 == 0 ):
		return calc(x,1)
	else:
		return calc(x,0)
	
enc=[]
i= 0
for L in flag:
	enc.append(get_val(ord(L),i))
	i+=1


print(enc)
#[69949.90776101458, 1.0186915887850467, 69949.90876951923, 1.0196078431372548, 69949.90352371817, 1.0277777777777777, 69949.90614710683, 1.017094017094017, 69949.90594534237, 1.0186915887850467, 69949.90473463427, 1.017391304347826, 69949.90715584248, 1.018181818181818, 69949.90614710683, 1.024390243902439, 69949.90917288068, 1.0208333333333333, 69949.90534001432, 1.04, 69949.90917288068, 1.02, 69949.90876951923, 1.0175438596491229, 69949.90392737999, 1.0212765957446808, 69949.90574357212, 1.02, 69949.90876951923, 1.0175438596491229, 69949.90392737999, 1.0212765957446808, 69981.0168870452, 1.0212765957446808, 69949.90513822675, 1.0178571428571428, 69949.90473463427, 1.0192307692307692, 69949.90372555197, 1.0166666666666666, 69949.90312003322, 1.2222222222222223]
```

As you can see, in the bottom of the script there is an array, all we need to do is to try encoding every character and compare the returned value with the flag array value.

```
#!python3 
from math import *

def calc(x,val):
	return {
		0: lambda x: (x+1)/(x-1),
		1: lambda x: 984512/(x-69964)+69964
	}[val](x)

def get_val(x,i):
	if ( i % 2 == 0 ):
		return calc(x,1)
	else:
		return calc(x,0)
	
flagarr = [69949.90776101458, 1.0186915887850467, 69949.90876951923, 1.0196078431372548, 69949.90352371817, 1.0277777777777777, 69949.90614710683, 1.017094017094017, 69949.90594534237, 1.0186915887850467, 69949.90473463427, 1.017391304347826, 69949.90715584248, 1.018181818181818, 69949.90614710683, 1.024390243902439, 69949.90917288068, 1.0208333333333333, 69949.90534001432, 1.04, 69949.90917288068, 1.02, 69949.90876951923, 1.0175438596491229, 69949.90392737999, 1.0212765957446808, 69949.90574357212, 1.02, 69949.90876951923, 1.0175438596491229, 69949.90392737999, 1.0212765957446808, 69981.0168870452, 1.0212765957446808, 69949.90513822675, 1.0178571428571428, 69949.90473463427, 1.0192307692307692, 69949.90372555197, 1.0166666666666666, 69949.90312003322, 1.2222222222222223]
dec=""
x = 0

for flagchar in flagarr:
    i = 10
    while (i < 127) :
        found = 0
        if (flagchar == get_val(ord(chr(i)), x)):
            dec += chr(i)
            found = 1
            x+=1
            break ;
        i+=1
    if (found == 0):
        dec += '?'
        x+=1

print(dec)
```
OUTPUT:
```
flag{InvolutionS_ar3_easy_peasy_?_squizy}
```
We can see that the flag is still missing one character ever after trying all ascii character. a quick search for lemon on utf.

```
flag{InvolutionS_ar3_easy_peasy_🍋_squizy}
```
