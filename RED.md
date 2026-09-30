# divy's_tech_tips — RED

## Challenge Overview

The challenge provides a PNG image named `red.png`.

When opened normally, the image appears to be nothing more than a **solid red square**. This suggests that there may be hidden information embedded within the file.

---

## 1. Initial Enumeration

Since the image itself does not appear to contain anything interesting visually, the first step is to inspect the file for hidden metadata or additional information.

We can use `cat` or `exiftool`:

```bash
cat red.png
```

or:

```bash
exiftool red.png
```

While inspecting the file, we find an interesting piece of text labelled **Poem**:

```text
Crimson heart, vibrant and bold,
Hearts flutter at your sight.
Evenings glow softly red,
Cherries burst with sweet life.
Kisses linger with your warmth.
Love deep as merlot.
Scarlet leaves falling softly,
Bold in every stroke.
```

At first, this looks like ordinary text, but the wording suggests that it may contain another clue.

---

## 2. Finding the Hidden Hint

Looking at the **capitalized letters** at the beginning of each line:

```text
Crimson
Hearts
Evenings
Cherries
Kisses
Love
Scarlet
Bold
```

Taking the first letter of each line gives:

```text
C H E C K L S B
```

However, inspecting the capitalization more carefully reveals the intended clue:

```text
CHECKLSB
```

This can be interpreted as:

> **CHECK LSB**

LSB stands for **Least Significant Bit**, which is commonly used in image steganography.

This suggests that the PNG may contain hidden data encoded in its least significant bits.

---

## 3. Checking the Image for Steganography

To investigate the image's bit planes, we can use **zsteg**.

`zsteg` is a tool commonly used to detect hidden data in PNG and BMP images.

Run:

```bash
zsteg red.png
```

Among the various results, one line immediately stands out:

```text
b1,rgba,lsb,xy .. text:
"YWNhZGVteXtyM2RfMXNfdGgzX3VsdDFtNHQzX2N1cjNfZjByXzU0ZG4zNTVffQ==YWNhZGVteXtyM2RfMXNfdGgzX3VsdDFtNHQzX2N1cjNfZjByXzU0ZG4zNTVffQ==YWNhZGVteXtyM2RfMXNfdGgzX3VsdDFtNHQzX2N1cjNfZjByXzU0ZG4zNTVffQ==YWNhZGVteXtyM2RfMXNfdGgzX3VsdDFtNHQzX2N1cjNfZjByXzU0ZG4zNTVffQ=="
```

The important part is:

```text
b1,rgba,lsb,xy
```

This confirms that `zsteg` found text hidden in the **least significant bit** of the RGBA channels.

---

## 4. Identifying the Encoding

The extracted text looks like:

```text
YWNhZGVteXtyM2RfMXNfdGgzX3VsdDFtNHQzX2N1cjNfZjByXzU0ZG4zNTVffQ==
```

The ending:

```text
==
```

and the character set strongly suggest that this is **Base64** encoded data.

We can therefore decode it using the `base64` command.

---

## 5. Base64 Decoding

Using:

```bash
echo "YWNhZGVteXtyM2RfMXNfdGgzX3VsdDFtNHQzX2N1cjNfZjByXzU0ZG4zNTVffQ==" | base64 -d
```

we obtain:

```text
academy{r3d_1s_th3_ult1m4t3_cur3_f0r_54dn355_}
```

The same encoded string appears multiple times in the extracted data, so the decoded flag is repeated as well.

We only need the unique flag.

---

## 6. Flag

```text
academy{r3d_1s_th3_ult1m4t3_cur3_f0r_54dn355_}
```

## Conclusion

The solution required following a chain of clues:

```text
red.png
   ↓
Hidden poem
   ↓
CHECKLSB
   ↓
Least Significant Bit steganography
   ↓
zsteg
   ↓
Base64-looking string
   ↓
Base64 decoding
   ↓
Flag
```

The key lesson from this challenge is to **inspect seemingly uninteresting image files for metadata and hidden data**, especially when the file contains suspicious text or clues pointing toward techniques such as LSB steganography.
