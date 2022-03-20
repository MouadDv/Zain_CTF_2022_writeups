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
We can see that the flag is still missing one character ever after trying all ascii character. a quick search for lemon on utf and replacing the missing character did the trick.



## Email Address

In this challenge we were given a target username and asked to get his email address.

Searching for the username in google revealed nothing, the next place i looked is github and we got a profile with one project.

https://github.com/c5oylchicrlnofu

All we have to do is to access the project commit patch.

https://github.com/c5oylchicrlnofu/MyFirstRepository/commit/9f15379fa7f92d37c6a5dda62ee9df73367709db.patch

```
flag{ch05o6wo7ribrlfrldrusp1ke@protonmail.com}
```

## Flipflap

In this challenge we were given the source code of a service, let's try to understand how this service behave.

```
#!/usr/bin/python3

from Crypto.Cipher import AES
from binascii import hexlify, unhexlify
from Cryptodome.Util.Padding import pad, unpad
import json, os
from string import ascii_lowercase, ascii_uppercase, digits, punctuation

printable = ascii_lowercase + ascii_uppercase + digits + punctuation + " "

FLAG = "flag{this_is_not_the_real_flag}"

KEY = os.urandom(16)
IV = os.urandom(16)


def encrypt(pt, key, iv):
    cryptor = AES.new(key, AES.MODE_CBC, iv)
    ct = cryptor.encrypt(pad(pt, 16))

    return hexlify(ct).decode()


def decrypt(ct, key, iv):
    ct = unhexlify(ct)

    cryptor = AES.new(key, AES.MODE_CBC, iv)
    pt = cryptor.decrypt(ct)

    return unpad(pt, 16)


def sanitize(s):
    r = b""
    for i in s:
        if chr(i) in printable:
            r += bytes([i])
    return r


def main():
    print(
        """
    1) get your ciphertext
    2) get the FLAG
    3) quit
    """
    )
    x = input("> ")
    if not x in ["1", "2", "3"]:
        print("Invalid input.")
        return
    if x == "1":
        name = input("Enter your name: ")
        if "admin" in name:
            print("You're lying, there is no admin in your name !")
            return

        obj = '{"type": "simple_user", "name": "' + name + '"}'

        ciphertext = encrypt(obj.encode(), KEY, IV)
        print(f"Here's your ciphertext : {ciphertext}")
    elif x == "2":
        try:
            ciphertext = input("Enter your ciphertext: ")
            obj = decrypt(ciphertext, KEY, IV)

            obj = sanitize(obj)  # to make sure that we have only readable chars in the name ;)

            obj = json.loads(obj)
        except:
            print("something is going wrong")
            return

        if obj["type"] == "simple_user":
            print("Poor simple user ! I have nothing for you :(")
        elif obj["type"] == "admin":
            print(f"Welcome back admin!  Here is your FLAG {FLAG}")
        else:
            print("Who are you ???")
    else:
        exit()


if __name__ == "__main__":
    print("Hello flipflap! choose one of the options:")
    
    while True:
        main()
```

1. The service read a name, the name gets checked if it contain "admin", if not it creat a json string.

```
name = input("Enter your name: ")
if "admin" in name:
	print("You're lying, there is no admin in your name !")
	return
obj = '{"type": "simple_user", "name": "' + name + '"}'
```
2. After that he encrypt the whole string using AES algorithm and outputs the cipher.

```
ciphertext = encrypt(obj.encode(), KEY, IV)
print(f"Here's your ciphertext : {ciphertext}")
```

3. If we try to get the flag, we are asked to provide the cipher and it get decrypted back to the json string.

```
ciphertext = input("Enter your ciphertext: ")
obj = decrypt(ciphertext, KEY, IV)
obj = sanitize(obj)
```

4. After that the string gets sanitized, lets look at the sanitize function

```
def sanitize(s):
    r = b""
    for i in s:
        if chr(i) in printable:
            r += bytes([i])
    return r
```

5. All this function do is iterate on the string and only pickup the printable characters.

# Exploit

We need to overwrite the type and replace its value with admin wish is easy to do.

```
Akira1337", type:"admin
```

But the first check will detect that we are trying to insert admin in the name variable.

what we can do next is to add non pritable characters to the admin so we can bypass the first check.

```
Akira1337", type:"adm[non printable chracters here]in
```

One way is to create a file and add to its content non printable characters and redirect the file content to service.

```
echo 1 >> test
printf "Akira1337\",\"type\": \"admi\00n" >> test
echo "" >> test
```

If we try 
```
python3 challenge.py < test
```
the script quit as read receive EOF signal.
One trick is to use cat.

```
cat test - | python3 challenge.py
```

![image](https://i.ibb.co/PhdHmQH/flipflap2.png)



## Camera

In this challenge we were given an image and asked to find the original.

Using an image reverse search engine reveals that the name of the place is thewillowsbnb.
Searching google we got a website but no image but we can use the wayback machine to get older versions of the website.

![original image](https://i.ibb.co/P1jC6zK/asdf.png)

all we need to do now is to get the md5sum of the image and get the model of camera using exiftool.


