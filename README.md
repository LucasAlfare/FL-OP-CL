# FL One Piece Card Language (FL-OP-CL)

## Overview

FL One Piece Card Language (FL-OP-CL) is my custom domain-specific language (DSL) designed specifically for the **One Piece Card Game**.

It defines a structured representation of cards, abilities, effects, conditions, costs, selections, and resolution procedures, grounded in the official game rules.

The language transforms card definitions from their original data and natural-language text into formal, machine-readable programs that can be validated, compiled, and executed by compatible game engines.

## Core Concept

FL-OP-CL is built around a computational model:

- **Card Program:** Describes a card's abilities and executable behavior.
- **Game State:** Represents the current state of the match.
- **Virtual Machine:** Interprets and executes card programs.
- **Rules Engine:** Coordinates execution according to One Piece Card Game rules.

Cards define behavior. The execution environment determines how that behavior affects the game state.

## Goals

- Formalize the representation of One Piece Card Game cards.
- Represent card abilities and effects as structured programs.
- Define deterministic semantics for supported game mechanics.
- Support translation from natural-language card text into machine-readable definitions.
- Enable independent validation and compilation.
- Explicitly identify unsupported or ambiguous mechanics instead of inventing approximations.

## Language Format

The official textual representation uses **TOML 1.0**.

The language includes structures for:

- Card identity and properties.
- Leaders, Characters, Events, Stages, and DON!! cards.
- Auto, Activate, Permanent, and Replacement Effects.
- Conditions, costs, and restrictions.
- Targeting, selection, and choice procedures.
- Game-state operations.
- Sequential effect resolution.

All language constructs are defined specifically in the context of One Piece Card Game rules.

## Scope

FL-OP-CL is exclusively designed for the One Piece Card Game.

It does not aim to provide a universal card-definition language or abstract the mechanics of unrelated trading card games.

Its architecture may inspire similar projects, but its grammar, semantics, and instruction set are specific to One Piece Card Game.

## Specification

The normative specification is maintained in a single document:

`FL_CARD_LANGUAGE_SPEC_v1.0.0.md`

It contains the language grammar, semantic contract, supported constructs, validation requirements, and TOML examples.

This document is the sole normative reference for version 1.0.0.

## Status

**Version 1.0.0 — Frozen**

The specification is frozen for initial card translation and validation experiments.

This does not imply that every One Piece Card Game card or mechanic is already supported by an implementation.

Cards requiring unsupported or unresolved mechanics must be explicitly marked as blocked rather than translated using speculative semantics.

## License MIT

[LICENSE](LICENSE)
