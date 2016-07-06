+++
title = "Encrypted Tweets"
date = "2011-06-22"
tags = ["aes", "cryptography", "cypher", "greasemonkey", "hash", "javascript", "security", "sha1", "twitter"]
+++

As a cryptography course project, me and
[@laisandrade](http://twitter.com/laissandrade) developed a twitter
extension which allows us to tweet in a way that only a group of
followers can decrypt and understand my tweet. The goal is to secure
broadcasting tweets to a group of followers and avoid anyone else to
understand it's content. This definition of security is safe against an
[eavesdropping attack](https://secure.wikimedia.org/wikipedia/en/wiki/Eavesdropping),
which is commonly considered secure for cryptography, what is not true.
Another concerns emerges when we consider other attacks or ways of
extracting informations from the cyphertext, as an example the
[Known-Plaintext Attack](https://secure.wikimedia.org/wikipedia/en/wiki/Known-plaintext_attack).

![Configuration of the Encrypted Tweets problem - Alice securely broadcasting messages to a 
specific group](/img/scheme.png)

In this project we should also consider other security issues such as
forgery attacks and secure information storage. The only real world
problem outside the project's scope was the private key distribution. We
assume that the user who wants to share it's keys to his followers could
use other mechanisms such as a Diffie-Hellman key exchange or using a
trusted key broker. The full project's specification can be found
[here](http://crypto.stanford.edu/%7Edabo/cs255/hw_and_proj/proj1.pdf).
For doing the task we used the
[Grease Monkey](https://addons.mozilla.org/en-US/firefox/addon/greasemonkey/)
Firefox's plugin. This plugin allow us to run custom javascript scripts
in specifics web pages, so we can add functionalities to known web
pages. In fact, we used a
[script stub](http://crypto.stanford.edu/%7Edabo/cs255/hw_and_proj/cs255.user.js) for doing it.
This script adds a encrypt tweet functionality as well as
a key manager at password settings page. It also gives a function to
store (unsafely) the user information. Was also our job to make it
secure. We assumed that the script couldn't be cracked, the focus where
we were concerned was about the information safety outside the script
execution memory.

 ![Encrypt button and Group selection close to Tweet button](/img/tweet.png)
  

Encryption Specification
------------------------

Encripted Message:
```
+--------+-----+--------------------------------------+
| "aes:" | Ctr | Extended-AES(Content, Ctr, GroupKey) |
+--------+-----+--------------------------------------+
```
 
* The Ctr is the base counter for the CTR block chaining cypher.
  Every message picks a random Ctr.
* The group key is the key to the group whom members are the only who should understand the
  message.
* The Extended-AES function will be detailed after.
* the "aes:" indicator is not a security threat due to Kerckhoffs's Principle.

Content:
```
+---------------+---------+
| sha1(Message) | Message |
+---------------+---------+ 
```

* The sha1 cryptographic hash is used to check the integrity of the stored keys and avoid 
  modifications on the massage bypass as a real message.

* The sha1 produces a 160 bits length message, which is convertes into a base64 string to send 
  it over twitter.

* This Message Authentication Code (MAC) is necessary for Chosen Ciphertext Security
  (CCA Security).  Message: the original message 

For our Extended-AES implementation, we used a
[Cypher Block Chaining](https://secure.wikimedia.org/wikipedia/en/wiki/Cipher_block_chaining) 
to extend the [AES](https://secure.wikimedia.org/wikipedia/en/wiki/Advanced_Encryption_Standard)
block cypher, that we use as a
[block cypher](https://secure.wikimedia.org/wikipedia/en/wiki/Block_cypher).
To do this we used a Counter (CTR) chaining to extend the cypher.
Note that the CTR mode uses a base counter (Ctr) as an
[Initialization Vector](https://secure.wikimedia.org/wikipedia/en/wiki/Initialization_vector)
(IV) which also must be transmitted to enable to be decrypted. This IV
is chosen at random by our
[pseudorandom generator](https://secure.wikimedia.org/wikipedia/en/wiki/Pseudorandom_generator).
If the IV is not truly random, an attacker may predict the IV sequence and get more
information from the message.
A real problem faced by pseudorandom generators are when the seed selection are not truly
random. To handle this, the script stub creates a seed using the mouse
positions at the first seconds of running (an entropy test is also used
to check the randomness of the seed).

![](/img/crt_encryption)

When someone receives an "aes:" tagged message, the script checks the
author of the message and get the key provided by him. To decrypt, we
also use the Extended-AES since it is symmetric (due to XOR
reversibility property). At last the SHA-1 hash is confronted with the
message to check if the key is correct or the message suffered a forgery
attempt.

 To store the groups keys we used the grease monkey's key value storage
(GM_setValue and GM_getValue functions). We store and recover the
values from the key `"twit-key-" + login`, which is stored as plaintext
in the hard drive. To avoid the stealth of this information from other
users, virus or worm we added an encryption layer over it. The stored
data representation developed was this one:
 
![Keys management screen](/img/twitter2.png)
  
Safe Storaging Specification
----------------------------

Stored Data: AES(Data, MasterKey)
MasterKey: User secret key to access all the other keys. This key requires user input every 
time it logs in.

Data:
```
+---------------------+-----------------+
|   sha1(GroupKeys)   |   Groups Keys   |
+---------------------+-----------------+
```

* This cryptographic hash is used to check the integrity of the stored keys.
* A malicious agent may override the values on the keys and may set the keys he can decrypt.  

Groups Keys:
```
+----------+----------+----------+----------+---------+----------+----------+
|   Item   |   "$$"   |   Item   |   "$$"   |   ...   |   "$$"   |   Item   |
+----------+----------+----------+----------+---------+----------+----------+ 
```

Item: 
```
+----------+---------+-----------+---------+---------+
|   User   |   "$"   |   Group   |   "$"   |   Key   |
+----------+---------+-----------+---------+---------+ 
```

User: base64 string Group: base64 string Key: base64 string  

 It's important to notice that we implemented the
[SHA-1](https://secure.wikimedia.org/wikipedia/en/wiki/Sha1) hashing
algorithm to ensure the data wasn't modified while stored or
transmitted. We trust it wasn't modified due to the difficulty to crack
the AES cypher to forge a new MAC. And if the attacker try to change
bits from the stored data, we trust that SHA-1 hash collision difficulty
will be enough to ensure us that someone will be able to produce valid
message, hash pairs without looking at the plaintext. If the pair is
corrupted, then the user is alerted that someone modified his keys. By
doing this, we may disable an attacker to modify bits from our keys and
have better chances to crack the scheme. The SHA-1 also has some
theoretical weakness, however, in practice it still is a very strong
cryptographic hash function.
