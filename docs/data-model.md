# Data model

This document describes what an OpenFODData record contains and why. It is the reference for the submission form, the review process and the API.

## Principle: values, not what they mean

OpenFODData records facts: measured FODMAP amounts, where each amount comes from, and what a normal portion weighs. It does not say whether a food is low, moderate or high, and it contains no thresholds or traffic lights.

Interpreting the values is up to the developers who use the data. Thresholds are averages, people react differently, and apps have different needs. Keeping interpretation out of the data keeps the facts neutral and the project small. A traffic-light service could be built later as a separate layer on top, without changing the database.

## Scope

OpenFODData covers single ingredients. A product belongs in the database if it is made from **one main raw material**. Processing is fine, a recipe is not. Salt, water and starter cultures don't count as extra ingredients.

| In | Out |
|---|---|
| Apple, carrot, milk, lentils | Ketchup, stock, pesto |
| Cheddar, yoghurt, tofu | Ready meals, spice mixes |
| Wheat flour, sourdough bread, plain pasta | Branded or specialty breads |
| Raisins, apple juice, coffee, wine | Soy sauce (soy and wheat) |

Combinations are left to the apps using the API: they can combine ingredient values into recipes or products themselves. Packaged branded products are already covered by [Open Food Facts](https://world.openfoodfacts.org/).

## Record structure

Every ingredient has these fields.

### Name

The name people use when they buy or ask for it. If something is bought by its own name, it gets its own record. Red cabbage, white cabbage and savoy cabbage are three ingredients, not one ingredient with three sub-types. OpenFODData is a list of ingredients, not a classification of species.

Apples are the default exception: there is one record, Apple, until someone brings a source showing a specific variety has different values.

Search takes care of finding related items. A search for "cabbage" should return every cabbage, without the database needing a parent "cabbage" record.

### Category

One food group per ingredient, with no subcategories.

1. Fruit
2. Vegetables (including mushrooms and seaweed)
3. Legumes and pulses
4. Grains and starches
5. Nuts and seeds
6. Dairy and alternatives (including plant milks)
7. Meat, fish and eggs
8. Fats and oils
9. Herbs and spices
10. Sweeteners
11. Drinks

Legumes have their own category because they're one of the most looked-up groups. Dairy alternatives sit with dairy because people search by how they use a food, not by what it's made from.

### Type

Either **whole** or **processed**.

- **Whole:** as harvested or produced, or only washed, cut, chilled or frozen. Examples: apple, milk, eggs, dried lentils as sold.
- **Processed:** made from one main raw material, with processing that changes what's in it: drying, canning, fermenting, milling, pressing, baking. Examples: prunes, cheddar, tofu, flour, canned chickpeas.

Drying and canning count as processing because they change the values. Dried fruit concentrates its sugars, and canned legumes lose some FODMAPs into the liquid.

### Base

Only for processed ingredients: the whole ingredient it's made from. Prunes have plum as their base, cheddar has cow's milk, tofu has soybeans. This lets the site and apps show related ingredients side by side.

### Variants

Only for the same item in a different state, which you can't buy separately:

- **Ripeness:** unripe or ripe banana
- **Part of the plant:** green tops or white bulb of spring onion

Each variant has its own set of FODMAP values.

### FODMAP values

Six subgroups, each stored as an amount in **grams per 100 grams** of the ingredient (or variant).

| Subgroup | FODMAP group |
|---|---|
| Fructans | Oligosaccharides |
| GOS | Oligosaccharides |
| Lactose | Disaccharides |
| Fructose and glucose | Monosaccharides |
| Sorbitol | Polyols |
| Mannitol | Polyols |

Per 100 grams is a storage base, not a serving size. Nobody eats 100 grams of garlic, but a fixed base keeps ingredients comparable and lets developers calculate any portion. Serving sizes are stored separately.

Fructose and glucose are stored as two separate amounts. Fructose only matters when there's more of it than glucose, and that excess can be calculated from the two.

Rules for every value:

- **Only measured amounts.** Sources that only say "high in fructose" without numbers are not accepted. They can be a lead to find the underlying study.
- **Unknown stays unknown.** If a subgroup wasn't measured, the value is unknown, never 0. A 0 would tell people something is safe when nobody checked.
- **Every value has its own source.** Fructans can come from one study and sorbitol from another.
- **Every value has a status** (see below).

### Status

Each value carries one of these, so developers can decide what they accept:

- **Verified:** backed by a published lab measurement and approved by two reviewers
- **Sourced:** has a citation, checked by one reviewer
- **Community:** submitted without a published source

Processed ingredients vary by method (how long bread ferments, how long cheese ages), so their submissions also need a short description of the method, to check against the source.

### Typical servings

The weight of normal portions, for example 1 clove of garlic = 3 grams. How servings are described and who decides what's typical is still open.

## Example

**Prunes**

- Category: Fruit
- Type: Processed
- Base: Plum
- Variants: none
- FODMAP values: six subgroups in grams per 100 g, each with a source and status, or unknown
- Typical servings: to be decided

## Open questions

- **Typical servings:** how portions are described (clove, slice, handful), who decides what's typical, and how to handle different national portion sizes.
- **Status levels:** whether "community" values without a source belong in the database at all, now that only measured amounts are accepted.
- **Licensing:** proposed ODbL for the data and MIT for the code, not decided yet.
