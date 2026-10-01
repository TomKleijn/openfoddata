# Data model

This document describes what an OpenFODData record contains and why. It is the reference for the submission form, the review process and the API.

## Principle: values, not what they mean

OpenFODData records facts: measured FODMAP amounts and where each amount comes from. It does not say whether a food is low, moderate or high, and it contains no thresholds or traffic lights.

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

Per 100 grams is a storage base, not a serving size. Nobody eats 100 grams of garlic, but a fixed base keeps ingredients comparable and lets developers calculate any portion.

Fructose and glucose are stored as two separate amounts. Fructose only matters when there's more of it than glucose, and that excess can be calculated from the two.

Rules for every value:

- **Only measured amounts.** Sources that only say "high in fructose" without numbers are not accepted. They can be a lead to find the underlying study.
- **Unknown stays unknown.** If a subgroup wasn't measured, the value is unknown, never 0. A 0 would tell people something is safe when nobody checked.
- **Every value has its own source.** Fructans can come from one study and sorbitol from another.
- **Every value has a status** (see below).

### Status

Each value has one of two statuses:

- **Submitted:** added by a contributor with a measured amount and a source, not yet reviewed
- **Verified:** checked by a reviewer against its source

Both are published. A new submission appears in the data straight away as submitted and moves to verified after review. Developers decide what they accept: a cautious app uses only verified values, others can use both. This keeps contributors' work visible while the review queue is long, and the status keeps the data honest.

Every submission needs a measured amount and a source. Processed ingredients vary by method (how long bread ferments, how long cheese ages), so their submissions also need a short description of the method.

### Review

To verify a value, the reviewer checks three things:

1. **The source is a real measurement:** a published study or lab report that exists and can be found.
2. **The number matches the source,** including the unit. Mixing up grams per 100 grams with grams per serving is the most likely mistake.
3. **The ingredient matches the source:** the same variant and, for processed ingredients, the same method.

Each verified value records who verified it and when. One reviewer is enough to start with. Once there are more volunteers, verification will require a second reviewer.

## Not included: serving sizes

OpenFODData doesn't store typical servings. Deciding what a normal portion is (half an apple or a whole one, a Dutch or an American portion) is a judgement, and judgements are left to developers.

Developers who need portion weights, such as how much a clove of garlic or a medium apple weighs, can get them from [USDA FoodData Central](https://fdc.nal.usda.gov/), a free and open database that lists household portion weights for thousands of foods.

## Example

**Prunes**

- Category: Fruit
- Type: Processed
- Base: Plum
- Variants: none
- FODMAP values: six subgroups in grams per 100 g, each with a source and status, or unknown

## File format

Each ingredient is one JSON file in `data/ingredients/`, named after its id. See [apple.json](../data/ingredients/apple.json) for a working example.

- All amounts are in grams per 100 grams, so the unit isn't repeated in every value.
- A subgroup that wasn't measured is `null`. A measured zero is `0.0` with a source.
- Each value refers to an entry in the record's `sources` list, so a source used for several values is written down once.
- An optional `note` explains anything a reviewer or developer should know, such as a source that measured only the flesh.
- An ingredient with variants keeps its values inside each variant instead of at the top level.

## Licensing

The data is published under the [Open Database License (ODbL) 1.0](https://opendatacommons.org/licenses/odbl/1-0/), see [LICENSE-DATA](../LICENSE-DATA). Anyone can use it for anything, including paid products, as long as they credit OpenFODData. Anyone who publishes an improved version of the database itself has to release it under ODbL too, so improvements flow back to everyone. Apps, websites and charts built with the data can stay closed; they only need the credit.

The code (website, API and submission form) is published under the MIT license, see [LICENSE](../LICENSE).

Contributors agree that their submissions are published under ODbL. The submission form asks for this with a checkbox: "I agree my submission is published under the Open Database License."
