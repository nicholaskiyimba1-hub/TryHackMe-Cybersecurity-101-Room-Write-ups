# CyberChef: The Basics

## Introduction

CyberChef is a web-based tool used to analyse and transform different types of data. It is useful in areas such as cybersecurity, digital forensics, incident response, and data analysis.

It provides many operations for working with data, including encoding, decoding, encryption, decryption, data conversion, extraction, and other transformations.

CyberChef can also be downloaded and run locally. This allows it to be used offline through a web browser, which can be useful when working with sensitive data that should not be uploaded to an external service.

## Task 1: Introduction to CyberChef

The first task introduced me to CyberChef and its purpose.

I learned that CyberChef can be used to perform different operations on data. Some of these operations include:

- Encoding
- Decoding
- Encryption
- Decryption
- Data conversion
- Data extraction
- Hashing
- Compression and decompression

Rather than being limited to one specific purpose, CyberChef provides many different operations in one interface.

This makes it useful when analysing data that may initially appear unfamiliar or unreadable.

## Task 2: Using CyberChef Offline

The second task showed that CyberChef can be used both online and offline.

Although CyberChef is primarily a web application, its files can be downloaded and run locally. When using the offline version, I can still interact with CyberChef through a web browser.

This is useful when working with potentially sensitive information because the data can be processed locally instead of being sent to an online service.

## Task 3: Understanding the CyberChef Interface

The third task introduced the main parts of the CyberChef interface.

The main areas I learned about were:

- Operations
- Recipe
- Input
- Output

There is also a **Bake** button used to process the recipe against the input.

### Operations

The Operations section contains the different transformations that CyberChef can perform.

Examples include encoding, decoding, extracting information, converting data formats, and working with dates and times.

### Recipe

The Recipe is where the operations are placed and arranged.

A recipe can contain one operation or several operations that are performed in sequence.

For example:

```text
Input
   ↓
Operation 1
   ↓
Operation 2
   ↓
Output
```

This makes the Recipe an important part of CyberChef because it defines what should happen to the input data.

### Input

The Input section contains the data that I want CyberChef to process.

The input can contain different types of data, such as text or other files supported by the operation being used.

### Output

The Output section displays the result produced after CyberChef processes the input using the selected recipe.

The result can also be copied to the clipboard or saved as a file when appropriate.

### Bake

The Bake button processes the recipe against the input and produces the resulting output.

## Task 4: Choosing the Right Operation

This task helped me understand how CyberChef can be used when investigating data that initially appears to be meaningless or unreadable.

For example, during a forensic investigation, I might encounter a string that looks like gibberish. Instead of immediately assuming that it is encrypted, I need to first examine the data and consider what type of representation it might be.

My general approach is:

1. **Determine the objective**

   I first identify what I am trying to find or accomplish.

2. **Examine the input**

   I place the suspicious or unfamiliar data into CyberChef's Input section.

3. **Identify the likely data format**

   I examine the characteristics of the data and consider whether it could be encoded, encrypted, hashed, URL-encoded, or represented in another format.

4. **Choose an appropriate operation**

   I select an operation that matches my understanding of the data.

5. **Bake the recipe**

   I process the input and examine the resulting output.

6. **Validate the result**

   If the output is not meaningful, I reassess the input and the operation instead of blindly assuming that the transformation was correct.

This showed me that CyberChef is not simply a tool for randomly trying different operations. Understanding the data first can help me choose a more appropriate operation and save time.

## Task 5: CyberChef Operations

The fifth task introduced me to several categories of CyberChef operations and gave me more practical experience with how they can be used.

### Extractors

The Extractors operations can be used to identify specific types of information within a larger piece of data.

Some examples I worked with were:

- **Extract IP addresses:** Identifies IPv4 and IPv6 addresses in the input.
- **Extract URLs:** Identifies URLs contained within the input.
- **Extract email addresses:** Identifies email addresses contained within the input.

This can be useful when a large block of text contains information that I need to extract quickly.

### Date and Time Operations

I also learned about Unix timestamps.

A Unix timestamp represents a point in time using the number of seconds elapsed since the Unix epoch.

CyberChef provides operations that allow timestamps and normal date and time representations to be converted between the two formats.

For example:

```text
Unix timestamp
      ↓
Human-readable date and time
```

and:

```text
Human-readable date and time
      ↓
Unix timestamp
```

This can be useful when analysing timestamps that appear in logs or other technical data.

## Data Formats and Decoding

Another area I explored was converting data between different formats.

CyberChef provides operations for working with different encodings and representations, including Base64.

The important lesson I learned is that **encoding is not the same as encryption**.

Encoding changes the representation of data so that it can be stored or transferred in another format. It is not intended to keep the information secret.

