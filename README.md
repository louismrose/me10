## Introduction

This website provide examples to support our paper "Comparing Model-Metamodel and Transformation-Metamodel Co-evolution, *Louis M. Rose, Anne Etien, David Mendez, Dimitrios S. Kolovos, Richard F. Paige and Fiona A.C. Polack*, in Proc. Workshop on Model Evolution 2010, co-located with MoDELS 2010, Oslo, Norway."

The paper discusses model-metamodel co-evolution and transformation-metamodel co-evolution. This website gives an example of each type of co-evolution, and re-uses the example of an evolution of a Petri net metamodel from [1][cicchetti], [2][garces], [3][rose] and [4][wachsmuth].

The metamodels, models and transformations are available as a [zip file](http://github.com/downloads/louismrose/me10/resources.zip), or by checking out this repository using Git:

    git clone http://github.com/louismrose/me10.git

## Metamodel Evolution

Suppose that we have the following metamodel:

![Original Petri nets metamodel](me10/raw/master/images/before.png)

Here a Petri Net comprises Places and Transitions. A Place has any number of source or destination Transitions. Similarly, a Transition has at least one source and destination Place. 

The metamodel is to be evolved so as to support weighted connections between Places and Transitions and between Transitions and Places. The evolved metamodel is shown below. Places are connected to Transitions via instances of PTArc. Likewise, Transitions are connected to Places via TPArc. Both PTArc and TPArc inherit from Arc, and therefore can be used to specify a weight.

![Evolved Petri nets metamodel](me10/raw/master/images/after.png)

The metamodel evolution might affect the conformance of Petri net models and the domain conformance of transformations that use the Petri nets metamodel. Exemplar model and transformation migration strategies for the Petri net metamodel evolution are discussed below.

Various approaches to formulating migration strategies have been proposed, and are discussed in the paper. These approaches vary in the extent to which they require guidance from the metamodel developer. For the metamodel evolution described above, the metamodel developer might envisage the following migration strategies.


## Model Migration
The following strategy is envisaged for model migration:

* For every instance, t, of Transition: 
  * For every Place, s, referenced by the src feature of t: 
      * Create a new instance, arc, of PTArc. 
      * Set s as the src of arc. 
      * Set t as the dst of arc. 
      * Add arc to the arcs reference of the Net referenced by t.
  * For every Place, d, referenced by the dst feature of t: 
      * Create a new instance, arc, of TPArc. 
      * Set t as the src of arc. 
      * Set d as the dst of arc. 
      * Add arc to the arcs reference of the Net referenced by t.
  * And nothing else changes.


## Transformation Migration
The listing below shows an exemplar transformation, written in QVTO, that consumes a graph (an instance of a metamodel comprising graph, node and edge metaclasses), and produces a Petri net.

    1.  mapping graph::Graph::toNet(): PN::Net {
    2.  	places+=self.nodes->map toPlace();
    3.  	transitions+=self.edges->map toTransition();	
    4.  }
    5. 
    6.  mapping graph::Node::toPlace(): PN::Place {
    7.  	name:=self.name;
    8.  	src:=self.is_source->map toTransition();
    9.  	dest:=self.is_target->map toTransition();
    10. }
    11. 
    12. mapping graph::Edge::toTransition(): PN::Transition {
    13.     name:=self.name;
    14. }

Following the Petri net metamodel evolution described above, migration of the transformation results in the listing below. Note that lines 8 and 9 of the original transformation, which populate the src and dst features of Transition, have been migrated. The equivalent code in the evolved transformation, on lines 10 and 11, now populates the src and dst features with subclasses of the Arc metaclass, which was introduced in the evolved metamodel (shown above). Note also that in and out are QVTO keywords, and features named in and out must be prefixed with an underscore (\_), as shown on lines 10 and 11.

    1.  mapping graph::Graph::toNet(): EPN::Net {
    2.      places+=self.nodes->map toPlace();
    3.      transitions+=self.edges->map toTransition();
    4.      arcs+=self.edges->map toPTArc();
    5.      arcs+=self.edges->map toTPArc();
    6.  }
    7.
    8.  mapping graph::Node::toPlace(): EPN::Place {
    9.      name:=self.name;
    10.	    _in:=self.is_source->map toTPArc();
    11.	    _out:=self.is_target->map toPTArc();
    12. }
    13.
    14. mapping graph::Edge::toTransition(): EPN::Transition {
    15.     name:=self.name;
    16. }
    17. 
    18. mapping graph::Edge::toTPArc(): EPN::TPArc{
    19.     src:=self->map toTransition();
    20. }
    21.
    22. mapping graph::Edge::toPTArc(): EPN::PTArc{
    23.     dest:=self->map toTransition();
    24. }


In general, the following transformation migration strategy is envisaged for transformations that use the Petri nets metamodel as a target:

* In rules creating a Place: 
  * Replace every statement that refers to src with a statement that refers to \_in, and invokes a TPArc rule rather than a Transition rule.
  * Replace every statement that refers to dst} with a statement that refers to \_out, and invokes a PTArc rule rather than a Transition rule.
* In rules creating a Net: 
  * Add statements to populate the arcs feature.
* Add a rule creating TPArcs, which:
  * Populates the src feature. 
* Add a rule creating PTArcs, which:
  * Populates the dst feature.
* And nothing else changes.
	

[cicchetti]: http://dx.doi.org/10.1109/EDOC.2008.44  "A. Cicchetti, D. DiRuscio, R. Eramo, and A.Pierantonio. Automating co-evolution in MDE. In Proc. EDOC, 2008."
[garces]: http://dx.doi.org/10.1007/978-3-642-02674-4_4  "K. Garces, F. Jouault, P. Cointe, and J. Bezivin. Managing model adaptation by precise detection of metamodel changes. In Proc. ECMDA-FA, 2009."
[rose]: http://dx.doi.org/10.1007/978-3-642-13688-7_13  "L.M. Rose, D.S. Kolovos, R.F. Paige, and F.A.C. Polack. Model migration with Epsilon Flock. In Proc. ICMT, 2010."
[wachsmuth]: http://dx.doi.org/10.1007/978-3-540-73589-2_28  "G. Wachsmuth, Metamodel Adaptation and Model Co-adaptation. In Proc. ECOOP, 2007"