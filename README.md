# Secure partial replication for SSB

Status: Design phase

This module aims to provide methods for synchronizing a subset of one
or more SSB feeds. The main motivation is to enable faster initial
sync by only replicating a certain subset of messages that can then be
supplimented with a frontier sync such as [EBT].

Feeds in SSB can contain multiple subset of messages, this could be
messages of a certain type (eg. contact or chess moves) or could be
private messages pertaining a [private group]. One could argue that it
might be better to have multiple feeds for different applications, but
for private groups there is an added security benefit that it is
harder to see when you are communicating with which group if you are
part of multiple without having to add random noise.

In classical SSB, feeds up to a hops count (normally 2 or 3) is
replicated in full. This means that any node has a full replica of the
feeds it is interested in, so the only way of having network
partitions (where a subset of the network and thus potentially feeds
is unavailable) is by having that part of the network (as defined by
hops) go offline faster than new nodes come in (the churn rate). Nodes
might go offline for an extended period of time or never come
back. This is somewhat mitigated by pubs.

Partial replication has a similar problem but on a different axis
where a node might be offline for a while and once it comes back all
the nodes it connects to only store a partial replica that does not
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
options: { limit 10, reverse: true },
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

## getTangle (TBD)

Tangles in SSB behaves differently than other types of messages in
that they are linked together across multiple feeds using links. This
allows for a different kind of synchonization than `getSubset`. There
are two common examples of where this could be useful, for getting all
the messages of an old message threads that is outside the frontier of
messages and to synchronize a private group (as all messages in those
are part of a single tangle). While private groups are by definition
private to external observers, the fact that nodes connecting in SSB
are authenticated by their ID and that rooms allow direct connections
between peer allows synchronization of a private group as if the
messages were public.

As private groups can grow quite large it is important that there is a
way to synchronize only subsets but also changes. And because of this,
it could also be adventurous to reuse observables here to describe
these subsets.

Consider using [set-reconciliation] for this.


[JITDB]: https://github.com/arj03/jitdb
[ssb-observable]: https://github.com/arj03/ssb-observables
[partial replication v1]: https://github.com/arj03/ssb-partial-replication
[set-reconciliation]: https://github.com/AljoschaMeyer/set-reconciliation
[ssb-secure-partial-feed]: https://github.com/arj03/ssb-secure-partial-feed
[EBT]: https://github.com/ssbc/epidemic-broadcast-trees/
[ssb-friend-pub]: https://github.com/ssbc/ssb-friend-pub
[private group]: https://github.com/ssbc/envelope-spec
