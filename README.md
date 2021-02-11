# brb_node_tree

[MaidSafe website](http://maidsafe.net) | [Safe Network Forum](https://safenetforum.org/)
:-------------------------------------: | :---------------------------------------------:

## About

This crate implements a P2P node (CLI) for using BRB over Quic protocol via [qp2p](<https://github.com/maidsafe/qp2p>).

In particular, this crate demonstrates use of a tree (brb_dt_tree --> crdt_tree) as the data type.
See also [brb_node_qp2p](https://github.com/maidsafe/brb_node_qp2p), which uses an orswot.

Please see the [brb crate](https://github.com/maidsafe/brb/).


# Example Session.

We will demonstrate an interactive session with two nodes A and B.

## Starting a Voting Group

`Peer A` starts and trusts itself, which makes it a voting member of it's own group.

(`trust` aka force_join is a BRB Membership local operation)

```
$ cargo run
[P2P] listening on 127.0.0.1:48479

[P2P] router cmd SayHello(127.0.0.1:48479)

[P2P] Sending message to 127.0.0.1:48479 --> Peer(i:51c6d3, 127.0.0.1:48479)

[P2P] received from 127.0.0.1:48479 --> Peer(i:51c6d3, 127.0.0.1:48479)

[P2P] router cmd AddPeer(i:51c6d3, 127.0.0.1:48479)

> trust me [P2P] router cmd Trust("me")

[P2P] Trusting actor: i:51c6d3

[BRB] i:51c6d3 is forcing i:51c6d3 to join

> peers [P2P] router cmd ListPeers

i:51c6d3@127.0.0.1:48479 (voting) (self)
```

## Joining an existing Group

`Peer B` starts, adds `A` as peer, and trusts `A`,

(`trust` aka force_join is a BRB Membership local operation)

```
$ cargo run
[P2P] listening on 127.0.0.1:46915

[P2P] router cmd SayHello(127.0.0.1:46915)

[P2P] Sending message to 127.0.0.1:46915 --> Peer(i:35240f, 127.0.0.1:46915)

[P2P] received from 127.0.0.1:46915 --> Peer(i:35240f, 127.0.0.1:46915)

[P2P] router cmd AddPeer(i:35240f, 127.0.0.1:46915)

> peer 127.0.0.1:48479 [REPL] parsed addr 127.0.0.1:48479

[P2P] router cmd SayHello(127.0.0.1:48479)

[P2P] Sending message to 127.0.0.1:48479 --> Peer(i:35240f, 127.0.0.1:46915)

[P2P] received from 127.0.0.1:48479 --> Peer(i:51c6d3, 127.0.0.1:48479)

[P2P] router cmd AddPeer(i:51c6d3, 127.0.0.1:48479)

[P2P] Sending message to 127.0.0.1:48479 --> Peer(i:35240f, 127.0.0.1:46915)


> trust i:51c6d3 [P2P] router cmd Trust("i:51c6d3")

[P2P] Trusting actor: i:51c6d3

[BRB] i:35240f is forcing i:51c6d3 to join


> peers
[P2P] router cmd ListPeers

i:35240f@127.0.0.1:46915        (self)
i:51c6d3@127.0.0.1:48479        (voting)
```

`Peer A` sponsors/proposes `B` to join voting group and members {`A`} vote/agree to include `B`.

(`join` is a BRB Membership p2p voting operation)

note: `B` may have asked `A` to sponsor out of band. ie, not part of membership protocol.

```
> join i:3497d6
[P2P] router cmd RequestJoin("i:35240f")

[P2P] Starting join for actor: i:35240f

[P2P] delivering packet to i:51c6d3 at addr 127.0.0.1:48479: Packet { source: i:51c6d3, dest: i:51c6d3, payload: Membership(P(Ji:35240f)@i:51c6d3G1), sig: sig:d240c1 }

[P2P] Sending message to 127.0.0.1:48479 --> Packet(Packet { source: i:51c6d3, dest: i:51c6d3, payload: Membership(P(Ji:35240f)@i:51c6d3G1), sig: sig:d240c1 })

[P2P] received from 127.0.0.1:48479 --> Packet(Packet { source: i:51c6d3, dest: i:51c6d3, payload: Membership(P(Ji:35240f)@i:51c6d3G1), sig: sig:d240c1 })

[P2P] router cmd Apply(Packet { source: i:51c6d3, dest: i:51c6d3, payload: Membership(P(Ji:35240f)@i:51c6d3G1), sig: sig:d240c1 })

[BRB] handling packet from i:51c6d3->i:51c6d3

[MBR] Detected super majority

[MBR] broadcasting super majority

[P2P] delivering packet to i:51c6d3 at addr 127.0.0.1:48479: Packet { source: i:51c6d3, dest: i:51c6d3, payload: Membership(SM{P(Ji:35240f)@i:51c6d3G1}@i:51c6d3G1), sig: sig:452d2a }

[P2P] Sending message to 127.0.0.1:48479 --> Packet(Packet { source: i:51c6d3, dest: i:51c6d3, payload: Membership(SM{P(Ji:35240f)@i:51c6d3G1}@i:51c6d3G1), sig: sig:452d2a })

[P2P] received from 127.0.0.1:48479 --> Packet(Packet { source: i:51c6d3, dest: i:51c6d3, payload: Membership(SM{P(Ji:35240f)@i:51c6d3G1}@i:51c6d3G1), sig: sig:452d2a })

[P2P] router cmd Apply(Packet { source: i:51c6d3, dest: i:51c6d3, payload: Membership(SM{P(Ji:35240f)@i:51c6d3G1}@i:51c6d3G1), sig: sig:452d2a })

[BRB] handling packet from i:51c6d3->i:51c6d3

[MBR] Detected super majority over super majorities


> peers
[P2P] router cmd ListPeers

i:35240f@127.0.0.1:46915        (voting)
i:51c6d3@127.0.0.1:48479        (voting)        (self)
```

`Peer B` is not yet aware of the vote, so initiates an `enti_entropy` round with `A` to obtain voting history. After which, `B` knows self to be a voting member of the group.

(`anti_entropy` is a read-only BRB operation)

```
> peers
[P2P] router cmd ListPeers

i:35240f@127.0.0.1:46915        (self)
i:51c6d3@127.0.0.1:48479        (voting)

> anti_entropy i:51c6d3 [P2P] router cmd AntiEntropy("i:51c6d3")

[P2P] Starting anti-entropy with actor: i:51c6d3

[P2P] delivering packet to i:51c6d3 at addr 127.0.0.1:48479: Packet { source: i:35240f, dest: i:51c6d3, payload: AntiEntropy { generation: 0, delivered: VClock { dots: {} } }, sig: sig:477f2d }

[P2P] Sending message to 127.0.0.1:48479 --> Packet(Packet { source: i:35240f, dest: i:51c6d3, payload: AntiEntropy { generation: 0, delivered: VClock { dots: {} } }, sig: sig:477f2d })

[P2P] received from 127.0.0.1:48479 --> Packet(Packet { source: i:51c6d3, dest: i:35240f, payload: Membership(SM{SM{P(Ji:35240f)@i:51c6d3G1}@i:51c6d3G1}@i:51c6d3G1), sig: sig:0fe95d })

[P2P] router cmd Apply(Packet { source: i:51c6d3, dest: i:35240f, payload: Membership(SM{SM{P(Ji:35240f)@i:51c6d3G1}@i:51c6d3G1}@i:51c6d3G1), sig: sig:0fe95d })

[BRB] handling packet from i:51c6d3->i:35240f

[MBR] Detected super majority over super majorities

[MBR] Adding vote to history

> peers
[P2P] router cmd ListPeers

i:35240f@127.0.0.1:46915 (voting) (self)
i:51c6d3@127.0.0.1:48479 (voting)
```

## Data Operation

`Peer B` initiates a CRDT (Tree) move operation to add element '3' with parent '1' and metadata '100' to the data set. (`mv` is a DataType operation, wrapped by BRB)

```
> read
{}

> mv 1 100 3

[BRB] i:35240f initiating bft for msg Msg { gen: 1, op: OpMove { timestamp: Clock { actor_id: i:35240f, counter: 1 }, parent_id: 1, metadata: 100, child_id: 3 }, dot: i:35240f.1 }

[BRB] broadcasting i:35240f->{i:35240f, i:51c6d3}

[P2P] router cmd Deliver(Packet { source: i:35240f, dest: i:35240f, payload: BRB(RequestValidation { msg: Msg { gen: 1, op: OpMove { timestamp: Clock { actor_id: i:35240f, counter: 1 }, parent_id: 1, metadata: 100, child_id: 3 }, dot: i:35240f.1 } }), sig: sig:e34aea })

[P2P] delivering packet to i:35240f at addr 127.0.0.1:46915: Packet { source: i:35240f, dest: i:35240f, payload: BRB(RequestValidation { msg: Msg { gen: 1, op: OpMove { timestamp: Clock { actor_id: i:35240f, counter: 1 }, parent_id: 1, metadata: 100, child_id: 3 }, dot: i:35240f.1 } }), sig: sig:e34aea }

[P2P] Sending message to 127.0.0.1:46915 --> Packet(Packet { source: i:35240f, dest: i:35240f, payload: BRB(RequestValidation { msg: Msg { gen: 1, op: OpMove { timestamp: Clock { actor_id: i:35240f, counter: 1 }, parent_id: 1, metadata: 100, child_id: 3 }, dot: i:35240f.1 } }), sig: sig:e34aea })

[P2P] received from 127.0.0.1:46915 --> Packet(Packet { source: i:35240f, dest: i:35240f, payload: BRB(RequestValidation { msg: Msg { gen: 1, op: OpMove { timestamp: Clock { actor_id: i:35240f, counter: 1 }, parent_id: 1, metadata: 100, child_id: 3 }, dot: i:35240f.1 } }), sig: sig:e34aea })

[P2P] router cmd Deliver(Packet { source: i:35240f, dest: i:51c6d3, payload: BRB(RequestValidation { msg: Msg { gen: 1, op: OpMove { timestamp: Clock { actor_id: i:35240f, counter: 1 }, parent_id: 1, metadata: 100, child_id: 3 }, dot: i:35240f.1 } }), sig: sig:e34aea })

[P2P] delivering packet to i:51c6d3 at addr 127.0.0.1:48479: Packet { source: i:35240f, dest: i:51c6d3, payload: BRB(RequestValidation { msg: Msg { gen: 1, op: OpMove { timestamp: Clock { actor_id: i:35240f, counter: 1 }, parent_id: 1, metadata: 100, child_id: 3 }, dot: i:35240f.1 } }), sig: sig:e34aea }

[P2P] Sending message to 127.0.0.1:48479 --> Packet(Packet { source: i:35240f, dest: i:51c6d3, payload: BRB(RequestValidation { msg: Msg { gen: 1, op: OpMove { timestamp: Clock { actor_id: i:35240f, counter: 1 }, parent_id: 1, metadata: 100, child_id: 3 }, dot: i:35240f.1 } }), sig: sig:e34aea })

[P2P] router cmd Apply(Packet { source: i:35240f, dest: i:35240f, payload: BRB(RequestValidation { msg: Msg { gen: 1, op: OpMove { timestamp: Clock { actor_id: i:35240f, counter: 1 }, parent_id: 1, metadata: 100, child_id: 3 }, dot: i:35240f.1 } }), sig: sig:e34aea })

[BRB] handling packet from i:35240f->i:35240f

[BRB] request for validation

[P2P] delivering packet to i:35240f at addr 127.0.0.1:46915: Packet { source: i:35240f, dest: i:35240f, payload: BRB(SignedValidated { msg: Msg { gen: 1, op: OpMove { timestamp: Clock { actor_id: i:35240f, counter: 1 }, parent_id: 1, metadata: 100, child_id: 3 }, dot: i:35240f.1 }, sig: sig:d924dc }), sig: sig:4f24d6 }

[P2P] Sending message to 127.0.0.1:46915 --> Packet(Packet { source: i:35240f, dest: i:35240f, payload: BRB(SignedValidated { msg: Msg { gen: 1, op: OpMove { timestamp: Clock { actor_id: i:35240f, counter: 1 }, parent_id: 1, metadata: 100, child_id: 3 }, dot: i:35240f.1 }, sig: sig:d924dc }), sig: sig:4f24d6 })

[P2P] received from 127.0.0.1:46915 --> Packet(Packet { source: i:35240f, dest: i:35240f, payload: BRB(SignedValidated { msg: Msg { gen: 1, op: OpMove { timestamp: Clock { actor_id: i:35240f, counter: 1 }, parent_id: 1, metadata: 100, child_id: 3 }, dot: i:35240f.1 }, sig: sig:d924dc }), sig: sig:4f24d6 })

[P2P] received from 127.0.0.1:48479 --> Packet(Packet { source: i:51c6d3, dest: i:35240f, payload: BRB(SignedValidated { msg: Msg { gen: 1, op: OpMove { timestamp: Clock { actor_id: i:35240f, counter: 1 }, parent_id: 1, metadata: 100, child_id: 3 }, dot: i:35240f.1 }, sig: sig:03f99b }), sig: sig:4e3ef7 })

[P2P] router cmd Apply(Packet { source: i:35240f, dest: i:35240f, payload: BRB(SignedValidated { msg: Msg { gen: 1, op: OpMove { timestamp: Clock { actor_id: i:35240f, counter: 1 }, parent_id: 1, metadata: 100, child_id: 3 }, dot: i:35240f.1 }, sig: sig:d924dc }), sig: sig:4f24d6 })

[BRB] handling packet from i:35240f->i:35240f

[BRB] signed validated

[P2P] router cmd Apply(Packet { source: i:51c6d3, dest: i:35240f, payload: BRB(SignedValidated { msg: Msg { gen: 1, op: OpMove { timestamp: Clock { actor_id: i:35240f, counter: 1 }, parent_id: 1, metadata: 100, child_id: 3 }, dot: i:35240f.1 }, sig: sig:03f99b }), sig: sig:4e3ef7 })

[BRB] handling packet from i:51c6d3->i:35240f

[BRB] signed validated

[BRB] we have supermajority over msg, sending proof to network

[BRB] broadcasting i:35240f->{i:35240f, i:51c6d3}

[P2P] delivering packet to i:35240f at addr 127.0.0.1:46915: Packet { source: i:35240f, dest: i:35240f, payload: BRB(ProofOfAgreement { msg: Msg { gen: 1, op: OpMove { timestamp: Clock { actor_id: i:35240f, counter: 1 }, parent_id: 1, metadata: 100, child_id: 3 }, dot: i:35240f.1 }, proof: {i:35240f: sig:d924dc, i:51c6d3: sig:03f99b} }), sig: sig:8d5849 }

[P2P] Sending message to 127.0.0.1:46915 --> Packet(Packet { source: i:35240f, dest: i:35240f, payload: BRB(ProofOfAgreement { msg: Msg { gen: 1, op: OpMove { timestamp: Clock { actor_id: i:35240f, counter: 1 }, parent_id: 1, metadata: 100, child_id: 3 }, dot: i:35240f.1 }, proof: {i:35240f: sig:d924dc, i:51c6d3: sig:03f99b} }), sig: sig:8d5849 })

[P2P] received from 127.0.0.1:46915 --> Packet(Packet { source: i:35240f, dest: i:35240f, payload: BRB(ProofOfAgreement { msg: Msg { gen: 1, op: OpMove { timestamp: Clock { actor_id: i:35240f, counter: 1 }, parent_id: 1, metadata: 100, child_id: 3 }, dot: i:35240f.1 }, proof: {i:35240f: sig:d924dc, i:51c6d3: sig:03f99b} }), sig: sig:8d5849 })

[P2P] delivering packet to i:51c6d3 at addr 127.0.0.1:48479: Packet { source: i:35240f, dest: i:51c6d3, payload: BRB(ProofOfAgreement { msg: Msg { gen: 1, op: OpMove { timestamp: Clock { actor_id: i:35240f, counter: 1 }, parent_id: 1, metadata: 100, child_id: 3 }, dot: i:35240f.1 }, proof: {i:35240f: sig:d924dc, i:51c6d3: sig:03f99b} }), sig: sig:8d5849 }

[P2P] Sending message to 127.0.0.1:48479 --> Packet(Packet { source: i:35240f, dest: i:51c6d3, payload: BRB(ProofOfAgreement { msg: Msg { gen: 1, op: OpMove { timestamp: Clock { actor_id: i:35240f, counter: 1 }, parent_id: 1, metadata: 100, child_id: 3 }, dot: i:35240f.1 }, proof: {i:35240f: sig:d924dc, i:51c6d3: sig:03f99b} }), sig: sig:8d5849 })

[P2P] router cmd Apply(Packet { source: i:35240f, dest: i:35240f, payload: BRB(ProofOfAgreement { msg: Msg { gen: 1, op: OpMove { timestamp: Clock { actor_id: i:35240f, counter: 1 }, parent_id: 1, metadata: 100, child_id: 3 }, dot: i:35240f.1 }, proof: {i:35240f: sig:d924dc, i:51c6d3: sig:03f99b} }), sig: sig:8d5849 })

[BRB] handling packet from i:35240f->i:35240f

[BRB] proof of agreement: Msg { gen: 1, op: OpMove { timestamp: Clock { actor_id: i:35240f, counter: 1 }, parent_id: 1, metadata: 100, child_id: 3 }, dot: i:35240f.1 }


> read
1
  3 [100]
```

`Peer A` also sees/agrees that the tree has been modified.

```
> read
1
  3 [100]
```


## License

This Safe Network software is dual-licensed under the Modified BSD (<LICENSE-BSD> <https://opensource.org/licenses/BSD-3-Clause>) or the MIT license (<LICENSE-MIT> <https://opensource.org/licenses/MIT>) at your option.

## Contributing

Want to contribute? Great :tada:

There are many ways to give back to the project, whether it be writing new code, fixing bugs, or just reporting errors. All forms of contributions are encouraged!

For instructions on how to contribute, see our [Guide to contributing](https://github.com/maidsafe/QA/blob/master/CONTRIBUTING.md).
