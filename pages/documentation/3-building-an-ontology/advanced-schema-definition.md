---
title: Defining an Advanced Schema
keywords: schema
last_updated: August 23, 2016
tags: [graph-api, java, advanced-grakn]
summary: "Demonstrates how to create an advanced schema for Grakn."
sidebar: documentation_sidebar
permalink: /documentation/building-an-ontology/advanced-schema-definition.html
folder: documentation
comment_issue_id: 22
---

{% include warning.html content="This page is deprecated and will soon be removed." %}

This document introduces some advanced schema concepts, which you can use to create more expressive graphs. We recommend that you read our document about [defining a simple schema](simple-schema-definition.html) before continuing.

Here, we also discuss creating a graph which supports Grakn reasoning and inference features.

## Sub-Classing Types

Let's say we wanted to model a graph that encapsulates films and tv shows, as well as the actors who can play in them. The naive way to do this would be as follows:

Create the Entity types:

```java
EntityType person = graknGraph.putEntityType("Person");
EntityType movie = graknGraph.putEntityType("Movie");
EntityType tvShow = graknGraph.putEntityType("TV Show");
```

Create the roles and relation types:

```java
RoleType actorInMovie = graknGraph.putRoleType("Actor In Movie");
RoleType movieCastIn = graknGraph.putRoleType("Movie Cast In");
RelationType starsInMovie = graknGraph.putRelationType("Stars In Movie").hasRole(actorInMovie).hasRole(movieCastIn);

RoleType actorInTvShow = graknGraph.putRoleType("Actor In Tv Show");
RoleType tvShowCastIn = graknGraph.putRoleType("Tv Show Cast In");
RelationType starsInTvShow = graknGraph.putRelationType("Stars In Tv Show").hasRole(actorInTvShow).hasRole(tvShowCastIn);
```

Allow the roles to be played:

```java
person.playsRole(actorInMovie).playsRole(actorInTvShow);
movie.playsRole(movieCastIn);
tvShow.playsRole(tvShowCastIn);
```

The above schema is valid and will allow us to model our data. However, there are a lot of duplicate definitions. For example, an actor should be able to star in anything (ideally): a film, a tv show, a theatre production etc.  Furthermore, there is a lot of contextual overlap between films and TV shows.

A more sophisticated approach uses sub-classing, which allows us to define things that fall into categories and sub-categories.
We can create a deep object-orientated schema where the rules of the super-class provide additional functionality to its sub-classes.

Let's convert our schema into something that is more expressive.   

Films and TV shows are similar, so we should have a super-class `Production` to represent both:

```java
EntityType production = graknGraph.putEntityType("Production");
movie.superType(production);
tvShow.superType(production);
```

Actors should be able to star in any type of `Production`, be it a film or a TV Show:

```java
RoleType actor = graknGraph.putRoleType("Actor");
RoleType productionCastIn = graknGraph.putRoleType("Production Cast In");
RelationType starsIn = graknGraph.putRelationType("Stars In").hasRole(actor).hasRole(productionCastIn);
```

Allow the role to be played:

```java
person.playsRole(actor);
production.playsRole(productionCastIn);
```

This schema is more expressive but is also shorter because the common role type (e.g. *Actor*) are inherited through a super-class.
This also makes defining the schema shorter and gives more opportunities for graph enhancements.
For example, we can start to differentiate between male and female actors without impacting the schema significantly:

```java
EntityType man = graknGraph.putEntityType("Man").superType(person);
EntityType woman = graknGraph.putEntityType("Woman").superType();
```

## Rule Types

Grakn supports rule-based backward-chained (BC) reasoning to allow automated capturing and evolution of patterns within the graph. The BC approach is a goal-driven approach to reasoning where the subset of applicable rules is controlled by a particular query. In the BC mode, all the inferrable information is available at query time.

The inference rules are expressed in the following general form:

```
if [rule-body] then [rule-head]
```

or in Prolog/Datalog terms:

```
[rule-head] :- [rule-body]
```

In logical terms, we restrict the rules to be definite Horn clauses (i.e. disjunctions of atoms with at most one unnegated atom). In our system we define both the head and the body of rules as Graql queries. Consequently, the rules are statements of the form:

```
p :- q1, q2, ..., qn
```
where p and q's are atoms that each correspond to a single Graql statement.


A classic reasoning example is the ancestor example: the two rules stated below define the ancestor (anc) relationship, given a parenthood relationship (parent) between individuals:

```
R1: anc(X, Y) :- parent(X, Y)
R2: anc(X, Y) :- parent(X, Z), anc(Z, Y)
```

### Rule Java API
All rule instances are of type inference-rule which can be retrieved by:

```java
RuleType inferenceRule = graknGraph.getMetaRuleInference();
```

Rule instances can be added to the graph both through the Graph API as well as through Graql. Considering the ancestor example, with the use of the Graph API we can add the rules in the following way:

```java
Pattern r1Body = var().rel("parent", "x").rel("child", "y").isa("Parent");
Pattern r1Head = var().rel("ancestor", "x").rel("descendant", "y").isa("Ancestor");

Pattern r2Body = and(
        var().rel("parent", "x").rel("child", "y").isa("Parent')"),
        var().rel("ancestor", "z").rel("descendant", "y").isa("Ancestor")
);
Pattern r2Head = var().rel("ancestor", "x").rel("descendant", "y").isa("Ancestor");

Rule rule1 = inferenceRule.addRule(r1Body, r1Head);
Rule rule2 = inferenceRule.addRule(r2Body, r2Head);
```

### Rule Graql Syntax
The addition of the rules specified above can be expressed via an insert Graql statement where the body and the head of the rule are separated with curly braces, the corresponding statement then reads:

```graql
insert
isa inference-rule,
lhs {
    (parent: $x, child: $y) isa Parent;
},
rhs {
    (ancestor: $x, descendant: $y) isa Ancestor;
};

isa inference-rule,
lhs {
    (parent: $x, child: $z) isa Parent;
    (ancestor: $z, descendant: $y) isa Ancestor;
},
rhs {
    (ancestor: $x, descendant: $y) isa Ancestor;
};
```

{% include links.html %}

## Comments
Want to leave a comment? Visit <a href="https://github.com/graknlabs/docs/issues/22" target="_blank">the issues on Github for this page</a> (you'll need a GitHub account). You are also welcome to contribute to our documentation directly via the "Edit me" button at the top of the page.
