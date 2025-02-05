# AWS-LC ML-DSA readme file

The source code in this folder implements ML-DSA as defined in FIPS 204 Module-Lattice-Based Digital Signature Standard [link](https://csrc.nist.gov/pubs/fips/204/final).

**Source code origin and modifications** 

The source code was imported from a branch of the official repository of the Crystals-Dilithium team: https://github.com/pq-crystals/dilithium. The code was taken at [commit](https://github.com/pq-crystals/dilithium/commit/444cdcc84eb36b66fe27b3a2529ee48f6d8150c2) as of 10/29/2024. At the moment, only the reference C implementation is imported.

The code was refactored in [this PR](https://github.com/aws/aws-lc/pull/1910) by parameterizing all functions that depend on values that are specific to a parameter set, i.e., that directly or indirectly depend on the value of `DILITHIUM_MODE`. To do this, in `params.h` we defined a structure that holds those ML-DSA parameters and functions
that initialize a given structure with values corresponding to a parameter set. This structure is then passed to every function that requires it as a function argument. In addition, the following changes were made to the source code in `pqcrystals_dilithium_ref_common` directory:

- `randombytes.{h|c}` are deleted because we are using the randomness generation functions provided by AWS-LC.
- `sign.c`: calls to `randombytes` function is replaced with calls to `RAND_bytes` and the appropriate header file is included (`openssl/rand.h`).
- `ntt.c`, `poly.c`, `reduce.c`, `reduce.h`: have been modified with a code refactor. The function `fqmul` has been added to bring mode code consistency with Kyber/ML-KEM. See https://github.com/aws/aws-lc/pull/1748 for more details on this change.
- `reduce.c`: a small fix to documentation has been made on the bounds of `reduce32`.
- `poly.c`: a small fix to documentation has been made on the bounds of `poly_reduce`.
- `polyvec.c`: a small fix to documentation has been made on the bounds of `polyveck_reduce`.

**Testing** 

The KATs were obtained from https://github.com/post-quantum-cryptography/KAT. We select the KATs for the signing mode `hedged`, which derives the signing private random seed (rho) pseudorandomly from the signer's private key, the message to be signed, and a 256-bit string `rnd` which is generated at random. The `pure` variant of these KATs were used, as they provide test vector inputs for "pure" i.e., non-pre-hashed messages. The KAT files have been modified to insert linebreaks between each test vector set.