# Hyper Network Generation Procedure

The concept of a hyper network is that each agency has its own “layer” of virtual links and nodes, which are only connected to the base network by auxiliary transit (walk) links that contain the transit fare cost . In this implementation, this has been generalized to have each transit line belong to a “group” (instead of a hard-coded operator), in order to give the procedure more flexibility. The groups are specified using a fare schema file (see next section for details). The main assumption of this approach is that transfers between lines within the same group are free (unless a boarding cost is applied during assignment).
To generate the appropriate hyper-network, six steps are followed:
1.	Transit lines are classified into groups based on the fare schema file.
2.	The network is pre-processed. The procedure attaches to each node a two sets of data: groups which are stopping at the node, and groups which are passing through the node. The two sets are mutually exclusive. Two groups of (non-zone) nodes are identified:
a.	Road (or Surface) nodes are connected to at least one link with the auto mode.
b.	Transit (or Station) nodes are connected to links with only transit or walk modes. 
3.	Surface nodes are processed first. For each group using a surface node (either stopping at or passing through), a new virtual node is created. The base node’s attributes are copied over. For groups stopping at the node, the associated virtual node is connected using walk links to the base node and to each other. These walk links are indexed for later use .
4.	The same procedure is followed for station nodes, except that virtual nodes are only created when two or more groups use the station node . This is done as station nodes already exist in their own layer in the network (e.g. subway links and nodes), and therefore do not need to “moved” over to a new layer. Any existing connected walk links are indexed appropriately.
5.	Virtual surface nodes and virtual station nodes are connected to each other using walk links.
6.	Finally, each transit line is moved over to its appropriate layer in the hyper network. In-vehicle links are created between virtual nodes when none exist.
It should be noted that, currently, no effort is made to make the network look visually appealing. Virtual nodes are given the exact same coordinates as their base node, and virtual links copy the same shape as their base links. Therefore, the created hyper-network appears to be identical to the base network – since the virtual nodes and links overlap the base precisely.


![alt text](images/15.png "Figure 15")

![alt text](images/16.png "Figure 16")