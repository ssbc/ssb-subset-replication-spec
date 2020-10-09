# Secure partial replication for SSB

Status: Design phase

Query interface like [JITDB]:

```
{
  type: 'AND',
  data: [
    { type: 'EQUAL', data: { type: 'type', value: 'post' } },
    { type: 'EQUAL', data: { type: 'author', value: '@6CAxOI3f+LUOVrbAl0IemqiS7ATpQvr9Mdw9LC4+Uv0=.ed25519' } }
  ]
}
```

and some extra syntax for stuff like reverse, offset, limit. Hops for proofs?

Result:
```
{
   proofs: [proof],
   data: [msgs]
}
```

Where a proof is defined as the latest version of a [ssb-observable]
that describes the thing you are querying. So e.g. contact messages
for an author. Note you can have multiple proofs in cases where not
all proofs are current or not from the author.

This api replaces getFeed, getFeedReverse, getMessagesOfType of
[partial replication v1]. This leaves getTangle.

This should be used in conjunction with [ssb-secure-partial-feed].

Open questions:
 - EBT like sync based on queries, so like observable but more general
 - How do you define a succint way to get new messages based on your
   current state of a tangle. Tangles can have branches and 
   potentially changes quite a way back in time. Maybe one needs something 
   like a merkle tree for this? Maybe something like [set-reconciliation]

[JITDB]: https://github.com/arj03/jitdb
[ssb-observable]: https://github.com/arj03/ssb-observables
[partial replication v1]: https://github.com/arj03/ssb-partial-replication
[set-reconciliation]: https://github.com/AljoschaMeyer/set-reconciliation
[ssb-secure-partial-feed]: https://github.com/arj03/ssb-secure-partial-feed
