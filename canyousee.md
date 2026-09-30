# divys_tech_tips — CanYouSee

## Challenge Overview

The challenge provides a **binary file** and an **image**. The objective is to inspect the provided files and find the hidden flag.

---

## 1. Initial Enumeration

Since the challenge involves an image, the first step is to inspect its metadata.

We can use `exiftool`:

```bash
exiftool ukn_reality.jpg
```

This displays various metadata associated with the image.

Most of the fields appear to be normal JPEG metadata, but one field immediately stands out:

```text
Attribution URL : cGljb0NURntNRTc0RDQ3QV9ISUREM05fM2I5MjA5YTJ9Cg==
```

This value does not look like a normal URL.

Instead, it consists of characters commonly found in **Base64-encoded data**.

---

## 2. Identifying the Encoding

The string:

```text
cGljb0NURntNRTc0RDQ3QV9ISUREM05fM2I5MjA5YTJ9Cg==
```

has the typical characteristics of Base64:

* It uses the Base64 character set.
* It ends with `==`, which is common Base64 padding.
* The decoded output is likely to contain readable text.

This suggests that the metadata itself may contain the flag encoded in Base64.

---

## 3. Decoding the Hidden Data

We can decode the string directly from the terminal using the `base64` utility:

```bash
echo "cGljb0NURntNRTc0RDQ3QV9ISUREM05fM2I5MjA5YTJ9Cg==" | base64 -d
```

The output is:

```text
picoCTF{ME74D47A_HIDD3N_3b9209a2}
```

---

## 4. Flag

The flag is:

```text
picoCTF{ME74D47A_HIDD3N_3b9209a2}
```

---

## 5. Takeaway

An important lesson from this challenge is that **metadata can contain more than just ordinary image information**.

When working with CTF files, it is worth checking:

```bash
exiftool <file>
```

before attempting more complicated techniques.

In this challenge, a binary file was also provided, but it was not required to obtain the flag. This may be an example of an **unintended solution path**, where inspecting the image metadata provides a much more direct route to the flag.

### Solution Chain

```text
ukn_reality.jpg
      ↓
exiftool
      ↓
Attribution URL
      ↓
Base64-encoded string
      ↓
base64 -d
      ↓
picoCTF{ME74D47A_HIDD3N_3b9209a2}
```
