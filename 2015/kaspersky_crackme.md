# Kaspersky Crackme
A simple crackme task for admission to Kaspersky SafeBoard traineeship http://safeboard.kaspersky.ru/

We've got KasperskyCrackme.exe, console application which we're asked to crack. After some IDA researching we can notice, that it takes 2 or 3 command line params. So:
```
D:\KasperskyCrackme>KasperskyCrackme.exe 1 2
[-] Wrong!
```

Lets take a closer look.

First of all, open .exe with DIE. It says the compiler is MS VC++ 2013. Not packed, good. 

Load exe into IDA Pro 6.1 with HexRays (sometimes it makes RE much simplier) and search for program main func. As it isn't obfuscated, the 'main' will be placed after some 'call GetCommandLineA' (int main(char[] argv, int argc) - program need to get those values before passing them as arguments to function).

Tracing the code and some instructions before 

```
push    esi                             ; uExitCode
```

(program exit point) we'll see 

```
push    eax
push    dword_11497FC ; argv
push    dword_11497F8 ; argc
call    sub_11312A0
```

So 'sub_11312A0' is our program main func. Double-click at and then Tab. "Pseudocode-A" window should now open. Such pseudocode is much easier to read than assembler instructions.

I renamed some functions and variables and now it is fully clear.

https://gist.github.com/bir0bin/2038eca0883f7fba43a0

After reading the code, we can restore the hashing algorythm. Here is some python code:

```
alphabet = '0123456789-.@_qwertyuiopasdfghjklzxcvbnmQWERTYUIOPASDFGHJKLZXCVBNM'

def get_khash(email):
    khash = 0
    for char in email:
        khash = (khash * 66 + alphabet.index(char)) % (2 ** 32)
    return khash
```

If you're not familar with python: for each symbol in email it search the position in alphabet string where this symbol is placed. Hash value is setted as `hash = hash * 66 + position`.

To check whether password correct or not, password is multiplied with 31337 and compared with hash value, if it matches, then password is correct.

First of all i tried to restore email for password 1 (so the hash will be exactly 31337):
`31337 = **53** + 66 * (**12** + 66 * **7**)`

So lets find chars with corresponding positions highlited in bold in reversed order: 7 12 53

That's '7@F'. Trying:

```
C:\KasperskyCrackme>KasperskyCrackme.exe 7@F 1
[+] Correct!
```

Ok! We're on the right way. Now we need to find a way to modify our email to be hashed with value divided by 31337.

I writed a python script that bruteforces 4 character suffix for my email and gives me valid variants.

https://gist.github.com/bir0bin/df6ccaf07d7bee4e1404

In about 1 sec it found the variant which was valid. 

## Done? Maybe not.

I think my solution is ugly, and it was possible to modify character case to find corresponding email. In my script 'charcase_mutator' does so, but it is very very slow. 

Shall I need to use an SMT solver?
