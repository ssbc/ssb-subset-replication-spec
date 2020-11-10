# Secure partial replication for SSB

Status: Design phase

This module aims to provide methods for synchronizing a subset of one
or more SSB feeds. The main motivation is to enable faster initial
sync by only replicating a certain subset of messages that can then be
supplimented with a frontier sync such as [EBT].

Feeds in classical SSB can said to contain multiple subset of
messages, this could be messages of a certain type (eg. contact or
chess moves) or could be private messages pertaining a [private
group].

In classical SSB, feeds up to a hops count (normally 2 or 3) are
replicated in full. There also exists a method to get a particular
message out of order, the security of this hinges on the fact that you
get the ooo message by the hash and so is a special case of engtangled
messages.

Here we are interested in adding APIs that, given [ssb-meta-feed],
allows two peers to exchange feeds and subset of feeds to allow for
partial replication.

We are mainly interested in two classes of messages. A subset of
messages of a classical SSB feed and a tangle of messages given a
common root message.

## getSubset(query, options): source

getSubset is mainly intended for the subset of classical feed case.

It can been seen as a more general version of createHistoryStream
where the id or what you are talking about first needs to be
defined. For this we add a query parameter. From this query the remote
end would either try to find a feed that can answer the query directly
or generate a feed that would answer the query.

The query parameters interface is similar to [JITDB]. To support
pagination, `offset`, `limit` and `reverse` can be
supplied. Furthermore it is possible to specifify hops to limit what
feeds we would accept generated feeds from.

To get the latest 10 post messages of a particular feed the following
query can be used:

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
   metadata: [md1, ...],
   data: [msg1, ...]
}
```

Here metadata would refer to the metadata attached to the feed in meta
feed.

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

As private groups can grow quite large it becomes important that there
is a way to synchronize only subsets but also changes or the latest
part of a tangle. The membership part of a private group has a tangle
root so can be replicated on its own.

Consider using [set-reconciliation] for this.

# Prior work

Earlier thread(%L9m5nHRqpXM4Zkha1ENTk5wNOXQMduve8Hc9+F0RLZI=.sha256)
on SSB discussing partial replication.

[JITDB]: https://github.com/arj03/jitdb
[ssb-meta-feed]: https://github.com/arj03/ssb-meta-feed
[set-reconciliation]: https://github.com/AljoschaMeyer/set-reconciliation
[EBT]: https://github.com/ssbc/epidemic-broadcast-trees/
[private group]: https://github.com/ssbc/private-group-spec
