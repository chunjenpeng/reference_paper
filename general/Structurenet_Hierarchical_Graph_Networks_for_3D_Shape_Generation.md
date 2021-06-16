# StructureNet: Hierarchical Graph Networks for 3D Shape Generation

## Introduction

- StructureNet is a hierarchical graph network that attemps to generate novel, diverse, and realistic 3D shapes along with associated part semantics and structure.
- It works directly with n-ary graph encoding of the input shape.

**Github:** https://github.com/daerduoCarey/structurenet

## Method

**1. Structure Representation**

  <div style='text-align:center'><img src='./figure/structurenet/4-Figure4-1.png'></div>

  * A shape **S** = (**P**, **H**, **R**) is represented by a set of parts **P** = {*P1*,...,*PN*} describing the geometry of the shape, and a structure (**H**, **R**) that describes how these parts are organized and related to each other.
  * Part's geometry can be presented as its minimum bounding box **Bi** = (ci, qi, ri). In addition to geometry, each part also has a semantic label *li* (encoded as one-hot vector.)
  * **H** are directed edges from a parent part to all the children.
  * Relationship **R** between siblings are denoted as edges *({Pj, Pk}, τ)*, with *τ* is the relationship type (encoded as one-hot vector). These edges form a graph (**Ci**, **Ri**) between siblings of the hierarchy.

**2. Architecture**

  <div style='text-align:center'><img src='./figure/structurenet/6-Figure5-1.png'></div>

  * Shape respresentation is encoded into latent space by a VAE.
  * The encoder works recursively in a bottom-up manner using two type of encoders.
      - Geometry encoder: Consists of a single layer perceptron.
      - Graph encoder: encode each child graph
        * We perform several iterations of message passing along the edges of the graph. In each iteration a node aggregates feature of its neighbors and the edges between to compute an updated feature vector. <div style='text-align:center'><img src='./figure/structurenet/graph_encoder1.png'></div> *M* is the number of neighbors and *h* is a SLP.

        * The feature vector of entire graph is computer by max-pooling over all child parts
        <div style='text-align:center'><img src='./figure/structurenet/graph_encoder2.png'></div>
        * We concatenate the grah feature vectors after each iteration and pass the through another SLP.
        <div style='text-align:center'><img src='./figure/structurenet/graph_encoder3.png'></div>
  * The decoder expand nodes in a top-down fashion. First it performs the reverse operation of the **graph encoder** using **graph decoder**, then transform the resulting feature vector of each child with **geometry decoder**.
      - Graph decoder: Transform a parent feature vector into the child graph.
        * Always decode a fixed maximum number *np* of child parts and all edges between them, together with a binary probability that a predicted part of edge exists in the graph.
        * Parts and their relationship are decoded simultaneously.
        <div style='text-align:center'><img src='./figure/structurenet/graph_decoder1.png'></div>
        <div style='text-align:center'><img src='./figure/structurenet/graph_decoder2.png'></div>

        * We can recover the edges between a pair of parts based on their pair of feature vectors <div style='text-align:center'><img src='./figure/structurenet/graph_decoder3.png'></div> *gxe* is a 2-layer MLP.

        * We perform 2 message passing iterations along predicted edges to get the feature vector *fj*

        * Finally, we decode the semantic labels *lj* and a probability *p_leaf* from feature vector *fj*
      - Geometry decoder: A multi-layer perceptron with two layers that transform *fj* to a bounding box *Bj*.

**3. Training and losses**

  <div style='text-align:center'><img src='./figure/structurenet/loss.png'></div>

  * Reconstruction loss
    <div style='text-align:center'><img src='./figure/structurenet/result4.png'></div>
      - Geometry loss
        <div style='text-align:center'><img src='./figure/structurenet/geo_loss.png'></div>
      - Normal loss
        <div style='text-align:center'><img src='./figure/structurenet/normal_loss.png'></div>
      - Part existence loss
        <div style='text-align:center'><img src='./figure/structurenet/part_loss.png'></div>
      - Edge existence loss
        <div style='text-align:center'><img src='./figure/structurenet/edge_loss.png'></div>
      - Semantic loss
        <div style='text-align:center'><img src='./figure/structurenet/semantic_loss.png'></div>
      - Leaf loss
        <div style='text-align:center'><img src='./figure/structurenet/leaf_loss.png'></div>

      Total distance
      <div style='text-align:center'><img src='./figure/structurenet/distance.png'></div>

  * Structure consistency loss
      <div style='text-align:center'><img src='./figure/structurenet/loss2.png'></div>
      <div style='text-align:center'><img src='./figure/structurenet/loss3.png'></div>
      <div style='text-align:center'><img src='./figure/structurenet/loss4.png'></div>
      <div style='text-align:center'><img src='./figure/structurenet/loss5.png'></div>

## Results

* Shape generation

  <div style='text-align:center'><img src='./figure/structurenet/result2.png'></div>
  <div style='text-align:center'><img src='./figure/structurenet/result1.png'></div>
  <div style='text-align:center'><img src='./figure/structurenet/result3.png'></div>

* Limitation

    * Data-driven network, so it will have data biases.

    * May contain models with detached parts or asymetric parts.

      <div style='text-align:center'><img src='./figure/structurenet/failure.png'></div>

    * The memory cost is quadratic of number of siblings encoded.

## Discussion

- Apply StructureNet for our document images synthesize problem.