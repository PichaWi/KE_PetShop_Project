# Pet Shop Ontology – Full Description

This document describes every part of `KEproject_Petshop.ofn`: classes, individuals, properties, SWRL rules, sample data and the competency-question queries with their expected results.

Namespace: `http://www.example.org/petshop#` (prefix `:`)

## Contents

1. [Purpose and scope](#1-purpose-and-scope)
2. [Design decisions](#2-design-decisions)
3. [Classes](#3-classes)
4. [Value individuals](#4-value-individuals)
5. [Object properties](#5-object-properties)
6. [Data properties](#6-data-properties)
7. [SWRL rules](#7-swrl-rules)
8. [Sample data](#8-sample-data)
9. [Competency questions and queries](#9-competency-questions-and-queries)
10. [Knowledge sources](#10-knowledge-sources)

---

## 1. Purpose and scope

The ontology models the products sold in a pet shop and the pets they are meant for. A system built on it can match a product to a pet by species, life stage, size, breed, lifestyle, health condition and ingredients. Customers and staff can then find suitable, safe products quickly.

**Supported applications**

- Product recommendation for a pet shop website or in-store kiosk
- Allergy filter that excludes foods containing an ingredient a pet reacts to
- Feeding guide that calculates a pet's daily calorie need and portion size
- Staff support tool for answering customer questions consistently

**Scope:** dog and cat products only (pet food, accessories, grooming and health-care products), with the pet characteristics, health conditions, ingredients, brands and staff needed to describe them.

**Out of scope:** other animals, veterinary diagnosis and treatment, live-animal sales, grooming and boarding services, and inventory logistics.

---

## 2. Design decisions

**Categories are classes; specific values are individuals.** A health condition such as hairball is one specific thing, so `hairball` is an individual of the class `DigestiveCondition`. The same applies to pet characteristics: `indoor` is an individual of `LivingEnvironment`, `senior` is an individual of `LifeStage`. SWRL rules can only refer to named individuals in their heads (for example `-> hasRiskOf(?p, hairball)`), so this design is required for the rules to work.

**Breed carries domain knowledge.** Each breed individual states its coat type, its size (for dog breeds) and the conditions it is predisposed to. A user only enters a pet's breed, and the reasoner derives the rest.

**Inferred-only properties.** Six object properties and two data properties are never asserted by hand. They are filled only by SWRL rules: `hasRiskOf`, `isSuitableForPet`, `matchesLifeStageOf`, `matchesLifestyleOf`, `recommendedForPet`, `isUnsafeFor`, `hasRER` and `hasDailyCalorieNeed`.

**Simple vs. complex questions.** A simple competency question is answered from asserted facts with SPARQL alone. A complex question needs SWRL rules to infer new facts first; with the reasoner off, its query returns nothing.

---

## 3. Classes

### 3.1 Top level

All seven top-level classes are mutually disjoint.

| Class | Description |
|---|---|
| Product | Any item sold in the pet shop |
| Pet | An animal a product is designed for |
| Pet_characteristics | Attributes of a pet used to match products: life stage, size, coat, breed and lifestyle |
| HealthCondition | A health need that affects which products suit a pet |
| Ingredient | A component of a pet food product |
| Brand | The manufacturer or brand of a product |
| Person | A member of the pet shop staff |

### 3.2 Product

| Class | Superclass | Description |
|---|---|---|
| PetFood | Product | Food eaten by a dog or cat |
| DryFood | PetFood | Dry kibble with low moisture, sold in bags |
| WetFood | PetFood | Canned or pouch food with high moisture |
| Treat | PetFood | Snacks for rewards, training or dental care; not a complete meal |
| PrescriptionDiet | PetFood | Food formulated for a specific health condition, such as kidney care |
| Accessory | Product | Non-food items for daily care |
| Toy | Accessory | Balls, chew toys, rope toys, cat wands |
| Bed | Accessory | Beds, cushions and mats |
| Bowl | Accessory | Food and water bowls and feeders |
| CollarAndLeash | Accessory | Collars, harnesses and leashes |
| GroomingProduct | Product | Products that clean and maintain coat, skin and nails |
| Shampoo | GroomingProduct | Pet shampoos and conditioners |
| Brush | GroomingProduct | Brushes, combs and de-shedding tools |
| HealthCareProduct | Product | Non-prescription products that support health |
| Supplement | HealthCareProduct | Vitamins, joint supplements and probiotics |
| FleaAndTickControl | HealthCareProduct | Spot-on treatments, sprays and collars against parasites |

### 3.3 Pet

| Class | Superclass | Description |
|---|---|---|
| Cat | Pet | Domestic cat. Disjoint with Dog |
| Dog | Pet | Domestic dog. Disjoint with Cat |

### 3.4 Pet_characteristics

`LifeStage`, `PetSize`, `CoatType`, `Breed` and `Lifestyle` are mutually disjoint. The three `Lifestyle` subclasses are also mutually disjoint.

| Class | Superclass | Description |
|---|---|---|
| LifeStage | Pet_characteristics | Age stage that sets nutritional needs |
| PetSize | Pet_characteristics | Body size group, mainly for dogs |
| CoatType | Pet_characteristics | Hair length; long hair raises hairball risk in cats |
| Breed | Pet_characteristics | Breed of the pet, carrying coat, size and predisposition knowledge |
| CatBreed | Breed | Cat breeds. Disjoint with DogBreed |
| DogBreed | Breed | Dog breeds. Disjoint with CatBreed |
| Lifestyle | Pet_characteristics | How the pet lives day to day |
| LivingEnvironment | Lifestyle | Indoor or outdoor |
| ActivityLevel | Lifestyle | How active the pet is |
| NeuterStatus | Lifestyle | Neutered or intact; affects calorie need |

### 3.5 HealthCondition

All seven subclasses are mutually disjoint. The conditions themselves are individuals (Section 4.2).

| Class | Description |
|---|---|
| FoodAllergy | Allergic reaction to a specific ingredient |
| DigestiveCondition | Conditions of the digestive tract |
| WeightCondition | Body weight problems |
| SkinAndCoatCondition | Skin and coat problems |
| UrinaryCondition | Kidney and urinary tract problems |
| JointCondition | Joint and bone problems |
| DentalCondition | Teeth and gum problems |

### 3.6 Ingredient

| Class | Description |
|---|---|
| ProteinSource | Main protein such as chicken, beef, salmon, lamb |
| Grain | Cereals such as wheat, rice, corn |
| VegetableAndFruit | Plant ingredients such as pumpkin, carrot |
| Additive | Vitamins, minerals and supplements such as taurine |

### 3.7 Person

| Class | Description |
|---|---|
| Staff | Person who is working in pet shop |
| StoreManager | Manages the shop, product range and staff |
| Cashier | Handles payment and sales at the counter |
| Customer | Person who come to use services |

### 3.8 Defined classes

These have necessary and sufficient conditions, so the reasoner classifies products into them automatically.

| Class | Definition | Inferred members (sample data) |
|---|---|---|
| CatProduct | `Product and (isSuitableFor some Cat)` | All products suitable for cats |
| DogProduct | `Product and (isSuitableFor some Dog)` | All products suitable for dogs |
| GrainFreeFood | `PetFood and (isGrainFree value true)` | AdultDogLambGrainFree, KidneyCareCatWet, LightDogChickenWet |
| SeniorFood | `PetFood and (targetsLifeStage value senior)` | IndoorCatChickenLight, KidneyCareCatWet, SeniorCatSalmonDry, SeniorSmallDogChicken |
| WeightControlFood | `PetFood and (isRecommendedFor some WeightCondition)` | IndoorCatChickenLight, LightDogChickenWet |

---

## 4. Value individuals

### 4.1 Pet characteristics

Each group is declared `DifferentIndividuals`, so two values in the same group can never be the same.

| Class | Individuals |
|---|---|
| LifeStage | juvenile, adult, senior |
| PetSize | small, medium, large |
| CoatType | shortHair, longHair |
| LivingEnvironment | indoor, outdoor, mixedEnvironment |
| ActivityLevel | lowActivity, moderateActivity, highActivity |

### 4.2 Health conditions

| Class | Individual | Triggered by (allergies only) |
|---|---|---|
| FoodAllergy | chickenAllergy | chicken |
| FoodAllergy | beefAllergy | beef |
| FoodAllergy | fishAllergy | salmon |
| FoodAllergy | wheatAllergy | wheat |
| DigestiveCondition | sensitiveStomach, hairball | |
| WeightCondition | obesity, underweight | |
| SkinAndCoatCondition | skinAllergy, drySkin | |
| UrinaryCondition | kidneyDisease, urinaryStones | |
| JointCondition | arthritis, hipDysplasia | |
| DentalCondition | dentalPlaque | |

### 4.3 Ingredients

| Class | Individuals |
|---|---|
| ProteinSource | chicken, beef, salmon, lamb |
| Grain | wheat, rice, corn |
| VegetableAndFruit | pumpkin, carrot |
| Additive | taurine |

### 4.4 Breeds

| Breed | Class | breedCoatType | breedSize | isPredisposedTo |
|---|---|---|---|---|
| persian | CatBreed | longHair | | kidneyDisease, hairball |
| maineCoon | CatBreed | longHair | | hipDysplasia |
| siamese | CatBreed | shortHair | | |
| britishShorthair | CatBreed | shortHair | | obesity |
| labrador | DogBreed | shortHair | large | obesity, hipDysplasia |
| goldenRetriever | DogBreed | longHair | large | hipDysplasia, skinAllergy |
| shihTzu | DogBreed | longHair | small | dentalPlaque |
| pomeranian | DogBreed | longHair | small | dentalPlaque |
| beagle | DogBreed | shortHair | medium | obesity |

### 4.5 Brands and species references

| Individual | Class | Purpose |
|---|---|---|
| BrandA, BrandB | Brand | Sample brands |
| cat_generic | Cat | Species reference used by `isSuitableFor` on products |
| dog_generic | Dog | Species reference used by `isSuitableFor` on products |

### 4.6 Person names

| Class | Individual |
|---|---|
| Customer | Nida, Somchai |

---

## 5. Object properties

### 5.1 Product properties

| Property | Domain | Range | Characteristics | Description |
|---|---|---|---|---|
| isSuitableForPet | Product | Pet | | Species the product is designed for |
| hasBrand | Product | Brand | Functional | Brand that makes the product |
| goesWellWith | Product | Product | Symmetric | Products sold or used together |
| isRecommendedFor | Product | HealthCondition | | Condition the product helps manage |
| hasIngredient | PetFood | Ingredient | | Ingredient in a food |
| hasMainProtein | PetFood | ProteinSource | Sub-property of hasIngredient | Main protein of a food |
| targetsCharacteristic | Product | Pet_characteristics | Super-property | Characteristic a product is designed for |
| targetsLifeStage | Product | LifeStage | Sub-property of targetsCharacteristic | Life stage a product is made for |
| isSuitableForSize | Product | PetSize | Sub-property of targetsCharacteristic | Pet size a product fits |
| targetsLifestyle | Product | Lifestyle | Sub-property of targetsCharacteristic | Lifestyle a product is made for, such as indoor or neutered |

### 5.2 Pet properties

| Property | Domain | Range | Characteristics | Description |
|---|---|---|---|---|
| hasCharacteristic | Pet | Pet_characteristics | Super-property | A characteristic of a pet |
| hasLifeStage | Pet | LifeStage | Functional, sub-property of hasCharacteristic | Current life stage (inferred from age) |
| hasSize | Pet | PetSize | Functional, sub-property of hasCharacteristic | Size group (inferred from breed for dogs) |
| hasCoatType | Pet | CoatType | Functional, sub-property of hasCharacteristic | Coat length (inferred from breed) |
| hasBreed | Pet | Breed | Functional, sub-property of hasCharacteristic | Breed of the pet |
| hasLivingEnvironment | Pet | LivingEnvironment | Functional, sub-property of hasCharacteristic | Indoor or outdoor |
| hasActivityLevel | Pet | ActivityLevel | Functional, sub-property of hasCharacteristic | Activity level |
| hasCondition | Pet | HealthCondition | | A diagnosed condition, including food allergies |

### 5.3 Knowledge properties

| Property | Domain | Range | Description |
|---|---|---|---|
| breedCoatType | Breed | CoatType | Typical coat of the breed |
| breedSize | Breed | PetSize | Typical size of the dog breed |
| isPredisposedTo | Breed | HealthCondition | Conditions the breed is prone to |
| triggeredBy | FoodAllergy | Ingredient | Ingredient that causes the allergy |

### 5.4 Inferred-only properties

These are filled by SWRL rules and must never be asserted by hand.

| Property | Domain | Range | Filled by | Meaning |
|---|---|---|---|---|
| hasRiskOf | Pet | HealthCondition | R3 | Pet is at risk of the condition |
| matchesLifeStageOf | PetFood | Pet | R5 | Food targets the pet's current life stage |
| matchesLifestyleOf | Product | Pet | R5 | Product targets the pet's environment, activity or neuter status |
| recommendedForPet | Product | Pet | R6 | Product addresses a condition or risk of the pet |
| isUnsafeFor | PetFood | Pet | R7 | Food contains an ingredient the pet is allergic to |

---

## 6. Data properties

| Property | Domain | Range | Characteristics | Description |
|---|---|---|---|---|
| hasName | Person or Pet or Product | xsd:string | | Display name |
| hasAge | Person or Pet | xsd:integer | Functional | Age in years |
| hasPrice | Product | xsd:decimal | Functional | Price in Thai Baht |
| hasWeight | Product | xsd:decimal | | Net package weight in kg |
| hasCaloriePerKg | PetFood | xsd:decimal | | Energy density in kcal per kg |
| hasExpiryDate | PetFood | xsd:date | | Expiry date of the lot |
| isGrainFree | PetFood | xsd:boolean | | True if the food contains no grain |
| hasBodyWeight | Pet | xsd:decimal | Functional | Pet's body weight in kg |
| hasRER | Pet | xsd:decimal | Inferred only | Resting energy requirement in kcal per day |
| hasDailyCalorieNeed | Pet | xsd:decimal | Inferred only | Daily energy requirement in kcal per day |
| hasNeuterStatus | Pet | xsd:boolean | | True if pet is neutered |

---

## 7. SWRL rules

The ontology contains 28 rules in eight groups. Run them with **Pellet**; HermiT does not support the `swrlb:` built-ins.

### R1. Breed to coat type and size (2 rules)

The pet inherits coat type and size from its breed.

```
hasBreed(?p, ?b) ^ breedCoatType(?b, ?c) -> hasCoatType(?p, ?c)
hasBreed(?p, ?b) ^ breedSize(?b, ?s) -> hasSize(?p, ?s)
```

### R2. Life stage from age, species and size (9 rules)

Larger dogs age faster, so they become senior earlier. The cut-off ages are approximate.

| Pet | Juvenile | Adult | Senior |
|---|---|---|---|
| Cat | under 1 | 1 to 6 | 7 and above |
| Small dog | under 1 | 1 to 7 | 8 and above |
| Medium dog | under 1 | 1 to 6 | 7 and above |
| Large dog | under 1 | 1 to 5 | 6 and above |

```
Pet(?p) ^ hasAge(?p, ?a) ^ swrlb:lessThan(?a, 1) -> hasLifeStage(?p, juvenile)
Cat(?p) ^ hasAge(?p, ?a) ^ swrlb:greaterThanOrEqual(?a, 1) ^ swrlb:lessThan(?a, 7) -> hasLifeStage(?p, adult)
Cat(?p) ^ hasAge(?p, ?a) ^ swrlb:greaterThanOrEqual(?a, 7) -> hasLifeStage(?p, senior)
Dog(?p) ^ hasSize(?p, small) ^ hasAge(?p, ?a) ^ swrlb:greaterThanOrEqual(?a, 1) ^ swrlb:lessThan(?a, 8) -> hasLifeStage(?p, adult)
Dog(?p) ^ hasSize(?p, small) ^ hasAge(?p, ?a) ^ swrlb:greaterThanOrEqual(?a, 8) -> hasLifeStage(?p, senior)
Dog(?p) ^ hasSize(?p, medium) ^ hasAge(?p, ?a) ^ swrlb:greaterThanOrEqual(?a, 1) ^ swrlb:lessThan(?a, 7) -> hasLifeStage(?p, adult)
Dog(?p) ^ hasSize(?p, medium) ^ hasAge(?p, ?a) ^ swrlb:greaterThanOrEqual(?a, 7) -> hasLifeStage(?p, senior)
Dog(?p) ^ hasSize(?p, large) ^ hasAge(?p, ?a) ^ swrlb:greaterThanOrEqual(?a, 1) ^ swrlb:lessThan(?a, 6) -> hasLifeStage(?p, adult)
Dog(?p) ^ hasSize(?p, large) ^ hasAge(?p, ?a) ^ swrlb:greaterThanOrEqual(?a, 6) -> hasLifeStage(?p, senior)
```

A dog's life stage depends on its size, and its size depends on its breed (R1). So these rules show chained inference.

### R3. Health risks (3 rules)

```
hasBreed(?p, ?b) ^ isPredisposedTo(?b, ?c) -> hasRiskOf(?p, ?c)
Pet(?p) ^ hasLivingEnvironment(?p, indoor) ^ hasNeuterStatus(?p, neutered) ^ hasActivityLevel(?p, lowActivity) -> hasRiskOf(?p, obesity)
Cat(?p) ^ hasCoatType(?p, longHair) ^ hasLivingEnvironment(?p, indoor) -> hasRiskOf(?p, hairball)
```

1. A pet inherits the risks of its breed.
2. An indoor, neutered, low-activity pet is at risk of obesity.
3. A long-haired indoor cat is at risk of hairballs. Coat type comes from R1.

### R4. Species match (2 rules)

```
Cat(?p) ^ isSuitableFor(?x, ?s) ^ Cat(?s) -> isSuitableForPet(?x, ?p)
Dog(?p) ^ isSuitableFor(?x, ?s) ^ Dog(?s) -> isSuitableForPet(?x, ?p)
```

### R5. Life stage and lifestyle match (4 rules)

```
PetFood(?f) ^ isSuitableForPet(?f, ?p) ^ hasLifeStage(?p, ?l) ^ targetsLifeStage(?f, ?l) -> matchesLifeStageOf(?f, ?p)
isSuitableForPet(?x, ?p) ^ hasLivingEnvironment(?p, ?e) ^ targetsLifestyle(?x, ?e) -> matchesLifestyleOf(?x, ?p)
isSuitableForPet(?x, ?p) ^ hasNeuterStatus(?p, ?n) ^ targetsLifestyle(?x, ?n) -> matchesLifestyleOf(?x, ?p)
isSuitableForPet(?x, ?p) ^ hasActivityLevel(?p, ?v) ^ targetsLifestyle(?x, ?v) -> matchesLifestyleOf(?x, ?p)
```

### R6. Recommendation (2 rules)

A product is recommended for a pet when it is the right species and addresses a condition the pet has or is at risk of.

```
isSuitableForPet(?x, ?p) ^ hasCondition(?p, ?c) ^ isRecommendedFor(?x, ?c) -> recommendedForPet(?x, ?p)
isSuitableForPet(?x, ?p) ^ hasRiskOf(?p, ?c) ^ isRecommendedFor(?x, ?c) -> recommendedForPet(?x, ?p)
```

### R7. Allergy safety (1 rule)

```
isSuitableForPet(?f, ?p) ^ hasCondition(?p, ?a) ^ FoodAllergy(?a) ^ triggeredBy(?a, ?i) ^ hasIngredient(?f, ?i) -> isUnsafeFor(?f, ?p)
```

`hasMainProtein` is a sub-property of `hasIngredient`, so the reasoner also catches a food whose *main protein* triggers the allergy.

SWRL cannot express negation ("does not contain"). The rule therefore infers which foods are *unsafe*, and the query removes them with SPARQL `FILTER NOT EXISTS`.

### R8. Daily calorie need (5 rules)

Resting energy requirement (RER) uses the linear formula RER = 30 × body weight (kg) + 70, valid for pets of about 2 to 45 kg. It is used instead of 70 × BW^0.75 because Pellet does not support the `swrlb:pow` built-in. The daily need is RER times a factor. The factors are approximate values for healthy adult pets.

| Pet | Factor |
|---|---|
| Neutered dog | 1.6 |
| Intact dog | 1.8 |
| Neutered cat | 1.2 |
| Intact cat | 1.4 |

```
Pet(?p) ^ hasBodyWeight(?p, ?w) ^ swrlb:multiply(?x, ?w, 30) ^ swrlb:add(?r, ?x, 70) -> hasRER(?p, ?r)
Dog(?p) ^ hasNeuterStatus(?p, neutered) ^ hasRER(?p, ?r) ^ swrlb:multiply(?m, ?r, 1.6) -> hasDailyCalorieNeed(?p, ?m)
Dog(?p) ^ hasNeuterStatus(?p, intact) ^ hasRER(?p, ?r) ^ swrlb:multiply(?m, ?r, 1.8) -> hasDailyCalorieNeed(?p, ?m)
Cat(?p) ^ hasNeuterStatus(?p, neutered) ^ hasRER(?p, ?r) ^ swrlb:multiply(?m, ?r, 1.2) -> hasDailyCalorieNeed(?p, ?m)
Cat(?p) ^ hasNeuterStatus(?p, intact) ^ hasRER(?p, ?r) ^ swrlb:multiply(?m, ?r, 1.4) -> hasDailyCalorieNeed(?p, ?m)
```

---

## 8. Sample data

### 8.1 Products

| Individual | Class | For | Life stage | Main protein | Other ingredients | Recommended for | Targets lifestyle | Price (THB) | kcal/kg | Grain-free | Brand |
|---|---|---|---|---|---|---|---|---|---|---|---|
| SeniorCatSalmonDry | DryFood | Cat | senior | salmon | rice | hairball | | 650 | 3700 | No | BrandA |
| IndoorCatChickenLight | DryFood | Cat | adult, senior | chicken | rice, pumpkin | obesity, hairball | indoor, neutered | 720 | 3300 | No | BrandB |
| KidneyCareCatWet | PrescriptionDiet | Cat | senior | chicken | | kidneyDisease | | 65 | 1100 | Yes | BrandA |
| KittenSalmonDry | DryFood | Cat | juvenile | salmon | rice, taurine | | | 420 | 4000 | No | BrandB |
| AdultDogLambGrainFree | DryFood | Dog | adult | lamb | pumpkin | chickenAllergy, skinAllergy | | 890 | 3500 | Yes | BrandB |
| LightDogChickenWet | WetFood | Dog | adult | chicken | carrot | obesity | neutered, lowActivity | 75 | 900 | Yes | BrandA |
| LargeDogBeefDry | DryFood | Dog | adult | beef | corn | hipDysplasia | | 1590 | 3600 | No | BrandB |
| SeniorSmallDogChicken | DryFood | Dog | senior | chicken | rice | dentalPlaque | | 560 | 3500 | No | BrandB |
| DentalChew | Treat | Dog (small) | | chicken | | dentalPlaque | | 180 | | | BrandB |
| JointSupplement | Supplement | Dog, Cat | | | | hipDysplasia, arthritis | | 450 | | | BrandB |
| DeSheddingBrush | Brush | Dog, Cat | | | | hairball | | 290 | | | BrandA |
| StainlessBowl | Bowl | Dog, Cat | | | | | | 120 | | | BrandB |
| RopeToyLarge | Toy | Dog (large) | | | | | | 150 | | | BrandB |

`StainlessBowl goesWellWith LightDogChickenWet, KidneyCareCatWet`.

### 8.2 Pets

Only the **asserted** facts are entered. Everything in the "Inferred" columns is produced by the reasoner.

| Pet | Species | Breed | Age | Weight (kg) | Environment | Activity | Neuter | Condition |
|---|---|---|---|---|---|---|---|---|
| Mali | Cat | persian | 9 | 4.5 | indoor | lowActivity | neutered | |
| Buddy | Dog | labrador | 4 | 30 | outdoor | highActivity | intact | chickenAllergy |
| Momo | Dog | shihTzu | 10 | 6 | indoor | lowActivity | neutered | |
| Leo | Cat | siamese | 2 | 4 | outdoor | highActivity | intact | |

| Pet | Inferred coat | Inferred size | Inferred life stage | Inferred risks | RER (kcal) | Daily need (kcal) |
|---|---|---|---|---|---|---|
| Mali | longHair | | senior | hairball, kidneyDisease, obesity | 205 | 246 |
| Buddy | shortHair | large | adult | obesity, hipDysplasia | 970 | 1746 |
| Momo | longHair | small | senior | dentalPlaque, obesity | 250 | 400 |
| Leo | shortHair | | adult | none | 190 | 266 |

Leo is the control case: an outdoor, active, short-haired cat with no breed predisposition, so no risk rule fires.

---

## 9. Competency questions and queries

Queries are written for the **Snap SPARQL** tab in Protégé, which sees inferred facts. All results below were checked against the ontology after running Pellet.

Every query starts with:

```sparql
PREFIX : <http://www.example.org/petshop#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
```

### 9.1 Simple competency questions

These are answered from asserted facts alone. They still work with the reasoner off, apart from type checks on parent classes such as `PetFood`.

**S1. Which products go well with a specific product?**

```sparql
SELECT DISTINCT ?other WHERE { :StainlessBowl :goesWellWith ?other . }
```

Result: KidneyCareCatWet, LightDogChickenWet

**S2. Which accessories suit a pet of a specific size?**

```sparql
SELECT DISTINCT ?toy WHERE { ?toy a :Toy ; :isSuitableForSize :large . }
```

Result: RopeToyLarge

**S3. Which products are recommended for a specific health condition?**

```sparql
SELECT DISTINCT ?product WHERE { ?product :isRecommendedFor :hairball . }
```

Result: DeSheddingBrush, IndoorCatChickenLight, SeniorCatSalmonDry

**S4. Which products are suitable for both dogs and cats?**

```sparql
SELECT DISTINCT ?product WHERE {
  ?product :isSuitableFor ?c , ?d .
  ?c a :Cat . ?d a :Dog .
}
```

Result: DeSheddingBrush, JointSupplement, StainlessBowl

**S5. Which products of a specific brand cost less than a given price?**

```sparql
SELECT DISTINCT ?product ?price WHERE {
  ?product :hasBrand :BrandA ; :hasPrice ?price .
  FILTER(?price < 500)
}
```

| product | price |
|---|---|
| KidneyCareCatWet | 65 |
| LightDogChickenWet | 75 |
| DeSheddingBrush | 290 |


Result: AdultDogLambGrainFree, KittenSalmonDry, LargeDogBeefDry, SeniorCatSalmonDry

### 9.2 Complex competency questions

These need SWRL rules first. With the reasoner off, each query returns nothing.

---

## 10. Knowledge sources

- Noy, N. F. and McGuinness, D. L. (2001). *Ontology Development 101: A Guide to Creating Your First Ontology.* Stanford Knowledge Systems Laboratory.
- AAFCO (Association of American Feed Control Officials): pet food nutrient profiles by life stage.
- WSAVA Global Nutrition Guidelines: diet-related conditions and energy requirements.
- Common Pet Health Issues
- Food Allergies in Dogs and Cats
- What are the characteristics of a good pet? – The Institute for Environmental Research and Education
- All pet food brands rated by PetFoodRate
- Manufacturer product catalogs (for example Royal Canin, Hill's Pet Nutrition, Purina): product categories, life-stage and breed-specific lines.
- Stanford Protégé documentation and SWRLTab documentation.
