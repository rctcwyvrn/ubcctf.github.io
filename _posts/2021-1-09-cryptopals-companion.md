---
layout: post
title: "A companion to the Monsanto Cryptopals challenges"
author: rctcwyvrn
---

Hello and welcome to my Cryptopals companion guide!

Introduction
---
Who is this guide for?
* Those new to CTF competitions and cryptography in general

Who are you and why did you write this?
* Hi I'm rctcwyvrn (call me arctic). I like crypto and I really like recommending cryptopals as a way to get newcomers into crypto. However I always felt like it needed a few more hints and explanations at times, to really make it accessible to those with zero knowledge about cryptography. For example, I went into cryptopals with next to zero experience even using cryptography, I didn't even know what AES or RSA did or what the difference was. I ended up having trouble at times and I wanted to patch up a few of those holes
* So this guide is written as a sort of guiding mentor through these challenges, with an extra focus on using these attacks in CTF competitions

What does this guide have?
* Hints and tips for the cryptopals challenges
* Some definitions that the cryptopals team chose not to include
* Asides and random tangents about things I find interesting
* Some notes about applying these attacks in CTF competitions

What does this guide not have?
* Solutions to the challenges, if you get truly stuck there are plenty of writeups and solutions just a google away
* Sensible formatting, good grammar, or accurate spelling
* A good sense of humor

What do I need to get started?
* Almost all ctfers use python for solving challenges across most of the categories, so that’s what I would recommend. You'll need:
   * A working python3 installation with [pycryptodome](https://pypi.org/project/pycryptodome/) installed
   * Something to write code with, `vscode` is nice
   * Note: I’ll be assuming you’ll be writing in python for this guide
* Some simple math/cpsc concepts
   * For sets 1-4 you'll need to know how XOR works and maybe have a basic understanding of what modulo does, that's it
   * For sets 5-6 you'll need to be more familiar with modular arithmetic and be ready to transcribe some scary looking equations. Other things to know include: what primes are and power rules from grade 7
   * If you're not confident in your math skills I swear it won't be that bad! Pinky promise!
   * The hard math is left to the professionals and we just copy their equations :)
* You don’t need (and aren’t expected to) know any cryptography beforehand
   * They do unfortunately skip out on defining some terms, but that’s what this companion is for
