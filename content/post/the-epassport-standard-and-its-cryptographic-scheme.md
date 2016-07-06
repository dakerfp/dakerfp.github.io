+++
title = "The ePassport Standard and it's Cryptographic Scheme"
date = "2013-05-30"
tags = ["epassport", "cryptography"]
+++

ePassport is the name given for the new passport specification
standardized by the [International Civil Aviation Organization](http://www.icao.int/) (ICAO). 
It was created to speed up and standardize the immigration procedures.
This standard specifies [RFID](http://en.wikipedia.org/wiki/Radio-frequency_identification)
as the communication channel between the passport and the reader. The main
feature present on it is the insertion of biometric data into the
passport for biometric checks, including facial recognition, fingerprint
and iris information. To keep this important data protected, an
interesting cryptographic layer was added in addition to the usual
document falsification mechanisms such as watermarks.

![Brazilian ePassport. One of few which has full compatibility with the ICAO specification.](/img/passaporte-br.jpg)

The name, nationality, birth date and other personal information as well
as the biometric informations are held on a contactless chip inside the
passport. This set of information are securely handled and is called the
**Machine Readable Zone** (MRZ) and the chip asserts their security.
These informations are transmitted to a ePassport reader at immigration
process. As the embedded chip has low processing power, the biometric
data is transmitted to a reader so it can be checked against the porter
of the passport.

![Anatomy of an ePassport](/img/epass.jpg)

As the physical communication layer of the ePassport broadcasts the
messages, even with the low range (less than a meter) RFID
communication. But as we know an eavesdropper can be very determined to
listen the communication ;-). I also has an electromagnetic shield on
the passport cover, however it doesn't assure to block the communication
between the chip and the outside world when it's closed, however it's an
extra obstacle. Another issue to prevent is the passport and digital
data forgery. Otherwise a falsary could get someone else passport and
insert it's biometrics information and easily bypass the immigration. As
we may see, the ePassport is a complete meal for cryptography
specialists.

Starring at this cryptographic layer we have: 

A **unique id** used as a private key. This key is hardcoded in the chip
manufacturing process and it's non traceable. Doing
[reverse engineering](https://secure.wikimedia.org/wikipedia/en/wiki/Reverse_engineering#Reverse_engineering_of_integrated_circuits/smart_cards)
on this kind of chip it's really difficult because the equipment to do
it is extremely expensive and hard to obtain, it is only available to
large chip manufacturers.

An access control to prevent an attacker to force the ePassport to send
information in the MRZ to readers unless the passport owner authorizes
it. Otherwise an attacker could use a reader and simply request the
biometric information to the passport. It uses a protocol called **Basic
Access Control (BAC)** to give access for the reader. It consists on a
[OCR](https://secure.wikimedia.org/wikipedia/en/wiki/Optical_character_recognition)
readable code inside the passport. With this code, the reader can say to
the passport it is an authorized reader, and then transmit data to it.
The BAC code is also used as key for the encrypted transmission of the
data, making an eavesdropping attack more difficult. It's important to
note that the BAC security entirely relies on the physical access to the
passport. The BAC is inside the ICAO standard but doesn't obligates it's
support.

![BAC happy path's sequence diagram](/img/bac.png)

The ICAO also standardizes a authentication for readers called
**Extended Access Control (EAC)**. It consists of sets of signatures of
terminals stored in the passport's chip. These signatures are changed
periodically by the countries (which signs them) on the terminals to
avoid keys stolen from equipments being used for much time. Some MRZ
sensitive data, such as iris and fingerprint, are just transmitted to
readers if it is EAC authenticated. Other informations are optional.

![EAC happy path's sequence diagram](/img/eac.png)

The **Passive Authentication (PA)** is a cryptographic mechanism  to
ensure the data in the passport is valid by the authority emitting the
passports. The PA is a kind of digital signature where the country who
emits passports ensures the veracity of the data with a asymmetric
Country Verification Certificate Authority (CVCA), a kind of
[message authentication code]
(https://secure.wikimedia.org/wikipedia/en/wiki/Message_authentication_code)
(MAC). This process is known as
[asymmetric cryptographic]
(https://secure.wikimedia.org/wikipedia/en/wiki/Asymmetric_cryptography)
hashing or [digital signing]
(https://secure.wikimedia.org/wikipedia/en/wiki/Digital_signature).
The PA MAC is created by the country when emits a passport, which must
keep a signing key safe, which is used to create the MAC code. Then it
publish the public key (all readers must know the countries public keys)
which can be used to check if the MAC is valid and it was really
generated by the organization which possess the signing key. The PA
scheme relies on mathematical functions which are easy to encrypt a MAC
with the private key, but computationally hard without them. The public
key can easily check if the MAC was generated by the ones who posses the
private key, and detect forged data. This process makes the forgery of
digital data a really hard computational problem, and ensures with
negligible chance of failure that the data is valid. All the MRZ data is
signed using such process. Using PA is mandatory by ICAO standards.

![PA happy path's sequence diagram](/img/pa.png)
  

The **Active Authentication (AA)** is a kind of
[challenge response authentication](https://secure.wikimedia.org/wikipedia/en/wiki/Challenge-response_authentication)
mechanism to prevent the cloning of passports. It relies on the
untraceable chip key. Cloning the MRZ data is a simple task to be done,
however you can't get a passport private key due to the difficulty of
doing reverse engineering to obtain it. The AA works as a challenge to
the passport, so you can verify if it really is. The reader encrypts a
random message that only him knows it's content with the passport's
public key. The message is then sent to the passport and it must decrypt
and response to the reader what was the original message. This process
works because of the duality principle of asymmetric cryptography. This
concept of challenge is the same concept behind
[CAPTCHAs](https://secure.wikimedia.org/wikipedia/en/wiki/CAPTCHA) and
the [Turing Test](https://secure.wikimedia.org/wikipedia/en/wiki/Turing_test).

![AA hapy path's sequence diagram](/img/aa.png)
