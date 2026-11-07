# Varhash

Varhash is a compact binary format for encoding hash digests. Its design evolved from [Multihash](https://multiformats.io), trading extensibility for smaller size and simplicity.

With a 1-byte overhead, it supports:
* Up to 64 hash functions
* Digest lengths of 16, 32 or 64 bytes

With a 2-byte overhead, it supports digest lengths from 1 to 256 bytes.

## Specification

### Bytes

1. **Header** – 1 byte  
    * **Function** – bits 7–2  
    An unsigned index from 0–63 into [the registry](#registry).
    * **Length** – bits 1–0  
      A flag indicating the digest length:
      
      | Flag | Length                              |
      |------|-------------------------------------|
      | `00` | Custom (specified in the next byte) |
      | `01` | 16 bytes                            |
      | `10` | 32 bytes                            |
      | `11` | 64 bytes                            |

2. **Custom length** – 1 byte (present only when the length flag is `00`)  
   An unsigned byte encoding the digest length minus one. Values 0–255 correspond to digest lengths of 1–256 bytes.

   Custom lengths must be ≤ the algorithm's default digest length. Any truncation of the digest must consist of the leading bytes.

   Digest lengths of 16, 32, and 64 bytes must use their dedicated length flags. Specifying these lengths with the custom length byte is invalid.

3. **Digest**  
   The binary digest of the hash function.

That's it.

### Layout

Bit 7 is the most significant bit of each byte.

|          | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|----------|---|---|---|---|---|---|---|---|
| Header   | F | F | F | F | F | F | L | L |
| Custom   | C | C | C | C | C | C | C | C |
| Digest   | D | D | D | D | D | D | D | D |
| Digest   | … | … | … | … | … | … | … | … |

- **F:** Function index
- **L:** Length flag
- **C:** Custom length (present only when the length flag is `00`)
- **D:** Digest

## Comparison

Varhash is not intended to replace Multihash, but a comparison can be useful.

* **Simpler**  
  Multihash uses [LEB128](https://en.wikipedia.org/wiki/LEB128) to achieve unlimited extensibility, adding parsing complexity and overhead. Varhash doesn't, making it simpler to implement, faster to process and smaller in size, while still supporting variable-length digests.

* **Smaller**  
  For common digest sizes (16, 32, or 64 bytes), Varhash incurs an overhead of only 1 byte. Depending on the hash function, Multihash requires 2–5 bytes to express the same information. It may not sound like much, but it adds up if you have a lot of hashes. 1 billion bytes = 1 GB.

* **Relatively future-proof**  
  Protocols rarely change their hash functions. NIST has approved 12 hash functions since 1993. By maintaining a curated registry of well-established hash functions, 64 slots should last a long time.  

* **Reasonably variable-length**  
  The custom length allows arbitrary digest sizes within reason, 1–256 bytes, including truncated hashes.

## Registry

Based on IANA's [Named Information Hash Algorithm Registry](https://www.iana.org/assignments/named-information#hash-alg).

```
 0: reserved
 1: sha-256
 2: sha-384
 3: sha-512
 4: sha3-224
 5: sha3-256
 6: sha3-384
 7: sha3-512
 8: blake2s-256
 9: blake2b-256
10: blake2b-512
```

Entries are immutable, but may be marked as deprecated.

---

_Made by a human being._

This specification is dedicated to the public domain ([CC0 1.0 Universal](./LICENSE)).
