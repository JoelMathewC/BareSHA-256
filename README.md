# BareSHA-256
This repository conatins all the deliverables related to the mini project of CS4035D Computer Security.

## Abstract
The Secure Hash Algorithm 256 (SHA-256) is a widely used cryptographic hash function that produces a fixed-size output of 256 bits, commonly known as the SHA256 checksum. Despite its widespread use, few cryptographers delve into the low-level implementation details of the SHA256 checksum algorithm.

This project aims to develop a barebone implementation of the SHA256 checksum, using only basic programming constructs and no in-built libraries. The implementation will be tested for correctness and performance against known test vectors.

Furthermore, this project will explore the vulnerability of the SHA256 checksum algorithm to a length extension attack. In this type of attack, an attacker who has access to the SHA256 checksum output of a message can append additional data to it, without knowing the message or the key used in the original checksum. The project will also simulate a length extension attack on the SHA256 checksum implementation and evaluate its security. This project will provide a better understanding of the inner workings of the SHA256 checksum algorithm and its limitations, and will be useful for researchers, developers, and practitioners in the field of cryptography.

Avalanche effect which refers to the degree of change in the output of a cryptographic function due to a small change in the input will also be observed through this project.

## Learnings
- Constants: The first 32 bits of the fractional parts of the cube roots of the first 64 prime numbers is defined. As per reading, we found that this approach is used in some cryptographic applications, specifically in the construction of the SHA-256 hash function. These values are used to "mix" the input message in a way that makes it difficult to reverse-engineer the original message from the output hash. The specific choice of the first 32 bits of the fractional part of the cube roots of the first 64 prime numbers was likely made for a few reasons.
    * Cube roots were chosen because they provide a good balance between randomness and predictability, making them suitable for use in a cryptographic algorithm. 
    * Using the fractional part of the cube roots ensures that the resulting values are uniformly distributed between 0 and 1, which is important for maintaining the security properties of the algorithm. 
    * Using the first 64 prime numbers ensures that the resulting values are large and unlikely to repeat, further enhancing the security of the algorithm.

## Implementation of SHA256
### To run SHA256 checksum of a file:
#### Help menu:
```console
$ python3 sha256.py -h
```
```console
usage: sha256.py [-h] -f F

options:
  -h, --help  show this help message and exit
  -f F        Name of the file to find the checksum
```
#### Run with a test file:
```console
$ python3 sha256.py -f tests/test1.pdf
```
```console
7e2903a8c60bf957824c330707617e8cc32283b5287bb9362acb2f45550810c1
```

## Testing of SHA256
Using the command line utility, ```sha256sum```, we can compare the sha256 hash generated by this implementation and verify if it matches.
Download Nessus via:
```console
curl --request GET \
  --url 'https://www.tenable.com/downloads/api/v2/pages/nessus/files/Nessus-10.5.0-ubuntu1404_amd64.deb' \
  --output 'tests/Nessus.deb'
```
```console
$ ./test.sh
```
```console
Passed: tests/test1.pdf Time taken: 3.989371038 seconds
Passed: tests/test2.txt Time taken: .062629790 seconds
Passed: tests/Nessus.deb Time taken: 299.809199438 seconds
```
Here, two files are given as test cases and the hash generated by our implementation from scratch matches the one generated by ```sha256sum```.

## Length Extension Attack (LEA)
This is an attack on hash functions based on the Merkle-Damgard Scheme. For a given message `m`, `H(m)` can be calculated where `H` is the hash function. LEA allows us to use knowledge of `H(m)` and the `len(m)` to calculate `H(m||e)` where || represents concatenation and `e` is an extension to the message. Essentially without knowledge of m, the has of extended messages of m can be calculated. The use of hash functions susceptible to LEA can have serious security concerns.

To illustrate the use of this attack the concept Message Authentication Code and Merkle-Damgard Scheme has been explained below.

### Message Authentication Code (MAC)
1. Used to authenticate origin and legitamacy of data (m).
2. The shared key (s) is known only to the sender and the receiver and hence they can only compute the hash.
3. The hash h(m || s) is appended to the message when transmitted.
4. The receiver can verify the origin/legitamacy of data.
5. Using LEA susceptible hash functions to generate the MAC is highly insecure.

```
--------------                                     ----------------
|            |          m || h(s||m)               |              | 
|   Sender   |      --------------------->         |   Receiver   | 
|            |                                     |              |
--------------                                     ----------------

m: Message
s: shared secret
```

### Merkle Damgard Construction
1. This scheme is used to build a collision-resistant iterated hash function from a collision-resistant compression function.
2. The compression function is a hash function that takes n-bits and m-bits fixed- size inputs. 
3. For hashing, the original message is appended with padding (0x80) and then as many 0's as necessary are added till only 8 bytes remain. These 8 bytes are used represent the size of message in bits.
4. The resulting message can thus be split into units of n-bit length. Before hashing, an m-bit digest is chosen as the initial input to the compression function, along with the first n-bit block. This digest is referred to as an initial vector or initial value. 
5. The compression function outputs an m-bit digest which is passed as input to the compression function again along with the next block of n-bits. 
6. This process repeats till all the blocks of the message are processed to give a final m-bit digest.
7. **Vulnerability:** The output of the hash function can be passed to the compression function along with another n-bit block to obtain the hash for the extended message.

### LEA on SHA-256
```
--------------                       ----------------                           ----------------
|            |     m || h(s||m)      |              |   (m||e) || h(s||m||e)    |              |
|   Sender   |   --------------->    |     MITM     |   ------------------->    |    Receiver  |
|            |                       |              |                           |              |
--------------                       ----------------                           ----------------

m: Message
s: shared secret
e: extension to message
```

### To run LEA Attack on SHA256:
#### Help menu:
```console
$ python3 lea.py -h
```
```console
usage: lea.py [-h] -m M -s S -e E

optional arguments:
  -h, --help  show this help message and exit
  -m M        Message to be hashed
  -s S        The secret shared by both sender and receiver
  -e E        The extension to the message
```
#### Run test:
```console
$ python3.9 lea.py -m "user=joel&role=user" -s "hello" -e "&role=admin"
```
```console
Original Message to be hashed: hellouser=joel&role=user
MAC for Original Message: cb8616e7d5e3d5195b5dd9d7c57b4d2ed430ddba3edf5396716fdcc3916fd4b1

--------------------------------------------------

Extended Message to be hashed: hellouser=joel&role=user\x80\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\xc0&role=admin

MAC for Extended Message: ff901d5e20f79d990badd0ab7ba1c94f26340ecb235c8644615d747e0c54d227
MAC for Extended Message with LEA attack: ff901d5e20f79d990badd0ab7ba1c94f26340ecb235c8644615d747e0c54d227

--------------------------------------------------

(LEA successfull) The two hashes are identical
```

## Team members
|S.L. No.| Name | Roll number | GitHub ID |
| ----- | -------- | -------- | -------- |
|1|Joel Mathew Cherian|B190664CS|[@JoelMathewC](https://github.com/JoelMathewC)|
|2|Pavithra Rajan|B190632CS|[@Pavithra-Rajan](https://github.com/Pavithra-Rajan)|
|3|Cliford Joshy|B190539CS|[@clifordjoshy](https://github.com/clifordjoshy)|


