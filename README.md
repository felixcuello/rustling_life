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
