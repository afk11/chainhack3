### bitcoin scripting

... intro? 

https://en.bitcoin.it/wiki/Script

http://github.com/bitcoin/bitcoin/blob/master/src/script/interpreter.cpp

stack carrying data + opcodes w/ effects

To join in with the fun, download and install kallewoof's btcdeb:

https://github.com/kallewoof/btcdeb

---

## Addition

This output / unlocking script simply checks that an addition was successful

    0x01 OP_ADD 0x02 OP_EQUAL

 - predicate. something + 1 == 2

ScriptSig: ??

---

    ./btcdeb "[
        0x01 OP_ADD 0x02 OP_EQUAL
    ]" 01

Note: when you enter the btcdeb shell just keep pressing step!

---

## Transaction puzzle (simple hashlock)

Transaction a4bfa8ab6435ae5f25dae9d89e4eb67dfa94283ca751f393c1ddc5a837bbc31b

    OP_HASH256 6fe28c0ab6f1b372c1a6a246ae63f74f931e8365e15a089c68d6190000000000 OP_EQUAL
    
scriptSig: ???????

Anyone recognize the hash / know this preimage? (hint: hash256 == sha256d)

---

It's the genesis block header!

    ./btcdeb "[
        OP_HASH256 6fe28c0ab6f1b372c1a6a246ae63f74f931e8365e15a089c68d6190000000000 OP_EQUAL
    ]" 0100000000000000000000000000000000000000000000000000000000000000000000003ba3edfd7a7b12b27ac72c3e67768f617fc81bc3888a51323a9fb8aa4b1e5e4a29ab5f49ffff001d1dac2b7c

---

## Transaction puzzle (hashlock without a specific hash committed) 

    OP_2DUP OP_EQUAL OP_NOT OP_VERIFY OP_SHA1 OP_SWAP OP_SHA1 OP_EQUAL

scriptSig: ????

This is a really nice usage of script. 

 - no one single answer
 - thinking outside the box in terms of typical usage

---

This script allow someone with knowledge of a SHA1 collision obtain a bounty.

https://bitcointalk.org/index.php?topic=293382.20

./btcdeb "[
OP_2DUP OP_EQUAL OP_NOT OP_VERIFY OP_SHA1 OP_SWAP OP_SHA1 OP_EQUAL
]" 255044462d312e330a25e2e3cfd30a0a0a312030206f626a0a3c3c2f57696474682032203020522f4865696768742033203020522f547970652034203020522f537562747970652035203020522f46696c7465722036203020522f436f6c6f7253706163652037203020522f4c656e6774682038203020522f42697473506572436f6d706f6e656e7420383e3e0a73747265616d0affd8fffe00245348412d3120697320646561642121212121852fec092339759c39b1a1c63c4c97e1fffe017f46dc93a6b67e013b029aaa1db2560b45ca67d688c7f84b8c4c791fe02b3df614f86db1690901c56b45c1530afedfb76038e972722fe7ad728f0e4904e046c230570fe9d41398abe12ef5bc942be33542a4802d98b5d70f2a332ec37fac3514e74ddc0f2cc1a874cd0c78305a21566461309789606bd0bf3f98cda8044629a1 255044462d312e330a25e2e3cfd30a0a0a312030206f626a0a3c3c2f57696474682032203020522f4865696768742033203020522f547970652034203020522f537562747970652035203020522f46696c7465722036203020522f436f6c6f7253706163652037203020522f4c656e6774682038203020522f42697473506572436f6d706f6e656e7420383e3e0a73747265616d0affd8fffe00245348412d3120697320646561642121212121852fec092339759c39b1a1c63c4c97e1fffe017346dc9166b67e118f029ab621b2560ff9ca67cca8c7f85ba84c79030c2b3de218f86db3a90901d5df45c14f26fedfb3dc38e96ac22fe7bd728f0e45bce046d23c570feb141398bb552ef5a0a82be331fea48037b8b5d71f0e332edf93ac3500eb4ddc0decc1a864790c782c76215660dd309791d06bd0af3f98cda4bc4629b1

---

## HTLC...

IF opcodes make some blocks conditional. 

    OP_IF
        [HASHOP] <digest> OP_EQUALVERIFY OP_DUP OP_HASH160 <seller pubkey hash>            
    OP_ELSE
        <num> [TIMEOUTOP] OP_DROP OP_DUP OP_HASH160 <buyer pubkey hash>
    OP_ENDIF
    OP_EQUALVERIFY
    OP_CHECKSIG

(Note: I didn't include an example signature here, exercise for the reader!)

---

See the 01 consumed by IF causes the first half to be active
eg:
./btcdeb "[OP_IF
    OP_HASH160 0x36e11d89e9138321fe57037752f0f5b33543735c OP_EQUALVERIFY 
    OP_DUP OP_HASH160 0xe1bcdc6e4ea443d08d7c912d54f0bcc760638e78
OP_ELSE
    0xc09aed5a OP_CHECKLOCKTIMEVERIFY OP_DROP 
    OP_DUP OP_HASH160 0x935ba41cd019cf2984bd2b8a15be24c6b82172f4
OP_ENDIF
OP_EQUALVERIFY OP_CHECKSIG
]" 0x0258d5242f4fc5a2a2a15014eb75f03139413ab0abd4e2561a8d54e71b0ca06a8d 04040404 01

---

See the 00 consumed by IF causes the _second_ half to be active.

./btcdeb "[ OP_IF
   OP_HASH160 0x1436e11d89e9138321fe57037752f0f5b33543735c OP_EQUALVERIFY 
   OP_DUP OP_HASH160 0x14e1bcdc6e4ea443d08d7c912d54f0bcc760638e78
OP_ELSE
   0x04919bed5a OP_CHECKLOCKTIMEVERIFY OP_DROP 
   OP_DUP OP_HASH160 0x14935ba41cd019cf2984bd2b8a15be24c6b82172f4
OP_ENDIF
OP_EQUALVERIFY OP_CHECKSIG
]" 031eb5c69ab71574848671de6b1b4094ab9a1e1687d2e5b0aa76208df59f0bd3b5 00

## Weird features, for a programming langauge

Kind of random, but you can repeat ELSE in bitcoin script!

OP_IF OP_1 OP_1 OP_ELSE OP_1 OP_1 OP_1 OP_ELSE OP_DROP OP_ELSE OP_DROP OP_DROP OP_ENDIF

---

01 causes the first two 1's to be PUSHED.
No more 1's pushed.
One 01 dropped (leaving 1)
No more 1's dropped.

./btcdeb "[
OP_IF 
    OP_1 OP_1 
OP_ELSE 
    OP_1 OP_1 OP_1 
OP_ELSE 
    OP_DROP 
OP_ELSE 
    OP_DROP OP_DROP 
OP_ENDIF
]" 01

---

00 causes the first part to be skipped.
The next section pushes 3 IF's
The next section is inactive (none dropped)
Last section, two dropped

./btcdeb "[
OP_IF 
    OP_1 OP_1 
OP_ELSE 
    OP_1 OP_1 OP_1 
OP_ELSE 
    OP_DROP 
OP_ELSE 
    OP_DROP OP_DROP 
OP_ENDIF
]" 00 
