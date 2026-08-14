# How to make a git-like graph database

Suppose you could branch and merge a database just like you can a Git repository. We will call this capability generically *version control*. Because branching, committing, and merging describe the edges of [a graph](https://en.wikipedia.org/wiki/Version_control#Graph_structure) with the instantaneous state of the database, confusingly called commits,[^commits] as the nodes, a graph database is a more natural paradigm than a traditional table-relational database. In fact, a little thought will reveal that a version control system on a graph database is the most generic version control system possible. After all, any dataset can be represented as a graph, as can the changes made to that dataset. 

As an aside, TerminusDB is the only real-world database technology I am aware of that supports version control natively. TerminusDB started as a graphical database and then adopted version control afterward. In this article is only meant to describe how *I* would implement a database like TerminusDB, not how TerminusDB is *actually* implemented, about which I have no idea.

The design challenge is to devise a system with the following features:

1. supports version control natively, specifically:

   a. branching

   b. merging

   c. checking out previous revisions

2. stores data in a reasonably space efficient way

3. can efficiently read from and write to the data store

4. admits concurrent read-write access

How to store data efficiently is a well-studied problem domain. What we will discover is that a single datastructure is the solution to all of our design challenges. Moreover, you are likely already very familiar with it, because it is how Git works. The underlying data structure is called a Merkle tree.[^hashtree]

## Merkle Trees



A Merkle tree is a tree in which each leaf node contains some data payload and the hash of the data, and each non-leaf node contains the hash of the list of hashes of all of its child nodes. Any tree data structure can be made into a Merkle tree by just labeling its nodes with hashes. This gives us a lot of utility: the hashes of two trees match if and only if the trees are equal.[^hashcollision] Thus,

* If two trees have the same hash, only one copy of the tree needs to exist—at least until one of the trees is mutated. (More on this later.) Consequently, multiple distinct graphs can share the same subgraph.
* It is trivial to determine if a change has occurred in the tree by comparing the last known hash of the root node to the current hash. If a change *has* occurred… 
* …it is easy to determine which part of the tree has changed by following the path of the differing hashes.

## Tries

A *trie* (pronounced /triː/ or /traɪ/, from re***trie***val) is a search tree in which all descendants of a node have a common prefix of the string associated with that node. Here is an example from Wikipedia in which seven words are encoded:

![](https://upload.wikimedia.org/wikipedia/commons/a/ae/Patricia_trie.svg)

Checking containment, adding, and deleting a string in a trie are all very efficient. Space is also well conserved, as common prefixes are combined and stored only once.





[^commits]: This is a misnomer. A *commit* produces a *revision*. Since commits and revisions are in 1-to-1 correspondence, revisions are often conflated with the commit that produced them, and people use the word commit to refer to a revision.
[^ hashtree ]: Some people call Merkel trees *hash trees*.
[^hashcollision]: Strictly speaking, the equality of the trees is not guaranteed. Mathematicians must go to great pains to deliberately construct hash collisions. The probability of a hash collision using any reasonable hash function is so close to zero that even God can't tell the difference.