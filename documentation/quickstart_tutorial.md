---
title: Quickstart Tutorial
---
# Quickstart Tutorial

This tutorial has been designed to teach you how to load both an ontology and
some data into a Mindmaps Graph. To do this we’re going to model how
philosophers are related!

Don't worry if you've already loaded some data into your graph. A Mindmaps
Graph can contain as many ontologies as you want.

> Skip this box if you will be using Graql exclusively. For this tutorial we
> will be creating an instance of Mindmaps graph which will use Tinkergraph as
> the storage backend. [TinkerGraph](http://www.tinkerpop.com/docs/3.0.0.M1/)
> is a simple in memory graph. This is how you can initialise a Mindmaps Graph
> with a TinkerGraph backend:
> 
> ```java
> MindmapsGraphFactory mindmapsGraphFactory = MindmapsTinkerGraphFactory.getInstance();
> MindmapsGraph mindmapsGraph = mindmapsGraphFactory.newGraph();
> ```
> A TitanGraph Backend can be started with:
> ```java
> MindmapsGraphFactory mindmapsGraphFactory = MindmapsTitanGraphFactory.getInstance();
> MindmapsGraph mindmapsGraph = mindmapsGraphFactory.newGraph(\"../conf/titan-cassandra-es.properties\");
> ```

## Concepts

> The examples provided with mindmaps.zip contains this quickstart tutorial.

We'll first add an ontology element - an entity type `person`...

```sql
bin/graql.sh
insert person isa entity-type;
```
```java
EntityType person = mindmapsGraph.putEntityType("person");
```

...and a bunch of ancient Greeks:

```sql
insert "Socrates" isa person;
insert "Plato" isa person;
insert "Aristotle" isa person;
insert "Alexander" isa person;
```
```java
Entity socrates = mindmapsGraph.putEntity("Socrates", person);
Entity plato = mindmapsGraph.putEntity("Plato", person);
Entity aristotle = mindmapsGraph.putEntity("Aristotle", person);
Entity alexander = mindmapsGraph.putEntity("Alexander", person);
```

> **Ancient History**
>
> Graql shell maintains a history of past commands with the 'up' and 'down'
> arrows. You can also autocomplete keywords, type and variable names using
> 'tab'!"

![](/docs/images/phil.png)

Now we've created a super-simple graph, with one concept type and four concept
instances.

> **Types**
>
> Every concept needs a type, using `isa`. `Plato` `is a` `person` and `person`
> `is a` `entity-type`.

We'll quickly check our data has loaded:

```sql
match $p isa person

$p id "Aristotle" isa person;
$p id "Plato" isa person;
$p id "Socrates" isa person;
$p id "Alexander" isa person;
```
```java
System.out.println("Instances of Person:");
person.instances().forEach(i -> System.out.println(i.getId()));
```

> **Variables**
>
> Graql variables start with a `$`. They represent "wildcards", and are
> returned as results in `match` queries. A variable name can contain
> alphanumeric characters, dashes and underscores.

Next, let's add some `schools` of thought:

```sql
insert school isa entity-type;
insert "Peripateticism" isa school;
insert "Platonism" isa school;
insert "Idealism" isa school;
insert "Cynicism" isa school;
```
```java
EntityType school = mindmapsGraph.putEntityType("school");
Entity peripateticism = mindmapsGraph.putEntity("Peripateticism", school);
Entity platonism = mindmapsGraph.putEntity("Platonism", school);
Entity idealism = mindmapsGraph.putEntity("Idealism", school);
Entity cynicism = mindmapsGraph.putEntity("Cynicism", school);
```

And look up one:

```sql
match $cyn id "Cynicism"

$cyn id "Cynicism" isa school;
```
```java
mindmapsGraph.getEntity("Cynicism").getId();
```

## Relations

How do we define a "philosopher"? Very smart people have argued for a very long
time about this, but we're going to put our fingers in our ears and say a
philosopher is someone who practices a philosophy.

First, we define a `relation-type` called `practice` that relates a
`philosopher` to their `philosophy`:

```sql
insert practice isa relation-type;
insert philosopher isa role-type;
insert philosophy isa role-type;
insert practice has-role philosopher, has-role philosophy;
```
```java
RoleType philosopher = mindmapsGraph.putRoleType("philosopher");
RoleType philosophy = mindmapsGraph.putRoleType("philosophy");
RelationType practice = mindmapsGraph.putRelationType("practice")
    .hasRole(philosopher).hasRole(philosophy);
```

> **I don't like typing!**
>
> Commas are totally optional in query patterns, the `has-role` line above
> could be rewritten as:
> ```sql
> insert practice has-role philosopher has-role philosophy
> ```
> Similarly, we could insert `practice` like this:
> ```sql
> insert practice, isa relation-type;
> ```

A `person` can be a `philosopher` and a `school` can be a `philosophy`:

```sql
insert person plays-role philosopher;
insert school plays-role philosophy;
```
```java
person.playsRole(philosopher);
school.playsRole(philosophy);
```

> **graql commit -am "changed some stuff"**
> 
> The changes you've made to the graph haven't been saved yet! Type `commit` in
> the Graql shell to commit any changes.

Let's relate some `philosophers` to their `philosophies`:

```sql
insert (philosopher "Socrates", philosophy "Platonism") isa practice;
insert (philosopher "Plato", philosophy "Idealism") isa practice;
insert (philosopher "Plato", philosophy "Platonism") isa practice;
insert (philosopher "Aristotle", philosophy "Peripateticism") isa practice;
```
```java
mindmapsGraph.putRelation(practice)
    .putRolePlayer(philosopher, socrates).putRolePlayer(philosophy, platonisim);
mindmapsGraph.putRelation(practice)
    .putRolePlayer(philosopher, plato).putRolePlayer(philosophy, idealism);
mindmapsGraph.putRelation(practice)
    .putRolePlayer(philosopher, plato).putRolePlayer(philosophy, platonisim);
mindmapsGraph.putRelation(practice)
    .putRolePlayer(philosopher, aristotle).putRolePlayer(philosophy, peripateticism);
```

![](/docs/images/practice.png)

A relation comprises of pairs of role types and role players. For example,
`Plato` is playing the role of `philosopher`. Our relations also have a type
just like normal concepts.

Now we can query for all our Platonists:

```sql
match (philosopher $phil, "Platonism") isa practice;

$phil id "Socrates" isa person;
$phil id "Plato" isa person;
```
```java
platonisim.relations(philosophy).forEach(relation -> {
    relation.rolePlayers().values().forEach(rolePlayer -> {
        if (!rolePlayer.equals(platonisim))
            System.out.println("    -> " + rolePlayer.getId());
    });
});
```

> **In Plain English...**
>
> This query can be read as "get me philosophers in a practice relationship
> with Platonism."
>
> We didn't specify the role of `Platonism` in this query, or the type of
> `$phil`, which is totally fine! Roles and types can be omitted in queries.
> For example, the query `match ($x, $y)` is a valid query (that will
> find *everything* in a relationship with *anything*).

Next let's talk about the relationships between our philosophers. Socrates
taught Plato, Plato taught Aristotle and Aristotle even taught Alexander the
Great!

First, our ontology:

```sql
insert education isa relation-type;
insert teacher isa role-type;
insert student isa role-type;
insert education has-role teacher, has-role student;
insert person plays-role teacher, plays-role student;
```
```java
RoleType teacher = mindmapsGraph.putRoleType("teacher");
RoleType student = mindmapsGraph.putRoleType("student");
RelationType education = mindmapsGraph.putRelationType("education")
    .hasRole(teacher).hasRole(student);
person.playsRole(teacher).playsRole(student);
```

Second, our data:

```sql
insert (teacher "Socrates", student "Plato") isa education;
insert (teacher "Plato", student "Aristotle") isa education;
insert (teacher "Aristotle", student "Alexander") isa education;
```
```java
mindmapsGraph.putRelation(education)
    .putRolePlayer(teacher, socrates).putRolePlayer(student, plato);
mindmapsGraph.putRelation(education)
    .putRolePlayer(teacher, plato).putRolePlayer(student, aristotle);
mindmapsGraph.putRelation(education)
    .putRolePlayer(teacher, aristotle).putRolePlayer(student, alexander);
```

> **Aristotle's Teacher**
>
> Try writing a query to see who taught Aristotle. Include all the roles and
> types (`teacher`, `student` and `education`), then remove them one by one to
> see when the results change!

## Resources

Some people have special titles and epithets and we want to talk about that!
So, we'll create some resource types that can be attached to a person:

```sql
insert title isa resource-type, datatype string;
insert epithet isa resource-type, datatype string;
insert person has-resource title, has-resource epithet;
```
```java
RoleType hasTitleOwner = mindmapsGraph.putRoleType("has-title-owner");
RoleType hasTitleValue = mindmapsGraph.putRoleType("has-title-value");
RelationType hasTitle = mindmapsGraph.putRelationType("has-title")
    .hasRole(hasTitleOwner).hasRole(hasTitleValue);

RoleType hasEpithetOwner = mindmapsGraph.putRoleType("has-epithet-owner");
RoleType hasEpithetValue = mindmapsGraph.putRoleType("has-epithet-value");
RelationType hasEpithet = mindmapsGraph.putRelationType("has-epithet")
    .hasRole(hasEpithetOwner).hasRole(hasEpithetValue);

ResourceType<String> title = mindmapsGraph.putResourceType("title", Data.STRING).playsRole(hasTitleValue);
ResourceType<String> epithet = mindmapsGraph.putResourceType("epithet", Data.STRING).playsRole(hasEpithetValue);

person.playsRole(hasTitleOwner).playsRole(hasEpithetValue);
```

Let's make Alexander "Great"!

```sql
insert "Alexander" has epithet "The Great";
```
```java
Resource<String> theGreat = mindmapsGraph.putResource("The Great", epithet);
mindmapsGraph.putRelation(hasEpithet)
    .putRolePlayer(hasEpithetOwner, alexander).putRolePlayer(hasEpithetValue, theGreat);
```

This is a quick way to add a relation between `Alexander` and an `epithet` with
value `"The Great"`.

![](/docs/images/epithet.png)

Let's add the rest of Alexander's titles while we're at it:

```sql
insert "Alexander" has title "Hegemon";
insert "Alexander" has title "King of Macedon";
insert "Alexander" has title "King of Persia";
insert "Alexander" has title "Pharaoh of Egypt";
insert "Alexander" has title "Lord of Asia";
```
```java
Resource<String> hegemon = mindmapsGraph.putResource("Hegemon", title);
Resource<String> kingOfMacedon = mindmapsGraph.putResource("King of Macedon", title);
Resource<String> shahOfPersia = mindmapsGraph.putResource("Shah of Persia", title);
Resource<String> pharaohOfEgypt = mindmapsGraph.putResource("Pharaoh of Egypt", title);
Resource<String> lordOfAsia = mindmapsGraph.putResource("Lord of Asia", title);
mindmapsGraph.putRelation(hasTitle)
    .putRolePlayer(hasTitleOwner, alexander).putRolePlayer(hasTitleValue, hegemon);
mindmapsGraph.putRelation(hasTitle)
    .putRolePlayer(hasTitleOwner, alexander).putRolePlayer(hasTitleValue, kingOfMacedon);
mindmapsGraph.putRelation(hasTitle)
    .putRolePlayer(hasTitleOwner, alexander).putRolePlayer(hasTitleValue, shahOfPersia);
mindmapsGraph.putRelation(hasTitle)
    .putRolePlayer(hasTitleOwner, alexander).putRolePlayer(hasTitleValue, pharaohOfEgypt);
mindmapsGraph.putRelation(hasTitle)
    .putRolePlayer(hasTitleOwner, alexander).putRolePlayer(hasTitleValue, lordOfAsia);
```

Now we can query for people, with their id and titles:

```sql
match $x isa person select $x(id, has title)

$x id "Socrates";
$x id "Plato";
$x id "Aristotle";
$x id "Alexander" has title "Pharaoh of Egypt" has title "Hegeon" has title "Shah of Persia" has title "King of Macedon" has title "Lord of Asia";
```

Wait, who's the Pharaoh again?

```sql
match $pharaoh has title "Pharaoh of Egypt"

$pharaoh id "Alexander" isa person;
```
```java
pharaohOfEgypt.ownerInstances().forEach(instance -> {
    System.out.println("    ->" + instance.getId());
}); //Pssssstttt Graql is much better at querying relationships!!
```

> **Predicates**
>
> When querying for an id, value or resource you can use predicates as well as
> direct values. For example, `has epithet contains "Great"`. See if you can
> write a query for everyone with a title containing "King".

## Relations as Role players

Philosophers know lots of things. We should probably include this in our
ontology.

```sql
insert knowledge isa relation-type;
insert thinker isa role-type;
insert thought isa role-type;
insert knowledge has-role thinker, has-role thought;
insert fact isa entity-type, plays-role thought;
insert person plays-role thinker;
insert description isa resource-type, datatype string;
insert fact has-resource description;
```
```java
RoleType thinker = mindmapsGraph.putRoleType("thinker");
RoleType thought = mindmapsGraph.putRoleType("thought");
RelationType knowledge = mindmapsGraph.putRelationType("knowledge")
    .hasRole(thinker).hasRole(thought);
EntityType fact = mindmapsGraph.putEntitytType("fact").playsRole(thought);
person.playsRole(thinker);
```

Aristotle knew some astronomy, Plato knew a lot about caves and Socrates didn't
really know anything at all.

```sql
insert "sun-fact" isa fact, has description "The Sun is bigger than the Earth";
insert (thinker "Aristotle", thought "sun-fact") isa knowledge;
insert "cave-fact" isa fact, has description "Caves are mostly pretty dark";
insert (thinker "Plato", thought "cave-fact") isa knowledge;
insert "nothing" isa fact;
insert (thinker "Socrates", thought "nothing") isa knowledge;
```
```java
Entity sunFact = mindmapsGraph.putEntity("sun-fact", fact).setValue("The Sun is bigger than the Earth");
Entity caveFact = mindmapsGraph.putEntity("cave-fact", fact).setValue("Caves are mostly pretty dark");
Entity nothing = mindmapsGraph.putEntity("nothing", fact);
mindmapsGraph.putRelation(knowledge)
    .putRolePlayer(thinker, aristotle).putRolePlayer(thought, sunFact);
mindmapsGraph.putRelation(knowledge)
    .putRolePlayer(thinker, plato).putRolePlayer(thought, caveFact);
Relation socratesKnowsNothing = mindmapsGraph.putRelation(knowledge)
    .putRolePlayer(thinker, socrates).putRolePlayer(thought, nothing);
```

A relation is actually just a special kind of instance. Just like how
`Socrates` can be a role player, relations themselves can also be role players.

For example, Socrates knew nothing, but he also *knew* that he knew nothing!

First, we have to state that someone can think about their own knowledge:

```sql
insert knowledge plays-role thought;
```
```java
knowledge.playsRole(thought);
```

We can now give Socrates one final piece of knowledge:

```sql
match $socratesKnowsNothing ("Socrates", "nothing") insert (thinker "Socrates", thought $socratesKnowsNothing) isa knowledge
```
```java
mindmapsGraph.putRelation(knowledge)
    .putRolePlayer(thinker, socrates).putRolePlayer(thought, socratesKnowsNothing);
```

Here, `socratesKnowsNothing` is the relation between `Socrates` and `nothing`.
We then put `socratesKnowsNothing` in a *new* `knowledge` relation as a role
player.

Finally, we'll check out everything Socrates knows:

```sql
match ("Socrates", $x) isa knowledge

$x id "nothing" isa fact;
$x id "knowledge-e387d27c-4f5e-11e6-beb8-9e71128cae77" isa knowledge;
```

> **knowledge-ohmygod-whatisthat-???**
>
> If you don't provide an ID for something such as a relation, it will get an
> automatically generated ID.

![](/docs/images/knowledge.png)

> **When Persisting The Data To Disk**
>
> Once you done make sure to use `mindmaps.sh stop && mindmaps.sh clean` if you
> would like to clean your graph quickly. **Warning :** This will delete all
> your graphs.

