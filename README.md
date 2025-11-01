# pynetics
A genetic soup in Python

The coding plan is going to be in [pynetics_plan.md](pynetics_plan.md) so we don't mix up the documentation with the
coding plan. This also would serve to the purpose of syncing with other people if needed.

## Introduction
I am interested in playing with genetics. Creating some sort of simulation with a small set of parameters that we can
use to simulate some evolution.

I have already done this in Ruby some time ago, but I wasn't satisfied with the results and I am trying again now with
Python.

I only do this for the merely fun of it and to learn more about Python and some sort of "genetics".

### The Goal
The goal is to create a small simulation where we can create some entities with a set of genes. These entitites will
then be able to reproduce, mutate and evolve over time.

### General Description of the Simulation

We will have some set of genes that will define the entity's characteristics. These genes will be represented as a
string floating point numbers between 0 and 1. For example a gene vector could look like this:

```python
[1.0, 0.5, 0.2, 0.8, 0.1, 0.2, ....]
```

Each gene will represent a different characteristic of the entity. For example:

- Gene 0: Stamina
- Gene 1: Energy Efficiency
- Gene 2: Attack Power
- Gene 3: Defense
- Gene 4: Speed X
- Gene 5: Speed Y
- Gene 6: Red
- Gene 7: Green
- Gene 8: Blue
...

The simulation will start with a random population of entities with a random set of genes, also we will add a random set
of environmental factors that will affect the entities survival such as food availability, sectors of the environment
that are more dangerous.

The world will run in a loop for a set number of iterations, after those iterations the world will reset and start
again ONLY with the best entities from the previous run.

In every iteration of the simulation the entity will lose some energy based on its genes and the environmental factors,
so for example an entity with high stamina will lose stamina based on the enrgy efficiency gene, however if the entity
is unable to move and can't find food it will eventually die.

The entity can't eat more food than it's stamina allows it to, so an entity with low stamina will be unable to eat a lot
of food and will eventually die of starvation.

The entities will be able to reproduce if they have enough energy, and when they reproduce they will create a new entity
with a combination of their genes and some random mutation, the entity who gave birth will lose some energy in the
process and the new entity will start with some fixed energy based on their genes.

The Red, Green abd Blue genes will define the color of the entity when we visualize it, and the change on the colors
would ve proportional to the average change in mutations over time, so the more the entities evolve, the more different
the colors will be.

## The Canvas

The canvas is where all the objects are contained. It's shaped like a toroid, therefore if an object goes to the left it
appears on the right (and vice versa), or up and down.

The canvas will have a fixed size at the beginning and will maintain a float list of all the objects in the canvas with
float coordinates (row, col). However not two objects can occupy the same space, so if two objects collide, they will
have to resolve the collision based on their characteristics.

The canvas contains a matrix of all the possible objects and also executes the main loop. On each epoch it will execute
the method `tick(canvas, row, col)` on each object, and the object will decide what to do based on its genes and the
environment, that will return the new desired position. If the position is occupied by another object, then a collision
will occur and the canvas will have to resolve the collision.

### Objects

- **Entities**: All the living creatures that have genes
- **Rocks**: Unavoidable objects in the canvas that can be randomize or have specific shapes in the canvas (that has to
  be defined before the world start).
- **Food**: Objects that can be eaten by the entities to gain energy.
- **Danger Zones**: Areas in the canvas that will make the entities lose energy if they enter them.

### Collisions

#### Evaluating the next zone

When a `tick(canvas, row, col)` is called on an object, the object will decide to move or not to a new position based on
its genes and the environment. For example an entity has a `fight_or_flight` gene will tell the entity how to react when
it encounters a danger zone or another entity, while the `food_seeking` gene will tell the entity how likely it is to
move towards food when it sees it in the environment. These genes will be used to decide the next position of the
entity. The zone types can be:

- Empty
- Rock
- Food
- Danger Zone
- Another Entity

#### Resolving Collisions

If the object decides to move to a position occupied by another object, then a collision will occur, and the canvas will
have to resolve the collision calling some methods on the objects involved in the collision. An entity can move to any
space except rocks.

- **Entity vs Rock**: An entity will **never** be able to move into a rock.
- **Entity vs Food**: If an tentity moves into a food space it will eat the food like `entity.eat(food)` gaining the
  energy in the food object and the food will be removed from the canvas.
- **Entity vs Danger Zone**: If an entity moves into a danger zone it will lose energy like this
  `entity.damage(danger_zone.damage_amount + entity.defense_gene_modifier())`.
- **Entity vs Entity**: If an entity moves into another entity space, then a fight will occur like this:
  `entity1.damage(entity2.attack_power_gene_modifier() - entity1.defense_gene_modifier())` and vice versa for the other
  `entity2.damage(entity1.attack_power_gene_modifier() - entity2.defense_gene_modifier())`, whoever has less energy
  after the fight will die and be removed from the canvas or if both have the same energy or no energy left both will
  die.

### Visualization

Each entity will be represented by a colored circle based on their RGB genes, while rocks will be gray squares, food
will be green squares and danger zones will be red squares. The canvas will be using the library defined (most likely to
be `arcade`).

The canvas size is smaller than the visualization window. For example if the entities are 3x3 pixels and the canvas is
200x200 pixels, then the visualization window will be 600x600 pixels. This has two benefits, on one side the entities
are easier to see, and on the other side the canvas size can be smaller, making the simulation faster.
