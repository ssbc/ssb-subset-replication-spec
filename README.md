# Secure partial replication for SSB

Status: Design phase

This module aims to provide methods for synchronizing a subset of one
or more SSB feeds. The main motivation is to enable faster initial
sync by only replicating a certain subset of messages that can then be
supplimented with a frontier sync such as [EBT].

Feeds in classical SSB can said to contain multiple subset of
messages, this could be messages of a certain type (eg. contact or
chess moves) or could be private messages pertaining to a [private
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

getSubset is mainly intended for the subset case of a classical feed.

It can been seen as a more general version of createHistoryStream
where the id, or what you are talking about, first needs to be
defined. For this we add a query parameter. From this query the remote
end would generate or reuse a feed that would answer the query.

The query parameters interface is similar to [JITDB]. To support
pagination, `offset`, `limit` and `reverse` can be
supplied. Furthermore there is in an option to specify whether to
include `auxiliary` data or not. This option could be used to include
blobs or could be used for feeds that mainly references existing
feeds. In this way the messages in the generated feed would only
contain a hash of what they are pointing to, and the auxiliary field
would include the original data.

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
   data: [msg1, ...],
   auxiliary: [aux1, ...]
}
```

Here metadata would refer to the metadata attached to the feed in meta
feed.

## getTangle (TBD)

Tangles in SSB behave differently than other types of messages in
that they are linked together across multiple feeds using links. This
allows for a different kind of synchonization than `getSubset`. There
are two common examples of where this could be useful: for getting all
the messages of an old message threads that is outside the frontier of
messages and to synchronize a private group (as all messages in those
are part of a single tangle). While private groups are by definition
private to external observers, the fact that nodes connecting in SSB
are authenticated by their ID and that rooms allow direct connections
between peer allows synchronization of a private group directly with
another group member who also knows which messages belong to the group.

As private groups can grow quite large it becomes important that there
is a way to synchronize only subsets but also changes or the latest
part of a tangle. The membership part of a private group has a tangle
root and thus can be replicated on its own.

Consider using [set-reconciliation] for this.

# Prior work

Earlier threads on SSB discussing partial replication:

- %L9m5nHRqpXM4Zkha1ENTk5wNOXQMduve8Hc9+F0RLZI=.sha256
- %VNCOf3pfP7hjL3lcFvBSSOzRPtR0WHDtvlXvkBcha3I=.sha256
- %gaYxXAEoSKf3dnw2OlI2EuoduFmkiU4kU+CncbBImPk=.sha256
- %jM6Dw0BEg9IPVkDH5VaoiVSbt3gCnUfSoO1BhX2Ntpw=.sha256
- %SEXkvEQCJd2jBrUXDQcFPVix/5eUffoKBURWyy5yYrU=.sha256

[JITDB]: https://github.com/arj03/jitdb
[ssb-meta-feed]: https://github.com/arj03/ssb-meta-feed
[set-reconciliation]: https://github.com/AljoschaMeyer/set-reconciliation
[EBT]: https://github.com/ssbc/epidemic-broadcast-trees/
[private group]: https://github.com/ssbc/private-group-spec
