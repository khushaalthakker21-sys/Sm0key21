divvy's\_tech\_tips -- ENC -- 



\## Challenge Overview



The challenge provides a Python function that transforms a string by combining every two characters into a single Unicode character:



```python

''.join(\[chr((ord(flag\[i]) << 8) + ord(flag\[i + 1]))

&#x20;        for i in range(0, len(flag), 2)])

```



The goal is to reverse this process and recover the original flag.



\---



\## Understanding the Encryption



The first step was to understand the functions used in the given code.



The important part is:



```python

(ord(flag\[i]) << 8) + ord(flag\[i + 1])

```



Since I was not familiar with `ord()` and the `<< 8` operator, I tested them using a simple Python program:



```python

flag = \["a", "b"]



print(ord(flag\[1]))

print(ord(flag\[1]) << 8)



print(bin(ord(flag\[1])))

print(bin(ord(flag\[1]) << 8))

```



This produced:



```text

98

25088

0b1100010

0b110001000000000

```



\### `ord()`



`ord()` converts a character into its integer Unicode value.



For example:



```python

ord("b")

```



gives:



```text

98

```



\### `<< 8`



The `<<` operator is a left bit-shift.



Therefore:



```python

ord("b") << 8

```



shifts the binary representation of `98` eight positions to the left.



In binary:



```text

0b1100010

```



becomes:



```text

0b110001000000000

```



This effectively moves the first character's 8 bits into the upper half of a 16-bit value.



\---



\## How Two Characters Are Combined



The original expression is:



```python

(ord(flag\[i]) << 8) + ord(flag\[i + 1])

```



The first character is shifted 8 bits to the left, and the second character is then added.



Conceptually, if we have two 8-bit values:



```text

AAAAAAAA BBBBBBBB

```



the first character is shifted left:



```text

AAAAAAAA 00000000

```



and the second character is added:



```text

AAAAAAAA BBBBBBBB

```



So two 8-bit characters are combined into one 16-bit value.



The resulting number is then passed to:



```python

chr()

```



which converts the number into a Unicode character.



This explains why the encrypted output contains unusual Unicode characters.



\---



\## Inspecting the Encrypted File



I first checked the file type of the encrypted file and found that it was a text file.



Displaying its contents produced:



```text

灩捯䍔䙻ㄶ形楴獟楮獴㌴摟潦弸形㝦㘲捡㕽

```



At first, these characters looked like Chinese/Mandarin characters.



However, since Unicode assigns numerical values to characters from many different writing systems, I suspected that these characters were actually the result of the 16-bit encoding described above.



\---



\## Reversing the Encoding



I checked the Unicode value of one of the characters:



```python

print(bin(ord("灩")))

```



which gave:



```text

0b111000001101001

```



This showed that the character's Unicode value could contain the two original 8-bit values.



Therefore, instead of combining two characters, I needed to split each Unicode value back into two 8-bit values.



The encrypted characters were stored in an array:



```python

flag = \[

&#x20;   "灩","捯","䍔","䙻","ㄶ","形","楴","獟","楮","獴",

&#x20;   "㌴","摟","潦","弸","形","㝦","㘲","捡","㕽"

]

```



I then used:



```python

result = ""



for i in range(0, len(flag)):

&#x20;   first = chr(ord(flag\[i]) >> 8)

&#x20;   second = chr(ord(flag\[i]) \& 0xff)



&#x20;   result += first + second



print(result)

```



\### Why does this work?



The original operation was:



```python

(ord(first) << 8) + ord(second)

```



To reverse the first part, we shift the combined value \*\*8 bits to the right\*\*:



```python

ord(flag\[i]) >> 8

```



This gives us the original first character.



For the second character, we only need the lowest 8 bits.



The hexadecimal value:



```text

0xff

```



is:



```text

11111111

```



in binary.



Therefore:



```python

ord(flag\[i]) \& 0xff

```



keeps only the rightmost 8 bits.



Finally, `chr()` converts both recovered numerical values back into characters.



\---



\## Final Solver



The complete solution is:



```python

flag = \[

&#x20;   "灩","捯","䍔","䙻","ㄶ","形","楴","獟","楮","獴",

&#x20;   "㌴","摟","潦","弸","形","㝦","㘲","捡","㕽"

]



result = ""



for i in range(0, len(flag)):

&#x20;   first = chr(ord(flag\[i]) >> 8)

&#x20;   second = chr(ord(flag\[i]) \& 0xff)



&#x20;   result += first + second



print(result)

```



Running the script produces:



```text

picoCTF{16\_bits\_inst34d\_of\_8\_b7f62ca5}

```



\## Flag



```text

picoCTF{16\_bits\_inst34d\_of\_8\_b7f62ca5}

```



\## Key Takeaway



The challenge was essentially reversing a simple 16-bit encoding scheme.



Two 8-bit characters were combined into one 16-bit Unicode value:



```text

first character << 8

&#x20;       +

second character

```



To reverse it:



```text

upper 8 bits → first character

lower 8 bits → second character

```



The important Python operations used were:



\* `ord()` — converts a character to its Unicode integer value.

\* `chr()` — converts an integer back into a character.

\* `<< 8` — shifts bits 8 positions to the left.

\* `>> 8` — shifts bits 8 positions to the right.

\* `\& 0xff` — extracts the lowest 8 bits.