* PS: Give the cryptopals homepage and [this article](https://blog.pinboard.in/2013/04/the_matasano_crypto_challenges/) a read if you don't believe me

About cryptopals
* These challenges are attacks, not puzzles with tricks. The challenge authors make an effort to make the challenges clear and "doable", instead of hiding details or requiring guessing like you'd see in competitions. This companion guide also helps clear up the occasional ambiguity and confusing wording
* The general structure of the sets is
   * Set 1 is preparation and “the qualifier set” 
   * Sets 2-4 are various attacks on symmetric ciphers and a few hashes in set 4
   * Sets 5-6 are attacks on various asymmetric primitives, all using primes and numbers

Disclaimer:
I like to put an extra little disclaimer whenever I recommend someone take a look at cryptopals because after solving these challenges, some people get a strong feeling of dread. Dread when they realize how fragile cryptography can be and how most people writing code that uses cryptography are **completely unaware** that these fragile points exist
  

_"There is no difference, from the attacker's point of view, between gross and tiny errors. Both of them are equally exploitable. In at least three challenges, the mere fact of getting distinguishable error messages was enough to recover the entire message._

_This lesson is very hard to internalize. In the real world, if you build a bookshelf and forget to tighten one of the screws all the way, it does not burn down your house"_

- Marciej Ceglowski

PS: In case you’re wondering how well cryptopals will prepare you for CTFs, take a look at [CSAW CTF 2020](https://ctftime.org/event/1079/tasks/). Three challenges are basically directly taken from cryptopals and can be solved by copying pasting the solutions, which is what I did during the competition
   * Authy is a length extension sha attack, covered in challenge 29
   * modus operandi is a ciphertext detection challenge, covered in challenge 11
   * adversarial is a fixed nonce CTR, covered in challenges 6 and 20


Some quick definitions
---
**Encryption**
   * Encryption is turning a message (like “cryptography is cool”) and changing into a message that appears to be a garbled mess (like “falkjdior30vjs”) based on some “key” in a way where the only way to recover the original message is to use the key
      * This may seem like too simple of a definition, but I think it's a good mental model for what encryption does. "Scramble a message in a reversible way" is how I remember it
   * The method of encrypting the message and what the key is vastly depends on the algorithm, for example in rsa the key is a number, in caesar ciphers the key is a table, and in aes the key is 16 bytes
   * **Plaintext** is used to refer to the message before encryption and **ciphertext** is used for the garbled message after encryption

Set 1
---
Intro
   * A lot of this set comes down to how comfortable you are writing code in python
   * There is a little hitch in that byte representations in python changed a lot between python2 and python3, meaning stackoverflow answers and such will be outdated and might not work, feel free to bug me on discord if you’re having trouble

Challenge 1: Some useful functions and notes about python bytes
   * `int(x, 16)` will parse the string x as a hex string, and return a number
   * The `Crypto.Util.number.long_to_bytes` and `Crypto.Util.number.bytes_to_long` functions from pycryptodome convert between numbers and bytes
   * Bytes in python are really just an array of ints between 0 and 255
      * `b”hi”` is really just `[104, 105]`
      * the `bytes()` constructor takes an int array and returns a bytes object
      * using a for loop or a list comprehension like `[x for x in my_bytes]` will return the int array
      * you can also index into bytes objects to get the integers, `b”hi”[0] == 104`
      * Remember that bytes are just numbers with base 255
   * `ord()` takes a single character and returns its integer value
   * `chr()` does the opposite, integer to char
   * the `base64` package has functions to go from bytes <-> base64 strings, namely `b64decode` and `b64encode`

Challenge 2: XOR in python
   * The xor operator in python is `^` and it takes two integers and returns the result of xor
   * A note about XOR in case you aren’t familiar: The operation cancels out with itself if you do it twice
      * Ie: `A xor B xor B == A`
      * If you aren’t sure why this is the case, try writing out the table for XOR
      * This is why “encryption” and “decryption” are actually the same action when using XOR, you end up doing the exact same thing
   * How do we use `^` with bytes objects? Remember that we can index into bytes objects to get the underlying integers, so try a for loop and then combining the results afterwards with `bytes()` back into a bytes object

Challenge 3
   * This general idea comes up sometimes in cryptography attacks, but the idea is that we know the original must have been some english sentence, so we can bruteforce and see which one is most like an english sentence
      * This idea also can be generalized to cases where we know some "structure" that the original message must be in and use that information to inform our bruteforce
   * Note: Do not be lazy and do it by eye, the “scoring” function will be needed in the next challenge
   * The `sorted()` function will probably come in handy
   * [https://en.wikipedia.org/wiki/Letter_frequency](https://en.wikipedia.org/wiki/Letter_frequency)

Challenge 4
   * Basically the same as challenge 3 but at a larger scale, same principle applies
   * We can assume that anything that isn’t encrypted with a single character XOR will result in a garbled mess when we perform an XOR on it

Challenge 5
   * Just coding, good luck!
   * The modulo operator `%` will probably come in handy here

Challenge 6
   * Hamming distance function tips
      * To format an integer as a binary string, use `“{0:b}”.format(x)`
      * Use in combination with `bytes_to_long` to convert the bytes to the bit string
      * If the lengths differ, the difference in length should be added to the distance
   * Solve tips
      * Try to reuse as much code from earlier challenges as possible
      * Transposing means taking `[[a,b,c], [1,2,3]]` and making it into `[[a,1], [b,2], [c,3]]`
   * An aside
      * I’ve actually used my code for this challenge quite a few times. You might be wondering what real world algorithm uses a repeating key xor and the answer is none, but many ciphers do actually use XORs. This challenge may seem like a toy challenge, but when we get to challenge 20 in set 3, you’ll see how this attack actually works **very well** on a popular block cipher mode.

Some definitions
* **AES** (Advanced Encryption Standard) is a **block cipher**, it takes a block of 16 bytes and garbles it based on a 16 byte key
   * However this presents a problem, what do you do if your message is longer than 16 bytes, for example a 32 byte message?
   * If you’re curious about what AES does under the hood (not needed for these challenges, but fun to learn about) [https://www.youtube.com/watch?v=O4xNJsjtN6E](https://www.youtube.com/watch?v=O4xNJsjtN6E)
* **ECB** stands for Electronic Code Book, and is a **block cipher mode of operation**. Block cipher modes are ways of making block ciphers (which can only encrypt a single block) usable for larger messages
   * ECB is the most straightforward, cut the message into blocks and encrypt each of them separately.
   * This turns out to be a terrible way of doing things, and you’ll see why in the next few challenges
   * [https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Electronic_codebook_(ECB)](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Electronic_codebook_(ECB))
  
  
A long aside about symmetric ciphers: What makes a good cipher?
* Properties that you want from a cipher tend to fall into two main categories
   * Properties that make the cipher more practical
   * Properties that make the cipher secure
* What properties make a cipher more practical?
   * Simplicity of the algorithm, for example AES rounds are "simple" enough to be encoded as a CPU instruction. 
      * Simplicity also means less chance of implementation errors and ease of analysis. 
      * Simpler also means easier to implement on restricted systems, like smart cards or IoT devices
   * Speed. Rijndael, the cipher that won the AES competition to be named AES, won over Serpent, another contender, due to the Rijndael having a much faster and efficient implementation
* What properties make a cipher more secure?
   * This section could probably be it's own textbook, but understanding these things may help with attacking the custom ciphers you come across in CTF challenges occasionally, though these attacks are all very advanced and I've only ever implemented two of them (slide and MITM)
   * A large enough key and block size to not be bruteforcable (see [DES](https://en.wikipedia.org/wiki/Data_Encryption_Standard))
   * Since many symmetric ciphers are round based: A strong round function to be resistant to slide attacks [1](https://www.robertxiao.ca/hacking/sarah2/), [2](https://rctcwyvrn.github.io/posts/2019-12-02-bsides_crypto.html)
   * Non linearity of the cipher, a linear change in plaintext should not result in a linear change in ciphertext
      * Protects against [linear cryptoanalysis](https://en.wikipedia.org/wiki/Linear_cryptanalysis)
   * Flips in plaintext should result in a uniform chance for flips in all other bits in the ciphertext, ie you should not be able to know that flipping bit 8 in the plaintext will result in a flip in bit 6 in the ciphertext
      * Protects against [differential cryptoanalysis](https://en.wikipedia.org/wiki/Differential_cryptanalysis)
   * Unable to mount a divide-and-conquer/meet-in-the-middle style algorithm to bruteforce, ie there shouldn't be a way to split up the encryption into parts, bruteforce them seperately, and meet in the middle. See [Double DES](https://en.wikipedia.org/wiki/Meet-in-the-middle_attack#:~:text=The%20MITM%20attack%20is%20one,DES%20and%20not%20Double%20DES.&text=Triple%20DES%20uses%20a%20%22triple,the%20size%20of%20its%20keyspace.)
      * Isn't it annoying how meet-in-the-middle and man-in-the-middle have the same acronym, even though they're completely different

  
Challenge 7

   * pycryptodome provides an AES implementation in Crypto.Cipher.AES
      * [https://pycryptodome.readthedocs.io/en/latest/src/cipher/aes.html](https://pycryptodome.readthedocs.io/en/latest/src/cipher/aes.html)
   * You should make both encryption and decryption functions, you’ll need it in a bit
  
Challenge 8
   * There is a hidden assumption in this challenge that I think really should have been mentioned. **The challenge assumes that the plaintext must have a repeated block**. 
   * The important point here is that ECB is the only cipher mode that has this property, that the same plaintext blocks will result in the same ciphertext blocks
      * This weakness of ECB turns out to cause many many problems, which you’ll see in set 2
      * The worst part is ECB is seen as the “default” setting for block ciphers, despite it being almost always a terrible idea


Set 2
---
Challenge 9
   * Fairly straightforward, just some more programming


Challenge 10: A new block cipher mode, CBC
   * **CBC** or Cipher block chaining is another block cipher mode
      * The goal of CBC is to not be deterministic like ECB, ie two identical plaintext blocks should encrypt to _different_ ciphertext blocks
      * How does it accomplish this?
         * CBC requires an extra **initialization vector** or IV for short, which is just random bytes the size of a block
         * For each plaintext block we want to xor our plaintext with something first, and then encrypt it with our cipher
            * For the first block we xor it with the IV before encrypting
            * For every other block we xor it with the _last ciphertext block_
         * Essentially “chaining” the different blocks together, so the resulting ciphertext of a block depends on the “sum” of all the previous blocks and the IV
      * [https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Cipher_block_chaining_(CBC)](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Cipher_block_chaining_(CBC)) has some very useful diagrams
   * Implementing it is fairly straightforward once you understand whats going on
   * Decryption is just running this in reverse, block cipher decryption first and then xor
   * Note: ECB mode with one block is the same as just running the cipher, so you can reuse that as long as you’re encrypting/decrypting only one block
  
![CBCEncryption](/assets/images/cryptopals/cbc_encryption.png)
  
Challenge 11
   * This is a bit of a weird challenge, I’m not really sure what it’s accomplishing?
   * I don’t think there’s a good way to detect CBC mode, so I just detected ECB mode and then guessed CBC otherwise
   * Again, it only works if you send a plaintext with repeated blocks
   * Idk, maybe skip this one, it kinda comes back in challenge 12 but as an “optional” thing

Challenge 12
   * Now don’t skip this one though, because it’s really cool (and also shows up in real CTFs!)
   * The explanation is pretty good for how to get the first byte of the unknown-string, but the next step and automating it can be a bit tricky
      * For the next byte you would first send 14 A’s + the first byte of the unknown string
         * The first block would then be aes(“A”s + first byte + second byte)
      * Then you want to start sending 13 A’s + first byte + guesses for the second byte
      * And then like last time stop when you find a match
   * The congratulations at the bottom is pretty accurate for CTFs too, every once in awhile I see another one of these pop up
      * [Here’s the last one I remember](https://ctftime.org/writeup/18675)

Challenge 13: Cut and paste
   * This one is fun to figure out, it’s the first time the authors really let you out and try to figure out how to do what it wants
   * The title is a big hint, if we could copy paste things, what _would_ we want to cut and replace?
   * Given that this is ECB mode, what exactly can we cut and replace? (bits? bytes? characters? blocks? messages?)

Challenge 14
   * Some clarification on this challenge, I originally thought the random prefix would be generated each time, a new random prefix of random length each time you touch the oracle
   * I honestly don’t really know how one would go about solving that, so I and everyone else who has write ups for cryptopals instead assumed that the random prefix would be generated once and be reused for all later oracle calls
   * So now the trouble is really just one thing, how long is that random prefix? How can you figure that out?

Challenge 15
   * More programming stuff, this is just setup for challenge 17, which is gonna be a big one

Challenge 16
   * Another fun one to figure out
   * Try to think about what happens in CBC decryption with the user data block and the block after it
   * How can we completely replace the second ciphertext block in a way to make the third block decrypt to what we want, namely “;admin=true;”? What happens to the second plaintext block in that case?


Set 3
---
Intro
   * For me, this is the set where things really started to get fun
   * This note at the start made me really hyped and I hope it makes you excited too
      * “We've also reached a point in the crypto challenges where all the challenges, with one possible exception, are valuable in breaking real-world crypto.”

Challenge 17
   * This is a great challenge
      * CBC is (unfortunately) still a relatively popular block cipher mode, and is seen as "the good alternative" to ECB
      * This directly builds off of what you learned about CBC decryption in challenge 16, how bit flips in one ciphertext block affect the resulting decryption in the next block
      * CBC padding oracles also show up from time to time in CTFs, though more rarely due to how famous this attack is
   * Here is a good explanation of the attack: [https://robertheaton.com/2013/07/29/padding-oracle-attack/](https://robertheaton.com/2013/07/29/padding-oracle-attack/ )
   * From my experience this challenge is not necessarily conceptually difficult, but can be a bit tricky to get right
      * Start with just finding one byte, then work towards automating it for the rest of the bytes in the block, and then finally for multiple blocks
      * Made sure the padding functions from set 2 are fully correct
         * A plaintext that is all padding should be accepted
         * A plaintext with no padding should be rejected
   * Aside: Anyone else feeling that dread after realizing that all it takes to break a perfectly secure algorithm like AES and a reasonably sensible mode like CBC is something as small as having two different error messages for wrong password vs bad padding? Because I definitely was
   * Aside: This challenge really shows off how useful these side channel leaks are. A **side channel leak** is when a system unintentionally reveals information about it's inner workings
      * For this challenge, the "decryption server" is leaking the fact that the padding is invalid
      * For later challenges you'll see that even leaking how long the code takes to run can be enough to cryptography
         * This is called a **timing attack**
      * Some other side channels you won't see in cryptopals: 
         * The amount of power the CPU is using can be used to retrieve AES keys from smart cards
         * Using information about the CPU cache to break OpenSSL's carefully implemented RSA algorithm [https://web.eecs.umich.edu/~genkin/cachebleed/index.html](https://web.eecs.umich.edu/~genkin/cachebleed/index.html)
         * An example of a timing attack using a side channel leak from a CTF challenge: [https://ubcctf.github.io/2020/11/dragonctf2020-bitflips/](https://ubcctf.github.io/2020/11/dragonctf2020-bitflips/)