### bitcoin scripting

... intro? 

https://en.bitcoin.it/wiki/Script

http://github.com/bitcoin/bitcoin/blob/master/src/script/interpreter.cpp

 - stack + opcodes/effects

... plus some examples

To join in with the fun, download and install kallewoof's btcdeb:

https://github.com/kallewoof/btcdeb

---

## Addition

    OP_1 OP_ADD OP_2 OP_EQUAL

ScriptSig: ??

---

## Transaction puzzle (simple hashlock)

Transaction a4bfa8ab6435ae5f25dae9d89e4eb67dfa94283ca751f393c1ddc5a837bbc31b

    OP_HASH256 6fe28c0ab6f1b372c1a6a246ae63f74f931e8365e15a089c68d6190000000000 OP_EQUAL
    
scriptSig: ???????

Anyone recognize the hash / know this preimage?

## Transaction puzzle (hashlock without a specific hash committed) 

    OP_2DUP OP_EQUAL OP_NOT OP_VERIFY OP_SHA1 OP_SWAP OP_SHA1 OP_EQUAL

scriptSig: ????

This is a really nice usage of script. 

 - no one single answer
 - thinking outside the box in terms of typical usage

## HTLC...

    OP_IF
        [HASHOP] <digest> OP_EQUALVERIFY OP_DUP OP_HASH160 <seller pubkey hash>            
    OP_ELSE
        <num> [TIMEOUTOP] OP_DROP OP_DUP OP_HASH160 <buyer pubkey hash>
    OP_ENDIF
    OP_EQUALVERIFY
    OP_CHECKSIG

... weird features

spk: OP_IF OP_ELSE OP_ELSE OP_ELSE OP_ELSE OP_ENDIF
 ss:

... some checksig examples?

