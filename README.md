# KE_PetShop_Project

A pet shop ontology built in Protégé for the Knowledge Engineering final project. It matches pet products to dogs and cats by species, life stage, size, breed, lifestyle and health condition, and uses SWRL rules to infer knowledge that is never entered by hand, such as health risks, safe foods and daily calorie needs.

## Scope

The ontology covers **dog and cat products** sold in a retail pet shop: pet food, accessories, grooming products and health-care products. It also models the pet characteristics needed for matching, health conditions, ingredients, brands and shop staff.

Out of scope: other animals, veterinary diagnosis and treatment, live-animal sales, grooming and boarding services, and inventory logistics.

## At a glance

| Item | Count |
|---|---|
| Classes | 53 (5 defined classes) |
| Object properties | 27 (6 inferred only) |
| Data properties | 11 (2 inferred only) |
| SWRL rules | 28 |
| Individuals | 71 (value individuals + sample data) |

## Class hierarchy

```
owl:Thing
├── Product
│   ├── PetFood ── DryFood, WetFood, Treat, PrescriptionDiet
│   │              (defined: GrainFreeFood, SeniorFood, WeightControlFood)
│   ├── Accessory ── Toy, Bed, Bowl, CollarAndLeash
│   ├── GroomingProduct ── Shampoo, Brush
│   ├── HealthCareProduct ── Supplement, FleaAndTickControl
│   └── (defined: CatProduct, DogProduct)
├── Pet ── Cat, Dog
├── Pet_characteristics
│   ├── LifeStage, PetSize, CoatType
│   ├── Breed ── CatBreed, DogBreed
│   └── Lifestyle ── LivingEnvironment, ActivityLevel
├── HealthCondition
│   └── FoodAllergy, DigestiveCondition, WeightCondition, SkinAndCoatCondition,
│       UrinaryCondition, JointCondition, DentalCondition
├── Ingredient ── ProteinSource, Grain, VegetableAndFruit, Additive
├── Brand
└── Person ── StoreManager, Cashier
```

Health conditions and pet characteristics are modeled as **individuals** inside category classes. For example, `hairball` is an individual of `DigestiveCondition`, and `indoor` is an individual of `LivingEnvironment`. This lets SWRL rules refer to them directly.

## Competency questions

**Simple** (answered from asserted facts with SPARQL only)

1. Which products go well with a specific product?
2. Which accessories suit a pet of a specific size?
3. Which products are recommended for a specific health condition?
4. Which products are suitable for both dogs and cats?
5. Which products of a specific brand cost less than a given price?

**Complex** (need SWRL rules to infer new knowledge before the query can answer)

1. 
2. 
3. 
4. 
5. 

The rules, queries and expected results for every question are in [description.md](description.md).

## How to use

1. Open `KEproject_Petshop.ofn` in [Protégé](https://protege.stanford.edu/) 5.6 or later.
2. Select **Reasoner → Pellet**, then **Reasoner → Start reasoner**. HermiT does not run SWRL math built-ins (`swrlb:add`, `swrlb:multiply`, `swrlb:greaterThanOrEqual`), so use Pellet.
3. Check inferred facts on an individual (for example `Mali`) in the **Individuals** tab. Inferred values are highlighted in yellow.
4. Run the competency-question queries in the **Snap SPARQL** tab. The default SPARQL Query tab does not see inferred facts.

## Repository files

| File | Content |
|---|---|
| `KEproject_Petshop.ofn` | The ontology (OWL Functional Syntax) |
| `README.md` | This overview |
| `description.md` | Full description of every class, property, individual, rule and query |

## Team

| Name | Student ID |
|---|---|
| Picha Wiwattanawongsa | 6610545430 |
| Panthut Ketphan | 6610545421 |
| Pannawit Mahacharoensiri | 6610545855 |
| Sorasit Kateratorn | 6610545944 |
| Tat Tanprasert | 6710545636 |
