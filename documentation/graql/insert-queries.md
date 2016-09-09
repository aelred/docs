# Insert Queries

```sql
insert
$p isa pokemon, id "Pichu", has pokedex-no 172;
(descendant Pikachu, ancestor $p) isa evolution;
```
```java
qb.insert(
    var("p").isa("pokemon").id("Pichu").has("pokedex-no", 172),
    var().isa("evolution")
        .rel("descendant", id("Pikachu"))
        .rel("ancestor", "p")
).execute();",
```

An insert query will insert the specified [variable
patterns](#variable-patterns) into the graph. If a [match
query](match-queries.md) is provided, the query will insert the given variable
patterns for every result of the match query.

A variable pattern is a pattern describing [properties](#properties) to set on
a particular concept. The variable pattern can optionally be bound to a
variable, an ID or a "new-type".

Variable patterns that are not bound, bound to a variable or bound to a
"new-type" are inserted. Variable patterns bound to a type are not
automatically inserted. "new-type" is indicated with the keyword `type`
followed by a string.

## Properties

### isa

```sql
insert "Totodile" isa pokemon;
```
```java
qb.insert(id("Totodile").isa("pokemon"));
```

Set the type of this concept.

### id

```sql
insert "Pikachu" isa pokemon
```
```java
qb.insert(id("Pikachu").isa("pokemon"));
```

Create a concept with the given id, or retrieve it if one with that id exists.
The created or retrieved concept can then be modified with further properties.

### value

```sql
insert $my-pokedex-no isa pokedex-no, value 152;
```
```java
qb.insert(var("my-pokedex-no").isa("pokedex-no").value(152));
```

Set the value of a resource concept.

### has

```sql
insert "Pichu" isa pokemon has height 30;
```
```java
qb.insert(id("Pichu").isa("pokemon").has("height", 30));
```

Add a resource of the given type to the concept.

### relation

```sql
insert (pokemon-with-type Pichu, type-of-pokemon electric) isa has-type;
```
```java
qb.insert(
  var()
    .rel("pokemon-with-type", "Pichu")
    .rel("type-of-pokemon", "electric")
    .isa("has-type")
);",
```

Make the concept a relation that relates the given role players, playing the
given roles.

## Type Properties

The following properties only apply to types.

### ako

```sql
insert gen2-pokemon ako pokemon;
```
```java
qb.insert(id("gen2-pokemon").ako("pokemon"));
```

Set the super type of this concept type.

### has-role

```sql
insert
trained-by isa relation-type, has-role trainer, has-role pokemon-trained;
```
```java
qb.insert(
  id("trained-by").isa("relation-type")
    .hasRole("trainer").hasRole("pokemon-trained")
);
```

Add a role to this relation type.

### plays-role

```sql
insert pokemon plays-role pokemon-trained;
```
```java
qb.insert(id("pokemon").playsRole("pokemon-trained"));
```

Allow the concept type to play the given role.
