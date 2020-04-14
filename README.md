# Decentralized Privacy Preserving Proximity Tracing

* Source code license: Apache 2.0

The full set of documents for DP3T is at <https://github.com/DP-3T/documents>.
Please refer to the technical documents and whitepapers for the descriptions
of the implementation.

At its core, DP3T describes a decentralized framework for proximity tracing
where the smart phones store all observed contacts in a local database.
During periodic intervals, the apps poll a database of infected SKt vectors
and check if they have been exposed to any of the beacons sent by an infected
app.

One of the core challenges in designing and implementing such a framework is
the large amount of unknowns. This mock implementation of the framework helps
us identify and measure some of these unknowns so that we can carry out better
decisions and fine tune our technical design. This mock implementation is not
meant to be a real implementation.

The mock implementation focuses on the app side. The backend is extremely
simple and may consist of just a set of flat files of `<SK, day>` tuples. The
app periodically polls and downloads lists of newly infected, checking locally
if they have been exposed.


## Cryptographic primitives: `KeyStore`

The cryptographic primitives are implemented in the class `DP3T.KeyStore`.
The main methods are `rotate_SK` which rotates the private SK to the next day
and `rotate_ephIDs` which creates the set of `ephIDs` for the new day. These
methods are straight forward implementations of the description in the
whitepaper.

LowCostDP3T.py, when run standalone, will generate test vectors to help
other implementors check their work:

```
$ python3 ./LowCostDP3T.py 
sk0 0000000000000000000000000000000000000000000000000000000000000000
ephID[0] 8fd521e6c47060efcbfdb9b801c30743
ephID[1] d86e56bb702117b8cf20dc4aadd42310
ephID[95] f38403173134f2c65682ee799e817ef3
sk1 66687aadf862bd776c8fc18b8e9f8e20089714856ee233b3902a591d0d5f2925
ephID[0] fda0fff97c5566886c5afcb0c7333f73
ephID[1] 1d1ae795488b0ad0e4aba33d4bed3f4a
ephID[95] b832b76cb203226cd7297f8483c60b76
```

## Contact tracing: `ContactManager`

The contact tracing and management is implemented in the class
`DP3T.ContactManager`. Newly received beacons come in through `receive_scans`
where they are added to local observations. Each epoch, these local
observations are evaluated in `process_epoch` and contacts that have been
around long enough (there was sufficient contact) are added to the daily
observed contacts. The `check_infected` method takes an `SK` and date and
checks all local contacts if one of the `EphIDs` of that `SK` were observed
and then issues a warning.


## Installation

You can run the DP3T reference implementation with your favorite `python3`
interpreter. The only additional package you need is `Cryptodome`. You can
install it, e.g., through `sudo apt install python3-pycryptodome` or
`pip3 install pycryptodomex` (or `pip` if you only have python3 installed).
Note that for `pip`, there is a difference between `pycryptodome` and
`pycryptodomex` as they install different subsets of the library API
functionality.


## Example run

The file `example_run.py` show cases the different aspects of proximity
tracing. The three persons Alice, Bob, and Isidor interact in different
settings and record the corresponding ephemeral IDs. At one point, Isidor
is tested positive and agrees to submit his `SK_t`. Next, Alice and Bob test
for this `SK_t` and Bob detects that he was at risk 1 day ago.

You can run the example with `python3 example_run.py`.
