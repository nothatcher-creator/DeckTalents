# Talent Shop

Version 1.1.1 keeps the existing Talent Shop content while improving pricing, transactions, and shop-session reliability.

The Talent Shop uses the same permanent Talent Point bank as deck and Global Talents. Each offer may be purchased once per shop and becomes available again after leaving for the next Blind.

## Earning points

- Defeat a Boss Blind: +1 Talent Point
- Skip a Small or Big Blind: +1 Talent Point

Both rewards use the animated Talent Point acquisition effect.

## Vanilla rewards

### The Soul — 7 points
Creates The Soul Spectral card in your consumable area.

### Black Hole — 5 points
Creates a Black Hole Spectral card.

### Cryptid — 4 points
Creates a Cryptid Spectral card.

### Negative Joker — 7 points
Creates a random Joker with the Negative edition.

### Polychrome Joker — 6 points
Creates a random Joker with the Polychrome edition.

### Double Tag Bundle — 4 points
Immediately grants two Double Tags.

## New creations

### Royal Forge — 5 points
Creates a hidden Spectral consumable. Select one playing card and use Royal Forge to turn it into:

- King of Spades
- Glass Card
- Red Seal

### Numerical Paradox — 5 points
Creates a hidden Spectral consumable that turns one selected card into an Omni Number.

An Omni Number:

- has a default value of 12 Chips
- counts as any numbered rank from 2 through 10 during poker-hand detection
- can complete pairs, multiples, full houses, and straights as the required numbered rank

For performance, the first two Omni Numbers in a hand receive full wild-rank evaluation. Additional Omni Numbers still score 12 Chips and use their default rank for hand detection.

### Talent Battery — 8 points
Permanently grants +1 hand and +1 discard for the current run.

### Vault Expansion — 9 points
Permanently grants +1 Joker slot and +1 consumable slot for the current run.

### Bossbreaker Sigil — 10 points
All played hands against Boss Blinds gain X1.5 Mult for the rest of the run. Multiple purchases from later shops stack multiplicatively.

## Shop rules

- An offer can be bought once in each shop.
- Purchases use a validated transaction against the permanent Talent Point bank.
- If a reward cannot be granted, its Talent Point cost is refunded automatically.
- Consumable purchases require an open consumable slot.
- Polychrome Joker requires an open Joker slot.
- Negative Joker can provide its own additional Joker capacity.
- Shop purchases work with all decks, including Talentless.

## Access

The Talent Shop is not unlockable. A **TALENT SHOP** button is injected into every normal shop and displays the current Talent Point balance. Version 1.1.4 hooks both known shop UI constructors for broader Steamodded and Cryptid compatibility.

## Version 1.1.5 display behaviour

The normal-shop access button is inserted horizontally beside an existing shop control so it does not add height to the shop. The Talent Shop overlay displays one centred offer per page; all offers remain available through the page arrows.

## Version 1.1.6 access behaviour

The normal-shop entry is rendered as a separate floating UIBox after the shop opens. It does not add a row or column to the vanilla/Cryptid shop definition. Deck Talents rechecks both known shop constructors throughout gameplay and re-wraps them if another mod replaces either function later in the load order.