Encryption, on the other hand, is designed to protect information from unauthorized access and normally involves a cryptographic key.

## URL Decoding

I also learned how URL decoding works.

URLs can contain percent-encoded characters. For example:

```text
https%3A%2F%2Fexample.com
```

can be decoded into:

```text
https://example.com
```

CyberChef's URL Decode operation converts these percent-encoded characters back into their corresponding characters.

This can be useful when analysing URLs or other data containing URL encoding.

## Understanding Concatenation

During the room, I came across the term **concatenate**.

In computing, concatenation means joining separate pieces of data together to form one continuous sequence.

For example:

```text
010011
+
100100
```

can be concatenated into:

```text
010011100100
```

Understanding this became important when learning how Base64 encoding works.

## Understanding Base64 Encoding

One of the concepts I explored was how text can be converted into Base64.

To understand the process, I first need to represent the characters using their ASCII values.

For example:

```text
N = 78
I = 73
K = 75
O = 79
```

These decimal values can then be represented as 8-bit binary numbers:

```text
N = 01001110
I = 01001001
K = 01001011
O = 01001111
```

The binary values are then concatenated:

```text
01001110010010010100101101001111
```

The resulting binary sequence is divided into groups of **6 bits**.

```text
010011
100100
100101
001011
010011
110000
```

Each 6-bit group is then converted from binary into a decimal value.

For example:

```text
010011
```

has the positional values:

```text
32  16  8  4  2  1
```

The `1`s occur at 16, 2, and 1:

```text
16 + 2 + 1 = 19
```

Therefore:

```text
010011 = 19
```

The resulting decimal value is then used with the Base64 index table to determine the corresponding Base64 character.

The overall process can therefore be represented as:

```text
Text
 ↓
ASCII values
 ↓
8-bit binary
 ↓
Concatenate binary
 ↓
Split into 6-bit groups
 ↓
Convert each group to decimal
 ↓
Use the Base64 index
 ↓
Base64 output
```

### Padding

Sometimes the binary data does not divide evenly into groups of six bits.

When this happens, additional zeros can be added to complete the final group. This is called **padding**.

Base64 may also use `=` characters as padding in the final encoded output when required.

## ASCII Values

While learning the Base64 process, I also learned a useful way of remembering the decimal values of uppercase English letters in ASCII.

The uppercase alphabet begins at:

```text
A = 65
B = 66
C = 67
```

The values then increase sequentially until:

```text
Z = 90
```

Therefore, instead of memorising every letter individually, I can use the sequence starting from 65.

For example:

```text
A = 65
B = 66
C = 67
D = 68
...
Z = 90
```

This is useful when manually working through character-to-ASCII conversions.

## Practical Investigation Approach

The main practical lesson I took from this room is that CyberChef can help break down unfamiliar data during an investigation.

When I encounter data that I do not immediately understand, I should not simply assume that it is encrypted.

A better approach is:

```text
Examine the data
      ↓
Identify its characteristics
      ↓
Form a hypothesis about its format
      ↓
Select an appropriate CyberChef operation
      ↓
Process the data
      ↓
Examine and validate the result
```

For example, a string containing `%` followed by hexadecimal values may suggest URL encoding, while a string with characteristics associated with Base64 may suggest that Base64 decoding is worth testing.

The important part is connecting the appearance and characteristics of the data to the operation being selected.

## Key Takeaways

After completing the **CyberChef: The Basics** room, I learned:

- CyberChef is a web-based data analysis and transformation tool.
- CyberChef can also be downloaded and used offline through a browser.
- A **Recipe** contains the operations that will be applied to the input.
- The **Input** contains the data being analysed or transformed.
- The **Output** displays the result of the recipe.
- **Bake** processes the recipe against the input.
- Data should be examined before selecting an operation.
- Encoding, encryption, and hashing are different concepts.
- Extractors can identify information such as IP addresses, URLs, and email addresses.
- Unix timestamps can be converted between timestamp and human-readable formats.
- URL decoding can convert percent-encoded data back into readable characters.
- Concatenation means joining separate pieces of data together.
- Base64 works with groups of **6 bits**, not 6 characters.
- Binary numbers can be converted to decimal using powers of two.
- Base64 uses decimal values from 0 to 63 to map binary groups to its character set.
- Padding is used when the data does not fit evenly into the required Base64 structure.

## Conclusion

This room gave me a practical introduction to CyberChef and showed me how it can be used to analyse, decode, convert, and extract information from different types of data.

The most useful lesson for me was learning to examine unfamiliar data before choosing an operation. Instead of treating every unreadable string as encryption, I can look for characteristics that suggest encoding or another data representation and then select an appropriate CyberChef operation.

This provides a useful foundation for using CyberChef in future cybersecurity and digital forensics investigations.