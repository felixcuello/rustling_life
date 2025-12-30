# Rustling Life
A genetic soup in Rust

The coding plan is going to be in [rustling_life.plan.md](./rustling_life.plan.md) so we don't mix up the documentation with the
coding plan. This also would serve to the purpose of syncing with other people if needed.

---

## Quick Start

### Prerequisites

You need to have conda/miniconda installed on your system:
- **macOS**: `brew install --cask miniconda`
- **Other platforms**: Download from [Miniconda](https://docs.conda.io/en/latest/miniconda.html)

### Environment Setup

TBD

---

## Introduction
I am interested in playing with genetics. Creating some sort of simulation with a small set of parameters that we can
use to simulate some evolution.

I have already done this in Ruby some time ago, but I wasn't satisfied with the results and I am trying again now with
Rust.

I only do this for the merely fun of it and to learn more about Rust and some sort of "genetics".

### The Goal
The goal is to create a small simulation where we can create some entities with a set of genes. These entitites will
then be able to reproduce, mutate and evolve over time.

### General Description of the Simulation

We will have some set of genes that will define the entity's characteristics. These genes will be represented as a
string floating point numbers between 0 and 1. For example a gene vector could look like this:

```rust
[1.0, 0.5, 0.2, 0.8, 0.1, 0.2, ....]
```

Each gene will represent a different characteristic of the entity. For example:

- Gene 0: Stamina
- Gene 1: Energy Efficiency
- Gene 2: Attack Power
- Gene 3: Defense
- Gene 4: Speed X
- Gene 5: Speed Y
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

---

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
- **Entity vs Strange Entity**: If an entity moves into another entity space, then a fight will occur like this:
  `entity1.damage(entity2.attack_power_gene_modifier() - entity1.defense_gene_modifier())` and vice versa for the other
  `entity2.damage(entity1.attack_power_gene_modifier() - entity2.defense_gene_modifier())`, whoever has less energy
  after the fight will die and be removed from the canvas or if both have the same energy or no energy left both will
  die.
- **Entity vs Same Entity**: If an entity moves into another entity of the same type, then they will try to reproduce
  like this: `child = entity1.reproduce_with(entity2)`, if they have enough energy to reproduce, then a new entity will
  be created with a combination of the genes of both entities and some random mutation, both entities will lose some
  energy in the process, the new entity will be placed in a random empty space in the canvas.

### Visualization

Each entity will be represented by a colored circle based on their genes, while rocks will be gray squares, food will be
green squares and danger zones will be red squares. The canvas will be using the library defined (most likely to be
`arcade`). The entity must have a method `get_color()` that will return the color of the entity based on its genes (only
the entities know how to color themselves).

The canvas size is smaller than the visualization window. For example if the entities are 3x3 pixels and the canvas is
200x200 pixels, then the visualization window will be 600x600 pixels. This has two benefits, on one side the entities
are easier to see, and on the other side the canvas size can be smaller, making the simulation faster.

## Living Entities

- **Entities**: All the living creatures that have genes
- **Rocks**: Unavoidable objects in the canvas that can be randomize or have specific shapes in the canvas (that has to
  be defined before the world start).
- **Food**: Objects that can be eaten by the entities to gain energy.
- **Danger Zones**: Areas in the canvas that will make the entities lose energy if they enter them.

### Implementation
All entities have to respond to the same methods, the difference is in the implementation of the methods. That makes the
loop and collision resolution easier to implement.

- `tick(canvas, row, col)`: This method is called on each epoch of the simulation for each object in the canvas. The
  object will decide what to do based on its genes and the environment, and will return the new desired position as a
  tuple `(new_row, new_col)`. The desired position can be the same as the current position if the object decides to stay
  in place, and the canvas will handle the collision if an object wants to move to a position occupied by another
  object.
- `damage(amount)`: This method is called when an entity takes damage. The entity will lose energy based on the amount
  of damage taken.
- `eat(food)`: This method is called when an entity eats food. The entity will gain energy based on the energy in the
  food object.
- `reproduce_with(other_entity)`: This method is called when two entities of the same type collide and decide to
  reproduce. The method will return a new entity with a combination of the genes of both entities and some random
  mutation.
- `type()`: This method will return the type of the entity as a string. This is used to determine if two entities are of
  the same type or not during collisions.
- `is_alive()`: This method will return a boolean indicating if the entity is alive or not. If the entity's energy is
    less than or equal to zero, then the entity is considered dead.
- `get_color()`: This method will return the color of the entity based on its genes. The color will be used for visualization
  purposes.

### Attributes

The entities will have genes and attributes. the attributes are things that are randomize at birth and will change over
time based on the situations the entity encounters. Perhaps some of the attributes will be influenced by the genes, but
not all of them. The attribute list:

- `energy`: The energy of the entity. This will be a floating point number that will decrease over time, depending on
  every `tick()`. If the energy reaches zero, the entity will die.
- `age`: The age of the entity. This will be an integer that will increase on every `tick()`. The age can be used to
  determine things like reproduction age or death by old age.
- The position is determined by the canvas, so the entity doesn't need to store its own position.
- `horizontal_speed`: The horizontal speed of the entity. This will be a floating point number that will determine how
  fast the entity can move horizontally. This can be influenced by the `speed_horizontal` gene.
- `vertical_speed`: The vertical speed of the entity. This will be a floating point number that will determine how fast
  the entity can move vertically. This can be influenced by the `speed_vertical` gene. For example, there are entities
  that can move faster in one direction than the other.
- `flying`: An integer value that tells if the entity is flying or not. It tells us how many ticks the entity can fly before
  stopping. If an entity is flying, it wants to continue moving in the same direction.
- `fighting`: A boolean value that tells if the entity is currently fighting another entity. If an entity is fighting, it
  wants to continue fighting until one of them dies.

### Genes

Each live entity will have a set of genes that will define its characteristics. The genes will be represented as a list
of floating point numbers between 0 and 1. The genes will be used to determine the entity's behavior and
characteristics. The list of genes is stored in a list called `genes`. Each gene will have a specific purpose that will
change the behavior of the entity.

The genes are the likelihood of a characteristic being present in the entity. For example, a gene with a value of 0.8
for `food_seeking` means that the entity has an 80% chance of moving towards food when it sees it in the environment,
but it also has a 20% chance of ignoring the food and doing something else.

- `stamina`: This gene will determine how much energy the entity loses on each tick. An entity with high stamina will
  lose less energy on each tick, while an entity with low stamina will lose more energy. This gene increase the BLUE
  color.
- `aggressiveness`: This gene will determine how likely the entity is to attack another entity when it encounters it.
  An entity with high aggressiveness will be more likely to attack, while an entity with low aggressiveness will be more
  likely to flee. This gene increase the RED color. A value of `0.5` means the entity is neutral and will decide based
  on other factors. A value of `1.0` means the entity will always attack, while a value of `0.0` means the entity will
  always flee.
- `food_seeking`: This gene will determine how likely the entity is to move towards food when it sees it in the
  environment. An entity with high food seeking will be more likely to move towards food, while an entity with low food
  seeking will be more likely to ignore the food. This gene increase the GREEN color.
- `reproduction_rate`: This gene will determine how likely the entity is to reproduce when it encounters another entity
    of the same type. An entity with high reproduction rate will be more likely to reproduce, while an entity with low
    reproduction rate will be less likely to reproduce. This increase GREEN and RED colors.
- `speed_horizontal`: This gene will determine how fast the entity can move horizontally. An entity with high
  speed_horizontal will be able to move faster horizontally, while an entity with low speed_horizontal will be slower.
- `speed_vertical`: This gene will determine how fast the entity can move vertically. An entity with high speed_vertical
  will be able to move faster vertically, while an entity with low speed_vertical will be slower.
- `defense`: This gene will determine how much damage the entity can resist when it is attacked. An entity with high
  defense will take less damage, while an entity with low defense will take more damage. This increase BLUE and RED
  colors.
- `attack_power`: This gene will determine how much damage the entity can inflict when it attacks another entity. An
  entity with high attack_power will inflict more damage, while an entity with low attack_power will inflict less
  damage. This increase RED color.
- `mutation_rate`: This gene will determine how likely the entity is to mutate when it reproduces. An entity with high
  mutation_rate will be more likely to mutate, while an entity with low mutation_rate will be less likely to mutate.
  This increase BLUE and GREEN colors.

### Mutation

When two entities reproduce, the child entity will inherit the genes of both parents like this:

```
mutation_sum = parent1.mutation_rate + parent2.mutation_rate
child_genes = []
for i in range(len(parent1.genes)):
    if(i == mutation_rate_gene_index):
        continue
    child_gene[i] = (parent1.genes[i] * parent1.mutation_rate + parent2.genes[i] * parent2.mutation_rate) / mutation_sum
```


