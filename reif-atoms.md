# Proposal - atomic reification

This is a reificiation propsoal.

The agreed syntax applies in Turtle.

The problems with triples for reification are:

1. It is verbose in Turtle and N-Triples.
2. It can be over-specificed (e.g. multiple rdf:subject)
3. It can be under-specificed (e.g. no rdf:subject)
4. Presence or absense of rdf:Statement is unclear leading to different SPARQL results.  
   (De facto, it is omitted.)
5. The reification triples may be split across parts of a document.
6. If a large graph is split into multiple files of mnanageable size, it can be split across those files (and blank nodes for reification are broken).

Agreed Turtle syntax hides some of these issues but not for N-triples, and not for SPARQL when thought as querying a triple table.

## General idea

Introduce a unit "reification atom" that means the rdf:subject/rdf:predicate/rdf:object is not split.

There is a syntax token "descriptor"; the name is choosen pro tem so that it does not presume too much.

It is distinct from 'triple' in the abstract syntax.

### Abstract Syntax
```
    graph            ::= (triple)* 
    triple           ::= subject predicate object 
    subject          ::= iri | BlankNode | descriptor 
    predicate        ::= iri 
    object           ::= term 
    term             ::= iri | BlankNode | literal | descriptor 
    descriptor       ::= descriptor(s p o)
```

Given a descriptor `x`, we denote the subject, predicate, object positions of `x` as `x.s`, `x.p`, `x.o`, respectively.  

A _named occurrence_ is a resource associated with a "reification atom" via a triple.

"named triple" is another possible name which may be more accessible to the general reader of RDF specs.

```
    M rdf:occurrenceOf descriptor .
```
where descriptor is the syntax token (RDF term) for a reification atom.

## The idea - Formalism 1 - via interpretation

Put "reification atom" as a resource into IR, the universe of I.
Put interpretation of a "reification atom" into I.

It is "along side" the reification vocabulary. It provides the same capability, a named token for a triple, but in an atomic form.

## The idea - Formalism 2 - via translation

Translate "descriptor" into RDF 1.1 reification before interpretation.

Semantic intepretation is unchanged from RDF 1.1.

This triple translation does show through into SPARQL meaning it SPARQL is not "querying the abstracts syntax". More in the detailed description.

## Concrete Syntax Implications

### Turtle

As agreed.

### N-Triples

A "descriptor" is syntactic representation of a reification atom `<<(s p o)>>`. It is a token/terminal in the grammar.

Descriptors can be written in Turtle but agreed syntax makes that a burdensome way to write RDF.

The triples `M rdf:occurrenceOf AR` correspond to the "edge set" and can be implemented that way, or as secondary indexes, for better handling of searching AR.

## SPARQL Implications

SPARQL works on the abstract syntax.

For data:
```
   _:b rdf:occurrenceOf <<(:s :p :o )>> .
```
the query
```
   SELECT (count(*) AS ?c) { ?s ?p ?o }
```
returns `?c = 1`.

For data assert-occurrence-annotate:
```
   :s :p :o {| :a :b |}
```
```
   :s :p :o .
   _:b rdf:occurrenceOf <<( :s :p :o )>> .
   _:b :a :b .
```
the query
```
   SELECT (count(*) AS ?c) { ?s ?p ?o }
```
returns `?c = 3`.

"Reification" cost is one triple.

```
   SELECT ?N { ?N rdf:occurrenceOf <<( :s :p :o )>> }
```
is all names for a specific reification atom.

```
   SELECT DISTINCT ?N { ?N rdf:occurrenceOf <<( :s ?p ?o )>> }
```
is all names for reification atoms that use subject `:s`.

### Other

As well as the Turtle syntax+variables pattern matching, SPARQL would also have accessor functions
[SUBJECT](https://w3c.github.io/rdf-star/cg-spec#subject),
[PREDICATE](https://w3c.github.io/rdf-star/cg-spec#predicate),
[OBJECT](https://w3c.github.io/rdf-star/cg-spec#object)
and parallels to other forms in the [CG report](https://w3c.github.io/rdf-star/cg-spec#builtin-functions).

## Detailed Descriptions

[Interpret Reification Atoms](./reif-atoms-interpret.md)

[Translate Reification Atoms](./reif-atoms-translate.md)