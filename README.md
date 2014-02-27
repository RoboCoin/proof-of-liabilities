# blproof - Blind liability proof

My attempt at implementing gmaxwell's
["prove-how-(non)-fractional-your-Bitcoin-reserves-are
scheme"](https://iwilcox.me.uk/v/nofrac). 

This project only implements
the "liability" part of the solvency proof (how many bitcoins the
operator **SHOULD** have - thier liabilities). 

The proof that the operator **DOES** have the assets to cover all its liabilities 
can be done using Bitcoin's signmessage feature. The complete scheme
will be described in details eventually.
 
Beer fund: 1ECyyu39RtDNAuk3HRCRWwD4syBF2ZGzdx

## Install

```
npm install blproof
```

## CLI Usage

```
# Create complete proof tree from an accounts file (see
# test/account.json for format)

$ blproof completetree -f test/accounts.json --human
$ blproof completetree -f test/accounts.json > complete.out.json

# Extract partial tree for user mark.

$ blproof partialtree mark -f complete.out.json --human
$ blproof partialtree mark -f complete.out.json > mark.out.json

# Display root node hash and value

$ blproof root -f complete.out.json --human

# Verify partial tree

$ blproof verify --hash "SLpDal8kYJNdLwczp6wrU68FOFrpoHT3w5nd15HOpwU=" --value 37618 -f mark.out.json
```

## Definitions

### Complete proof tree
  
The complete proof tree is a binary tree where the leaf nodes
represent all the user accounts and the interior nodes generated using the
NodeCombiner function described below.

The complete tree should be kept private by the operator in order to
protect the privacy of its users. Only the root node should be puslished
publicly and each individual user should have private access to their
own partial proof tree.

### Interior node (NodeCombiner function)

Interior nodes are generated using the NodeCombiner function described
below.

The node's value is equal to the sum of its two child node's values.

The node's hash is equal to the sha256 of its value concatenated with
its child's hashes.

```
function NodeCombiner (left_node, right_node) {
  var n = {};
  n.value = left_node.value + right_node.value;
  n.hash = sha256((left_node.value + right_node.value) + '' + left_node.hash + '' + right_node.hash);
  return n;
}
```

### Leaf node

Leaf nodes represent user accounts. They possess the following values:

  - `user`: A unique identifier for the user. It is necessary for a user
    to assess the uniqueness of this value so it is recommended to use their username or email.
  - `nonce`: A random nonce to prevent its neighboor node from discovering its `user` value
  - `value`: The user's balance.
  - `hash`: A sha256 hash of user concanetated with the nonce.


### Root node

The root node of the tree like all interior nodes possesses a hash and a
value. This data must be published publicly as a way to prove that all users
are part of the same proof tree.


### Partial proof tree

A partial proof contains only the nodes from the complete root a given
user needs to verify he was included in the tree. 

It can be generated by starting from the user's leaf node and moving up
the tree until reaching the root node. Then the sibblings of each
selected node on the path must be added to the tree. The user's leaf
sibling which is also a leaf node must be stripped of its `user` and
`nonce` values so that only the `hash` and `value` remain.

Partial trees should be disclosed privately to each individual users so
they can verify the proof.

## Serialized data formats (work in progress / draft)

This section is intended to standardize the way root nodes and trees are
distributed in order to make implementations compatible. 

All formats are based on JSON. 

### Root node:

```javascript
{ 
  "root": {
    "value": 37618,
    "hash": "2evVTMS8wbF2p5aq1qFETanO24BsnP/eshJxxPHJcug="
  }
}
```

### Partial trees:

Partial trees are represented as a `node` object graph. They have the
following format:

```javascript
{ 
  "partial_tree": <node>
}

```

Nodes have the following format:

```javascript
{
  "parent": <node>,
  "left": <node>,
  "right": <node>,
  "data": <node_data>
}
```

`<node_data>` is an object which must contain the following keys:

- `value` number
- `hash` string
- `user` string (optional) Only the node belonging to the user this partial tree
  was generated for should have this key set.
- `nonce` number (optional) Only the node belonging to the user this partial tree
  was generated for should have this key set.

## Some sample outputs

```
$ ./cli.js completetree -f test/accounts.json --human
37618, ARoRyENGrJyGieLK79OuQWuSE/1znzOLyWAOYmAECOw=
 |_ 24614, PgWlVmieGIUq+g3l0H/+er8BXaA0mNjc8jcxc8uTIZg=
 | |_ 21072, WpPDcPcLHyRqzKnvYXAvipVFPplDzgKtZqqRJz3BO94=
 | | |_ 4167, nvTw6l/trAzY07BJW7XLFZ8w9EGtK65UEO95c5b9UPA=
 | | | |_ 122, LD2m4AZBVNuTBazJfYf31hIvIB5WA+oZ0jeqlJnzMe4=
 | | | | |_ 39, einstein, 0.27035144134424627, NuW19GTHeDh8kem5E/DfFdDHWVzELqVak6IuuIGR+Ck=
 | | | | |_ 83, picasso, 0.19446203880943358, EI3jlqZjR74zphiajQHDsmAXb8/qEgofHyHysiS5sAc=
 | | | |_ 4045, olalonde, 0.323900283081457, h3N10zqNemNHMJ31aqLUNCUQ7rxQFRkRzz1Mayyz2cI=
 | | |_ 16905, SvJCozx3KLid8gyChfCQSNJe5W6Kf6AZy1sTAuQ1LAU=
 | |   |_ 6905, gmaxwell, 0.7565273905638605, 5Crv8D6gIPVcwaRuEL1CPj/6KvvQkuMcPkPO6JnpdZg=
 | |   |_ 10000, satoshi, 0.7995808166451752, F8dXs5Cf/IlFSxpuAU2idJrVLt9B0H/o4dgO8cMI5mA=
 | |_ 3542, CZcqeTmpSRNVj6OoGzesDsfJThLLuGZU5oI+lBRuimw=
 |   |_ 3327, M1BxypagXgZDXEIDIjOpNOrG1yQPnYGR2AnghBSXJf8=
 |   | |_ 300, luke-jr, 0.552029364509508, afE1DC5LV0y6WfZuiC5H5+yq3mKSBlJJuQr/Yj4mtes=
 |   | |_ 3027, sipa, 0.8273184080608189, v3ul5BLGkLU0sOaMAMNXHUP6q8wYvXzB+yUrIujHe7I=
 |   |_ 215, 68oIqC3/xs90UDrIY2R8UaJ/sbZYqevMGal6845UDJs=
 |     |_ 200, codeshark, 0.9785103949252516, K4OBBdecagopyk+MVyeH15R19Zj3GTB0llG5Dm1bvOg=
 |     |_ 15, gribble, 0.06026308285072446, qGifo7VltIcTX1aFmdl5KEXB/3HI6OgYgU8gF0oUgoQ=
 |_ 13004, b6nOnnYmEOCR6LmdfG2KUeXjudfla97B6CaMUUKNN60=
   |_ 9901, pgpCn3hILMxj01Dba3zJ9TsQwepRcEBdJHoAV4ywx64=
   | |_ 12, +QaapnLGXcJ3Hjya8Xo1Ju89fn5FgB2ViPvjcrUO9EM=
   | | |_ 0, alice, 0.16321996343322098, uNKPPjtmNpeX19AJg0DTt6F9l1Hta2H7miQxqgtHtgs=
   | | |_ 12, bob, 0.45615525753237307, UUhgdge7KiWplNdCNMG1lW05aQgnGdY5kuDcceCyujA=
   | |_ 9889, 5M/VQ9wGbg4xFmLZMRcNr6qpkt9P7fRuCHQEv3ovha0=
   |   |_ 9427, charlie, 0.5375598601531237, 91R6nQU+uOE63+PiAmj3SQXL6CPaF1SFDgbYSJImBtk=
   |   |_ 462, mark, 0.6156334422994405, KV1Rx2kmM3PZkp6vVJ0Dc8Rgj/6bOuxNmp941hIHmqc=
   |_ 3103, jrhEVauiHrtRFsfRLPfL/TdoNalXD52eOaPONAJlO0g=
     |_ 3032, GIehVY5ZEwefbBBvP2LXN551RbDFicdmHLlu2LWPa9E=
     | |_ 12, anax, 0.7285208490211517, MD6AkV09BiyJPNPgNbdqYmUJgj0l1vUcIwolKIBA4Kg=
     | |_ 3020, gavin, 0.34977371199056506, y2lMSrLc8M5LPVV062iNib0P5we9gXtV8fxWYGZx9vU=
     |_ 71, Oy1k8UR/dVPhODv8cFxm6/4oZReoMCK30DgmCcLxAws=
       |_ 68, stacy, 0.24734967038966715, 4hv2lQy2lCN6Z3y70empHCWIBmzJ7lebeFjIub0mgMg=
       |_ 3, justin, 0.786616450175643, jB+xQC8s8IGL9cB+R9I2zEGGOAMV723LcM8/SnYC5ps=

$ ./cli.js completetree -f test/accounts.json > complete.out.json

$ ./cli.js partialtree mark -f complete.out.json --human
37618, 2evVTMS8wbF2p5aq1qFETanO24BsnP/eshJxxPHJcug=
 |_ 24614, vZRQ3uGn4Kr4490RcjN6a7ddZjDtiP09MFXGPVtHdTE=
 |_ 13004, pTk3eTymd+Eee6hIsw9pImID+QlkRa3ro8QCaaUMn+E=
   |_ 9901, WNH+sUQcyV9VECsnFSrVUmpdJ7+d0XAVpClonR9osZI=
   | |_ 12, eP0+1iEkq0uJJQLD4bZbn8duggxNUdoSHsPbjPYL5Ng=
   | |_ 9889, wY6rZAEZvn2Ts9UxcEB22dbnxD01LHKCX9em45tboq4=
   |   |_ 9427, charlie, 0.6493051161523908, 4P3d/9OMhV2otvzmc0wxNt2FXYwMaA2ANt8Grm5iLuI=
   |   |_ 462, mark, 0.9704850744456053, g3F8hhQ5YGIzsBz5YeoQpOeZHXEIDneMdHTd9/TiyBM=
   |_ 3103, /I8csAnivlstZvjS+1C5ecbEt3rm4SbLOFdm5fwewjE=

$ ./cli.js partialtree mark -f complete.out.json > partial.out.json

$ ./cli.js root -f complete.out.json
{"value":37618,"hash":"2evVTMS8wbF2p5aq1qFETanO24BsnP/eshJxxPHJcug="}

$ ./cli.js verify --value 37618 --hash "2evVTMS8wbF2p5aq1qFETanO24BsnP/eshJxxPHJcug=" -f partial.out.json
Partial tree verified successfuly!
```

## References

- [Reddit announcement](http://www.reddit.com/r/Bitcoin/comments/1yzil4/i_implemented_gmaxwells/)
- [HN post](https://news.ycombinator.com/item?id=7277865)
- [Example of how a shared wallet website could use the CLI](http://www.reddit.com/r/Bitcoin/comments/1yzil4/i_implemented_gmaxwells/cfp50ib)


