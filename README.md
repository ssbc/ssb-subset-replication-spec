# Secure partial replication for SSB

Status: Design phase

This module comes from the desire to synchronize a subset of a SSB
log. The main motivation is to enable faster initial sync by only
replicating a certain subset of messages that can then be supplimented
with a frontier sync such as [EBT].

In classical SSB, feeds up to a hops count (normally 2 or 3) is
replicated in full. The good thing about this is that any node has a
full replica of the feeds it is interested in, so the only way of
having network partitions (where a subset of the network and thus
potentially feeds is unavailable) is by having that part of the
network (as defined by hops) go offline faster than new nodes come
in. Nodes might go offline for an extended period of time or never
come back. This is somewhat mitigated by pubs.

Partial replication has a similar problem but on a different axis
where a node might be offline for a while and once it comes back all
the nodes it connects with only store a partial replica that does not
overlap on the frontier. In this case the old node would need to go
into a mode similar to initial replication. It can use its existing
subset of data of say contact messages to only request changes for
those so the sync time would be less than a full sync.

It is recommended to use this module in conjunction with
[ssb-secure-partial-feed] to automatically write statements about
where particular messages are in a feed.

This module consists of two methods:

## getSubset(query): source

The query parameters interface is similar to [JITDB]. To support
pagination, `offset`, `limit` and `reverse` can be supplied. To get
the latest 10 post messages of a particular feed the following query
can be used:

```
options: { limit 10, reverse: true},
query: {
  type: 'AND',
  data: [
    { type: 'EQUAL', data: { type: 'type', value: 'post' } },
    { type: 'EQUAL', data: { type: 'author', value: '@6CAxOI3f+LUOVrbAl0IemqiS7ATpQvr9Mdw9LC4+Uv0=.ed25519' } }
  ]
}
```

Which will result in the following:

```
{
   statements: [statement1, ...],
   data: [msg1, ...]
}
```

Where a statement is defined as the latest version of a
[ssb-observable] that describes the thing you are querying. This can
be useful for contact messages for an author where messages needs to
be in order and nothing left out by a third party. It might even be
essential for building certain applications on top of this
system. Note you can have multiple statements. This allows feeds other
than the author to add statements to a result set.

This api replaces `getFeed`, `getFeedReverse`, `getMessagesOfType` of
[partial replication v1].

Open problems:

 - how do we handle malicious actors that either sends wrong
   statements or withholds certain either messages or statements. Both
   locally but on a protocol level (blocking?). If one is introduced
   into the network by an existing user, by dowloading their feed in
   full first and then using their friend graph to either connect to a
   pub run by a friend ([ssb-friend-pub]) or directly to one of their
   friends using a room, you could minimize the harm malicious actors
   could do.

## getTangle(rootId): source

Source is a stream of messages that refers to the rootId. Tangles are
different from other messages in that messages are linked together and
thus does not need statements. `getTangle` can be used as a way to get
older messages more conveniently than getting a subset from multiple
feeds. This is useful in a social network application where someone
mentions an old thread that is not part of your frontier of
messages. `getTangle` is a special case of [set-reconciliation] where
you don't have any messages in the set. Because of this, the protocol
can be a lot simplier by just specifying the root you are interested
in. New messages in the tangle will be get replicated using the
frontier synchonization the same way as say new contact messages.

If only messages within a certain number of hops is needed, the
results can be filtered locally. In the same way this method can be
used even if you had some of the messages of a tangle by doing a local
comparison and filtering based on existing messages. This assumes that
tangles stay relativily small.

Open problems:

 - Would it be better to implement a more complicated set replication
   to allow partial syncing of tangles?

[JITDB]: https://github.com/arj03/jitdb
[ssb-observable]: https://github.com/arj03/ssb-observables
[partial replication v1]: https://github.com/arj03/ssb-partial-replication
[set-reconciliation]: https://github.com/AljoschaMeyer/set-reconciliation
[ssb-secure-partial-feed]: https://github.com/arj03/ssb-secure-partial-feed
[EBT]: https://github.com/ssbc/epidemic-broadcast-trees/
[ssb-friend-pub]: https://github.com/ssbc/ssb-friend-pub
