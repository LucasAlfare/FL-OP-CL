# FL Card Language (FCL) — Specification 1.7.0

Normative document: `language_spec.md`\
Textual format: TOML 1.0.0\
Status: `FROZEN — READY FOR CARD-TRANSLATION TESTING`

## 1. Purpose and Authority

The FL Card Language (FCL) is a declarative language designed exclusively for the One Piece Card Game. It represents card abilities and effects as structured programs interpreted by a Rules Engine/Virtual Machine (VM).

Cards declare behavior, official rules establish procedures and consequences, and the engine executes those procedures.

FCL is not a general-purpose programming language. It does not support Kotlin code, user-defined functions, callbacks, mutable variables, arbitrary state access, unrestricted loops, or recursion.

FCL defines the complete textual format, semantic profile, vocabularies, effects, conditions, selections, payments, resolution structures, continuous effects, Replacement Effects, validation rules, and blocking policy. It supersedes previous specifications and addenda.

The external syntax is TOML 1.0.0. FCL defines a closed set of permitted structures, fields, types, enumerations, and semantic invariants. `language_version` identifies the language structure and semantics.

`FROZEN` means that the language contract is closed and ready for card-translation testing. It does not imply complete coverage of published cards or an implemented compiler and VM.

Behavior outside the defined profile must be blocked rather than approximated.

Structural or semantic changes require a new language version.

## 2. Scope and Responsibility Boundaries

The normative foundation is the ONE PIECE CARD GAME Comprehensive Rules v1.2.1 (August 28, 2026), together with applicable official card text, errata, and FAQs.

### 2.1 Included

- Card identity, characteristics, and printing-specific gameplay overrides.
- Editorial metadata and source provenance.
- Auto, Activate, Permanent, and Replacement Effects.
- Recognized Keyword Effects and their official procedures.
- Conditions, predicates, filters, validity, and evaluation contexts.
- Event activation and Event-resolution timing.
- Opponent Character K.O. timing and contextual event references.
- Values, references, selections, quantities, and information.
- Activation costs and supported payments.
- Optional payments and their distinct semantics.
- Life-to-hand payments.
- Hand-to-bottom-of-Deck payments.
- Character-to-owner-hand payments.
- Direct source-instance payments.
- Multiple-card resting payments using `REST_CARD`.
- Ordered resolution, branches, dependencies, simultaneity, and finite repetition.
- LOOK-result manipulation and remainder sets.
- Additional attack-target permissions.
- Restrictions on attack targets.
- Prevention of K.O. caused by specified battle attackers.
- Enumerated game operations.
- Continuous effects, scope, validity, and duration.
- Source references, traceability, validation, and blocking.

### 2.2 Excluded

- Complete implementation of turns, phases, battles, Blocker, Counter, damage, Life, Trigger, K.O., priority, `rule processing`, and defeat.
- Rules Engine/VM implementation, bytecode, and binary layout.
- Format legality, banlists, and deck construction.
- Operations, conditions, timings, durations, properties, and behaviors not explicitly enumerated here.
- Card-specific executable code.
- Arbitrary modification of game procedures.
- Generic effect negation.
- Arbitrary attack redirection.
- Unrestricted action prohibition.
- Generic granting of arbitrary abilities.
- Unrestricted attack-rule modification.

Deck construction effects are outside the executable gameplay profile. Editorial legality information may be stored separately.

### 2.3 Mandatory Semantic Principles

1. Category and duration are distinct. The four categories are `AUTO`, `ACTIVATE`, `PERMANENT`, and `REPLACEMENT`. `ONE_SHOT` and `CONTINUOUS` describe effect form, not additional categories.
2. Activation, payment, selection, choice, and resolution are distinct. Their timing and responsibilities must not be conflated.
3. Impossibility is contextual. An unsatisfied condition, impossible cost, impossible operation, absent designated object, empty selection, zero `UP_TO` choice, declined optional payment, declined replacement, and invalidated effect are different situations.
4. No inferred equivalences. K.O. is not Trash; damage is not Life movement; drawing is not selecting from the Deck; playing is not ordinary movement to the Field; Trigger is not arbitrary Life movement; effective characteristics are not base characteristics; owner is not controller; chooser is not executor.
5. Syntax does not establish semantics. Unknown players, objects, timing, quantities, ordering, costs, scope, or consequences require blocking.
6. The card does not reimplement the engine. Priority, phases, battle, damage processing, K.O., Trigger, continuous-effect stabilization, information procedures, replacement priority, and general loops belong to the engine.
7. Logical identity does not guarantee identical printings. Verified gameplay differences between printings use `card.printing_overrides`, never editorial metadata.
8. Multicolor cards possess every printed color. Comparisons use complete color sets.
9. LOOK results are temporary identified sets, not permanent selections or independent areas.
10. `SOURCE` identifies an exact instance, never an arbitrary substitute.
11. Attack-target permissions, attack-target restrictions, and battle K.O. prevention are distinct, narrowly defined continuous consequences. None is a generic mechanism for modifying game rules.
12. A `SUPPORTED` translation requires structural validation, semantic completeness, and normative justification.
13. Event-triggered effects must preserve the identity and relevant context of the event that fulfilled their timing. Subsequent state changes must not retroactively alter event-time conditions when official rules establish that those conditions are evaluated at the event.

A valid TOML document can still contain an incorrect translation.

## 3. Authoring Unit

### 3.1 One Logical File per Card

The authoring unit is one TOML file per logical card identity (official card number).

Alternate artwork and physical printings sharing the same gameplay identity must not duplicate card programs.

A file has five conceptual sections:

1. `[card]` and `[[card.printing_overrides]]`: gameplay identity and characteristics.
2. `[[abilities]]`: executable abilities and effects.
3. `[source]`: original/current text and normative references.
4. `[translation]`: translation status and blockers.
5. `[editorial]`: printings and non-executable metadata.

Multiple API records with the same official card number must be consolidated when they represent the same logical card.

Provider-specific differences in display names, artwork labels, image identifiers, suffixes, rarity, set names, and printing identifiers do not independently establish gameplay conflicts.

A genuine gameplay conflict concerns an unresolved difference in authoritative gameplay information, including:

- Current official text.
- Category.
- Colors.
- Printed types.
- Attributes.
- Cost, Power, Life, or Counter.
- Another characteristic affecting gameplay.

Unofficial API differences alone are insufficient evidence of a gameplay conflict.

Use the official card name in `[card].name`. Represent physical artworks through `[[editorial.printings]]`.

Verified printing-specific gameplay differences use §6.3.

### 3.2 Non-Instructions

Identity, characteristics, provenance, and editorial information are not executable instructions.

Editorial metadata and normative references must not be sent to the VM.

Printing-specific gameplay characteristics are gameplay data and must be available when constructing and evaluating card instances.

## 4. Mandatory TOML Rules

The language accepts TOML 1.0.0 only.

| Rule           | Requirement                                                       |
| -------------- | ----------------------------------------------------------------- |
| Encoding       | UTF-8                                                             |
| Keys           | Case-sensitive `snake_case`                                       |
| Enumerations   | Strings in `UPPER_SNAKE_CASE`                                     |
| IDs            | Stable strings, unique in their defined scope                     |
| Arrays         | Order matters for sequences, choices, alternatives, and printings |
| Tables         | Physical order has no semantic meaning                            |
| Missing values | Omit inapplicable fields; TOML has no `null`                      |
| Unknown keys   | Reject, except authorized `[editorial.extra]`                     |
| Duplicate keys | Reject                                                            |
| Dates          | ISO `YYYY-MM-DD` strings                                          |
| Numbers        | Decimal integers unless a string representation is specified      |
| Comments       | Allowed; no gameplay meaning                                      |

Do not use YAML features, custom tags, aliases, or free-form text in place of required semantic structures.

### 4.1 Ordering

Resolution order is represented by TOML arrays, never physical table/key order.

An array of Conditions represents conjunction only when its containing field explicitly defines that semantics.

### 4.2 Inline Tables

Inline tables represent references, values, conditions, and compact nodes. They must follow TOML 1.0.0 syntax.

Long resolution sequences may use arrays of tables.

## 5. Top-Level Structure

The only permitted top-level keys/tables are:

- `format`
- `language_version`
- `[card]`
- `[source]`
- `[translation]`
- `[editorial]`
- `[[abilities]]`

`[[card.printing_overrides]]` is a permitted descendant of `[card]`.

Structural example:

```
format = "FL_CARD_LANGUAGE"
language_version = "1.6.0"

[card]
number = "EXAMPLE-001"
name = "Example Leader"
category = "LEADER"
colors = ["RED"]
types = ["Example Crew"]

[source]
raw_text = "..."
current_text = "..."
language = "en"
region = "EN_NA_EU_OC_LATAM_ME"

[translation]
status = "SUPPORTED"

[[abilities]]
id = "A1"
source_text = "..."
status = "SUPPORTED"
```

## 6. Game Identity — `[card]`

Permitted fields:

| Field                 | Type             | Requirement                            |
| --------------------- | ---------------- | -------------------------------------- |
| `number`              | string           | Required; official card number         |
| `name`                | string           | Required; official card name           |
| `category`            | enum             | Required                               |
| `colors`              | array of enums   | Required; non-empty                    |
| `types`               | array of strings | Printed types                          |
| `attributes`          | array of enums   | Leader/Character only                  |
| `cost`                | integer          | Character/Event/Stage only             |
| `power`               | integer          | Leader/Character only                  |
| `life`                | integer          | Leader only                            |
| `counter`             | integer          | Character only                         |
| `default_printing_id` | string           | Required when printing overrides exist |

### 6.1 Enumerations and Types

`category`:

- `LEADER`
- `CHARACTER`
- `EVENT`
- `STAGE`
- `DON`

`colors`:

- `RED`
- `GREEN`
- `BLUE`
- `PURPLE`
- `BLACK`
- `YELLOW`

`attributes`:

- `SLASH`
- `STRIKE`
- `RANGED`
- `SPECIAL`
- `WISDOM`
- `QUESTION_MARK`

Types are literal strings, not closed enumerations.

Every distinct official type occupies one array element.

Example:

```
types = ["Fish-Man", "Straw Hat Crew"]
```

Do not split concatenated API type strings by whitespace alone. Multiword type names exist. Establish individual boundaries from an authoritative source; do not invent them.

### 6.2 Applicability

- `life`: Leader only.
- `power`: Leader or Character.
- `counter`: Character only.
- `cost`: Character, Event, or Stage.
- `attributes`: Leader or Character.

Do not populate inapplicable fields with zero or empty strings. Absence and numerical zero are different.

### 6.3 Printing-Specific Gameplay Characteristics

`[[card.printing_overrides]]` represents verified gameplay characteristic differences between printings of the same logical card.

Each entry accepts:

| Field         | Type    | Requirement                             |
| ------------- | ------- | --------------------------------------- |
| `printing_id` | string  | Required; matches an editorial `art_id` |
| `colors`      | array   | Optional override                       |
| `types`       | array   | Optional override                       |
| `attributes`  | array   | Optional override                       |
| `cost`        | integer | Optional override                       |
| `power`       | integer | Optional override                       |
| `life`        | integer | Optional override                       |
| `counter`     | integer | Optional override                       |

`number`, `name`, and `category` cannot be overridden.

When overrides exist:

- `default_printing_id` is required and must match exactly one editorial `art_id`.
- It identifies the printing represented by the base `[card]` values.
- Each `printing_id` must match exactly one editorial `art_id`.
- A printing may have at most one override.
- The default printing cannot have an override.
- Omitted override fields inherit their base values.

If all printings share the same characteristics, no override table is needed.

Never combine differing characteristics through unions or intersections.

Only reliable official evidence authorizes an override. Unofficial API conflicts, names, artwork labels, and suffixes are insufficient.

Example:

```
[card]
number = "OP01-008"
name = "Cavendish"
category = "CHARACTER"
colors = ["RED"]
types = ["Beautiful Pirates", "Supernovas"]
attributes = ["SLASH"]
cost = 4
power = 5000
counter = 0
default_printing_id = "OP01-008"

[[card.printing_overrides]]
printing_id = "OP01-008_p1"
types = ["Beautiful Pirates", "Straw Hat Crew", "Supernovas"]
```

The engine applies the matching override before evaluating the instance's gameplay characteristics. Without a matching override, the base values apply.

## 7. Normative Source — `[source]`

Required fields:

- `raw_text`
- `current_text`
- `language`
- `region`

`edition` is optional.

`raw_text` preserves exactly the input received.

`current_text` contains the applicable current official gameplay text after errata and corrections.

Requirements:

1. Official errata takes precedence over original printed wording within its stated scope.
2. Preserve wording, order, capitalization, punctuation, numbers, negations, restrictions, and meaningful symbols.
3. Do not paraphrase, simplify, or invent synonyms.
4. Exclude API annotations and editorial messages from `current_text`. Such annotations may remain in `raw_text`.
5. Determine current text before decomposing abilities.
6. Historical differences caused by errata do not create different gameplay programs.

All normative sources share one `[[source.references]]` list.

Permitted fields:

| Field      | Type   | Requirement                                 |
| ---------- | ------ | ------------------------------------------- |
| `id`       | string | Required; unique                            |
| `kind`     | enum   | `CR`, `FAQ`, `ERRATA`, `OFFICIAL_CARD_TEXT` |
| `document` | string | Required; identifiable document             |
| `url`      | string | Required; actual source URL                 |
| `version`  | string | Optional                                    |
| `section`  | string | Optional                                    |
| `date`     | string | Optional; ISO date                          |

Example:

```
[[source.references]]
id = "CR-8-3-1-4"
kind = "CR"
document = "ONE_PIECE_CARD_GAME_COMPREHENSIVE_RULES"
version = "1.2.1"
section = "8-3-1-4"
url = "https://en.onepiece-cardgame.com/pdf/rule_comprehensive.pdf?20260828="
```

`source_refs`, `replacement.target.rule_ref`, and `translation.blockers.references` must reference existing IDs.

A FAQ establishes only the cases it addresses.

References must identify actual sources. Never fabricate URLs, dates, versions, sections, or identifiers.

An `ERRATA` reference must identify the actual correction applied. If the correction or its scope cannot be established, do not invent corrected text.

An applicable `ERRATA` reference is mandatory whenever official errata causes `current_text` to differ from `raw_text`.

When an interpretation requires an official FAQ, associate its reference with the applicable ability using `source_refs`.

References are required for external normative determinations not sufficiently established by the card text and this specification.

Ordinary authoritative card characteristics do not require artificial references.

## 8. Translation Result — `[translation]`

Required: `status`.

Optional: `notes`, which has no executable meaning.

Permitted statuses:

- `SUPPORTED`
- `BLOCKED_NORMATIVE`
- `BLOCKED_CAPABILITY`
- `INVALID_SOURCE`
- `NON_GAME_TEXT`

Blockers use `[[translation.blockers]]`.

| Field            | Type   | Requirement                                                 |
| ---------------- | ------ | ----------------------------------------------------------- |
| `code`           | enum   | `BLOCKED_NORMATIVE`, `BLOCKED_CAPABILITY`, `INVALID_SOURCE` |
| `scope`          | enum   | `CARD`, `ABILITY`, `EFFECT`                                 |
| `ability_id`     | string | Required for ABILITY/EFFECT; forbidden for CARD             |
| `effect_id`      | string | Required only for EFFECT                                    |
| `source_excerpt` | string | Exact excerpt from `current_text`                           |
| `reason`         | string | Required objective explanation                              |
| `references`     | array  | Optional existing source IDs                                |

Example:

```
[translation]
status = "BLOCKED_CAPABILITY"

[[translation.blockers]]
code = "BLOCKED_CAPABILITY"
scope = "ABILITY"
ability_id = "A1"
source_excerpt = "..."
reason = "The required semantic construct is not defined in this language version."
```

### 8.1 Consistency

- Only `SUPPORTED` programs may be compiled for gameplay.
- Any blocked gameplay ability blocks the entire card.
- Status severity: `INVALID_SOURCE` > `BLOCKED_NORMATIVE` > `BLOCKED_CAPABILITY` > `SUPPORTED`.
- Every blocked status requires at least one corresponding blocker.
- `SUPPORTED` and `NON_GAME_TEXT` cannot have blockers.
- Every ABILITY/EFFECT blocker must reference existing declarations.
- CARD blockers cannot contain `ability_id` or `effect_id`.
- Every blocked ability must have corresponding blocker evidence.
- `notes` cannot replace `reason`.

A `NON_GAME_TEXT` ability contains no executable instruction.

`translation.status = "NON_GAME_TEXT"` is permitted only when the entire card has no gameplay ability, including a card with no card text.

If gameplay abilities exist, the card status is the most severe status among those abilities and CARD-scope blockers. `NON_GAME_TEXT` abilities do not participate.

A card cannot be `SUPPORTED` when any gameplay instruction has been omitted.

## 9. Editorial Metadata — `[editorial]`

`[editorial]` is optional and never executable.

Allowed top-level fields:

- `rarity`
- `set_id`
- `set_name`
- `illustrator`
- `thumbnail_url`
- `rights`
- `block_number`

Allowed descendant structures:

- `[[editorial.printings]]`
- `[editorial.extra]`

Each printing accepts only:

- `set_id`
- `rarity`
- `art_id`
- `illustrator`
- `thumbnail_role`
- `thumbnail_url`
- `rights`

`thumbnail_role` accepts `SAMPLE`, `CUSTOM`, or `FINAL`.

`rights` is descriptive metadata, not a license.

`art_id` identifies a physical printing.

`[editorial.extra]` accepts only TOML scalar values (`string`, `integer`, `float`, `boolean`) and homogeneous arrays of those scalars. Tables and arrays of tables are forbidden.

Example:

```
[[editorial.printings]]
set_id = "OP-01"
rarity = "SR"
art_id = "OP01-008"
thumbnail_role = "SAMPLE"
thumbnail_url = "https://example.invalid/card.png"
rights = "UNVERIFIED"

[editorial.extra]
source_api = "optcgapi"
collector_note = "Alternate art"
```

Artwork differences never duplicate executable programs. Verified gameplay differences use §6.3.

## 10. Abilities — `[[abilities]]`

Each entry represents one logical unit of gameplay text.

Fields:

- `id`: required and unique within the card.
- `source_text`: required; exact excerpt from `current_text`.
- `status`: required; one of the five translation statuses.
- `source_refs`: optional existing source IDs.
- `keyword_effects`: optional recognized Keyword Effects.
- `event_activation`: optional Event-card activation information.
- `effects`: zero or more effect declarations.

A `SUPPORTED` ability must be fully represented by `effects`, `keyword_effects`, or `event_activation`.

Blocked abilities may retain understood portions but block the entire card.

Do not merge or split abilities when doing so changes timing, costs, conditions, choice, order, or dependency.

### 10.1 Keyword Effects

The closed `keyword_effects` vocabulary is:

- `RUSH`
- `DOUBLE_ATTACK`
- `BANISH`
- `BLOCKER`
- `TRIGGER`
- `RUSH_CHARACTER`
- `UNBLOCKABLE`

The engine executes each Keyword's official procedure.

`TRIGGER` requires exactly one of these representations:

1. Non-Event: `keyword_effects = ["TRIGGER"]` plus `AUTO/timing = "TRIGGER"`.
2. Event whose Trigger text says `Activate this card.`: `keyword_effects = ["TRIGGER"]` plus `event_activation.window = "TRIGGER"`.

Never declare both representations for the same occurrence or record only the keyword.

Other Keywords are represented according to their function: timing, condition, cost, usage limit, operation, or restriction.

### 10.2 Event Card Activation

`event_activation` describes authorization and procedure for activating the Event card itself, not its payment or a movement operation.

```
event_activation = { window = "MAIN" }
```

`window` accepts:

- `MAIN`
- `COUNTER`
- `TRIGGER`

Use `TRIGGER` only when the Trigger instructs the player to activate the Event card.

If the Trigger contains its own effect, represent it as `AUTO/timing = "TRIGGER"`.

Normal Event text is represented separately as `ACTIVATE` with `activation_window = "EVENT_CARD_RESOLUTION"`.

Activating an Event card and activating an ability are different actions.

A Trigger explicitly activating an Event's `[Main]` or `[Counter]` effect is itself a distinct gameplay instruction and must be represented.

Event activation windows and procedures belong to the engine.

If an Event is activated but its optional Activation Cost is declined, the Event card activation and the execution of its associated effect remain distinct. The Event may be activated without that effect resolving, consistently with the applicable FAQ.

### 10.3 Source Coverage and Ability Completeness

Every gameplay ability and instruction in `source.current_text` must be represented.

Identify all logical abilities before translation, including:

- Main, Counter, and Activate instructions.
- Auto Effect timings.
- Trigger instructions.
- Keyword Effects.
- Continuous permissions and restrictions.
- Conditions and conditional instructions.
- Clauses introduced by `Then`.
- Instructions referring to or activating another ability.
- Any other instruction changing, restricting, authorizing, or determining gameplay.

`source_text` must be an exact excerpt.

An ability may contain multiple sequential clauses, branches, and operations. Keep them together when splitting would change the original semantics.

Distinct activations remain separate even when they execute identical instructions.

In particular:

- Counter and Trigger are not interchangeable.
- A Counter representation does not cover a separate Trigger.
- `[Trigger] Activate this card's [Counter] effect.` is a separate instruction.
- A Trigger with its own effect must be represented independently.
- Translating Main while omitting Trigger is incomplete.

The division of text among `source_text` excerpts must account for every gameplay instruction. Whitespace and non-gameplay annotations do not require executable representation.

A card cannot be `SUPPORTED` when:

1. A current-text ability has no ability entry.
2. An entry omits a gameplay clause.
3. A clause lacks semantic representation.
4. A Trigger, Keyword Effect, cost, restriction, or instruction is ignored.
5. The complete current text has not been examined.

Completeness must be checked against the entire `source.current_text`, not merely the abilities already identified.

## 11. Effect Declarations — `[[abilities.effects]]`

Each effect requires:

- `id`: unique within the ability.
- `category`: `AUTO`, `ACTIVATE`, `PERMANENT`, or `REPLACEMENT`.
- `source_refs`: optional existing source IDs.
- Category-specific fields.

Category is normative; never infer it from an operation.

### 11.1 Auto Effect

Required fields:

- `timing`
- `effect_form`
- `resolution` for `ONE_SHOT`, or `continuous` for `CONTINUOUS`

Optional fields:

- `activation_conditions`
- `activation_costs`
- `usage_limit`

Supported timings:

- `ON_PLAY`
- `ON_KO`
- `WHEN_ATTACKING`
- `ON_OPPONENT_ATTACK`
- `ON_BLOCK`
- `TRIGGER`
- `START_OF_GAME`
- `START_OF_TURN`
- `START_OF_MAIN_PHASE`
- `END_OF_YOUR_TURN`
- `END_OF_OPPONENT_TURN`
- `AFTER_EVENT_CARD_RESOLUTION`
- `ON_OPPONENT_CHARACTER_KO`

#### Timing semantics

Existing timings

- `ON_KO` follows the official K.O. sequence for the source instance itself, including its prescribed source movement and resolution timing. Never substitute generic Trash for K.O.
- `AFTER_EVENT_CARD_RESOLUTION` occurs only after an actual Event card activation has completely resolved. It does not occur for Event Trigger activation, ordinary Event movement, non-Event card effects, or mere revelation, selection, or Trash movement of an Event.
- `EVENT_ACTIVATED_BY` identifies the player who activated the Event associated with the current `AFTER_EVENT_CARD_RESOLUTION`.
- `TRIGGER` is the window in which a card with Trigger may be activated during official Life damage processing, instead of being added to its owner's Hand.
- `START_OF_GAME` is supported only when the official game-start procedure establishes that timing.
- Other timings follow their corresponding official game procedures.

`ON_OPPONENT_CHARACTER_KO` — FCL 1.6.0

This timing fulfills when an opponent's Character is actually K.O.'d.

It is distinct from `ON_KO`:

- `ON_KO` observes the K.O. of the effect's own source instance.
- `ON_OPPONENT_CHARACTER_KO` observes another Character's K.O., specifically a Character in the opponent's Character Area.

The following rules are mandatory:

1. The opponent is determined relative to the player controlling the source ability when the event occurs.
2. The event occurs only when a Character in that opponent's Character Area is officially K.O.'d through battle or a card effect.
3. Ordinary Trash or movement to the Trash does not generate this timing.
4. A K.O. prevented or replaced before it actually occurs does not generate this timing.
5. The engine retains the exact K.O.'d Character instance as `EVENT_OBJECT`.
6. `SOURCE` continues to identify the instance containing the observing ability. It must never be confused with `EVENT_OBJECT`.
7. The engine retains the contextual player relationship and event identity for the corresponding Auto Effect.
8. Each individual Character K.O. is an event occurrence. When multiple Characters are K.O.'d, the engine processes the corresponding timing occurrences under the official event and effect-ordering rules.
9. Activation conditions are evaluated at the K.O. event's occurrence, not at a later point when the engine eventually activates the pending effect.
10. Conditions involving the source's state or attached DON!! must use their values at that event point. Subsequent changes do not retroactively change whether the condition was satisfied.
11. Normal Auto Effect activation restrictions still apply. If the source leaves its required area before activation, the pending Auto Effect does not activate under CR 8-1-3-1-3.
12. Activation and resolution ordering follow the applicable Comprehensive Rules, including their procedures for events occurring during damage processing or another effect's resolution.

The official OP01-061 FAQ confirms the event-time condition requirement: if the Leader loses the DON!! required by `[DON!! x1]` simultaneously with the opponent's Character being K.O.'d, the ability does not activate because its condition was not satisfied when that Character was K.O.'d.

`PLAYER_TURN`, `DON_GIVEN_TO_SOURCE`, and other applicable activation conditions must therefore be evaluated against the state at the triggering occurrence.

`EVENT_OBJECT` identifies the exact Character responsible for the timing and remains available as a contextual reference throughout its associated effect. Its identity is retained even after its K.O. movement to its owner's Trash.

Do not use `ON_KO`, `ON_PLAY`, `WHEN_ATTACKING`, or another existing timing as an approximation of this event.

The engine is responsible for event generation, timing fulfillment, and official ordering. FCL only declares the supported timing and its associated behavior.

### 11.2 Activate Effect

Required fields:

- `activation_window`
- `effect_form`
- `resolution` for `ONE_SHOT`, or `continuous` for `CONTINUOUS`

Optional fields:

- `activation_conditions`
- `activation_costs`
- `usage_limit`

`activation_window` accepts:

- `MAIN_PHASE`
- `EVENT_CARD_RESOLUTION`

The Event card's own `MAIN`, `COUNTER`, or `TRIGGER` authorization remains in `event_activation`.

### 11.3 Permanent Effect

Required: `continuous`.

`continuous` contains exactly one of `subject` or `scope`, subject to the restrictions of its `kind`.

`valid_when` is optional. If omitted, the effect remains valid whenever the card and rules permit it. If present, it is a continuously evaluated Condition.

Do not use `resolution` for one-time actions in a Permanent Effect.

### 11.4 Replacement Effect

Required: `replacement`.

Optional:

- `available_when`
- `duration`, only when text/rules delimit availability.

Replacement is its own category, not an operation or Continuous subtype.

It has no `effect_form`.

If a replacement procedure creates a continuous consequence, that consequence must declare its own continuous structure and duration.

## 12. Conditions and Predicates

A Condition is an inline table containing exactly one form:

```
all = [CONDITION, ...]
```

```
any = [CONDITION, ...]
```

```
not = CONDITION
```

```
test = "TEST_NAME"
```

`all` and `any` require at least one item.

Conditions are permitted in:

- `activation_conditions`
- `valid_when`
- `available_when`
- Selection `filter`
- `IF.condition`
- Applicable continuous-effect conditions and filters.

Evaluation point:

- Activation conditions: before activation, at the timing-defined point. For `ON_OPPONENT_CHARACTER_KO`, this is the event occurrence.
- Validity conditions: continuously.
- Resolution conditions: when the corresponding step is processed.
- Filters: during candidate evaluation.

### 12.1 Tests and Fields

| Test                  | Required fields                                                                                        |
| --------------------- | ------------------------------------------------------------------------------------------------------ |
| `PROPERTY_COMPARE`    | `subject`, `property`, `operator`, `value`; for `INCLUSIVE_RANGE`, use `minimum` and `maximum` instead |
| `PLAYER_TURN`         | `value`: `YOUR_TURN` or `OPPONENT_TURN`                                                                |
| `DON_GIVEN_TO_SOURCE` | `operator`, `value`                                                                                    |
| `DON_IN_FIELD`        | `player`, `operator`, `value`                                                                          |
| `AREA_COUNT`          | `area`, `player`, `operator`, `value`                                                                  |
| `OBJECT_COUNT`        | `object`, `areas`, `player`, `operator`, `value`                                                       |
| `SUM_COST`            | `selection`, `operator`, `value`                                                                       |
| `EVENT_ACTIVATED_BY`  | `player`                                                                                               |
| `COLOR_SET_COMPARE`   | `subject`, `relation`, `other_subject`                                                                 |

`PROPERTY_COMPARE.property` accepts:

- `CATEGORY`
- `NAME`
- `COLOR`
- `TYPE`
- `ATTRIBUTE`
- `POWER`
- `BASE_POWER`
- `COST`
- `BASE_COST`
- `LEADER_LIFE`
- `COUNTER`
- `STATE`
- `AREA`
- `PLAYER_CONTEXT`
- `OWNER`
- `CONTROLLER`
- `KEYWORD_EFFECT`
- `KEYWORD`
- `DON_GIVEN_TO_CARD`

Value types must match their properties:

- Numeric: Power, Cost, Life, Counter, DON given.
- String: Name, Type, Keyword.
- Enum: Category, Color, Attribute, State, Area, Keyword Effect.
- PLAYER reference: Owner, Controller, Player Context.

Undefined combinations are blocked.

For `COLOR` with `EQ`, the comparison succeeds if the card possesses that color, including multicolor cards.

`COLOR_SET_COMPARE` compares complete sets of two card objects.

Fields:

- `subject`
- `other_subject`
- `relation`

Relations:

- `DISJOINT`: no shared color.
- `OVERLAPS`: at least one shared color.
- `EQUAL`: exactly the same colors.

For "a different color than [specified card]", use `DISJOINT`, not array inequality.

`subject = CANDIDATE` is permitted only in a selection filter. `other_subject` must be a singular reference available at evaluation time.

`EVENT_ACTIVATED_BY` identifies the player who activated the Event associated with the current `AFTER_EVENT_CARD_RESOLUTION`.

It is valid only in `activation_conditions` of an `AUTO` effect with that timing. It cannot be used as a generic predicate, selection filter, or resolution condition.

`DON_GIVEN_TO_SOURCE` queries DON!! attached to the source instance. `[DON!! xX]` corresponds to `GTE X`.

`DON_IN_FIELD` counts DON!! in Field areas, excluding attached DON!!.

`AREA_COUNT.area` uses the §14 area catalog.

`OBJECT_COUNT.areas` contains AREA references. Its optional filter uses `CANDIDATE`.

`SUM_COST` sums Cost across a selected set; it does not compare individual costs.

Operators:

- `EQ`
- `GT`
- `GTE`
- `LT`
- `LTE`
- `INCLUSIVE_RANGE`

For `INCLUSIVE_RANGE`, use numeric `minimum` and `maximum`, with minimum no greater than maximum. `value` is forbidden.

For every other operator, `value` is required and `minimum`/`maximum` are forbidden.

There is no `NE` or general arithmetic.

### 12.2 Evaluation Context

`CANDIDATE` is the current object evaluated by a filter; it is not a general-purpose variable.

`EVENT_ACTIVATED_BY` reads the contextual Event activation retained by the engine, not the current Turn Player or card owner.

`EVENT_OBJECT` identifies the exact Character instance that fulfilled `ON_OPPONENT_CHARACTER_KO`. It is usable only inside the associated Auto Effect, in structures accepting singular references. It is not a global variable or arbitrary instance lookup.

When `EVENT_OBJECT` is used to examine the triggering Character, event-context characteristics are evaluated using the event information retained by the engine, subject to applicable rules and continuous effects.

Color comparison reads complete effective color sets at evaluation time, including printing overrides.

`PROPERTY_COMPARE` reads effective characteristics, including applicable printing overrides and continuous modifications.

Within `PREVENT_BATTLE_KO.attacker_condition`, `ATTACKER` is the actual attacker in the relevant battle, evaluated when the protected object would be K.O.'d.

Never infer `ATTACKER` from the Turn Player, source card, or controller.


## 13. Values

Values use inline objects containing exactly one form:

- `literal`: integer, string, boolean, or enum.
- `property = { subject = REF, name = "BASE_POWER" }`.
- `count_area = { area = AREA_REF }`.
- `count_objects = { object = "CARD", areas = [...], player = PLAYER_REF, filter = CONDITION }`.
- `count_selection = "selection_id"`.
- `don_count = { player = PLAYER_REF, scope = "FIELD" }`.
- `don_given = { subject = REF }`.
- `sum_cost = { selection = "selection_id" }`.
- `scale = { count = COUNT_VALUE, multiplier = INTEGER }`.

### 13.1 Count-Scaled Values

`scale` multiplies a supported dynamic count by a fixed positive integer:

`result = count × multiplier`

`COUNT_VALUE` must be an inline Value object containing exactly one of:

- `count_area`
- `count_objects`
- `count_selection`
- `don_count`
- `don_given`

Restrictions:

1. `count` must produce a non-negative integer.
2. `multiplier` must be a positive TOML integer greater than zero.
3. Nested `scale` expressions are forbidden.
4. `literal`, `property`, and `sum_cost` are not valid `scale.count` sources.
5. The resulting value is a non-negative integer.
6. `scale` is permitted only where a numeric Value is accepted.
7. All references and filters within `COUNT_VALUE` retain the rules and restrictions of their original Value forms.
8. Multiplication uses exact integer arithmetic. Implementations must not silently round, wrap, or saturate the result.

`scale` represents one calculated numeric value. It does not repeat an operation, create a loop, or execute a separate instruction for each counted object.

### 13.2 Evaluation

The underlying count is evaluated at the evaluation point established by the structure consuming the Value.

For resolution steps, conditions, and other one-time evaluations, use the evaluation point defined by the corresponding construct and applicable rules.

For continuous effects:

- Evaluate the current count whenever the engine evaluates the continuous effect.
- Recalculate the resulting value using the current game state.
- Do not snapshot the first calculated value when the effect activates, enters the Field, or is initially evaluated.
- Changes to the counted objects must be reflected through the engine's normal continuous-effect evaluation and stabilization procedures.

The engine remains responsible for continuous-effect timing, ordering, precedence, and stabilization under the Comprehensive Rules. `scale` does not introduce additional game events or evaluation checkpoints.

If evaluation involves a circular dependency that cannot be resolved by applicable official rules, use `BLOCKED_NORMATIVE` rather than inventing an evaluation order.

### 13.3 Existing Value Rules

`don_count.scope` accepts only:

- `FIELD`
- `COST_AREA`
- `DON_DECK`

`property.name` uses the queryable properties in §12.1.

`count_selection` and `sum_cost.selection` must reference a compatible earlier selection in the same resolution.

`count_area.area` is an AREA reference.

The count sources accepted by `scale` retain their original evaluation and referential constraints.

FCL permits only the explicitly defined `scale` multiplication. General arithmetic, addition expressions, subtraction expressions, division, arbitrary formulas, variables, functions, and arbitrary ID access remain unsupported.

## 14. References

### 14.1 Areas and States

Areas:

- `DECK`
- `DON_DECK`
- `HAND`
- `TRASH`
- `LEADER_AREA`
- `CHARACTER_AREA`
- `STAGE_AREA`
- `COST_AREA`
- `LIFE_AREA`

`NONE` is not an area.

States:

- `ACTIVE`
- `RESTED`

Attached DON!! has neither state (CR 4-4-2).

Faces:

- `FACE_UP`
- `FACE_DOWN`

Positions:

- `TOP`
- `BOTTOM`
- `NORMAL`, only when authorized by the relevant procedure.

### 14.2 Reference Forms

References identify instances, players, areas, or earlier results.

| `kind`                   | Additional fields | Meaning                                                         |
| ------------------------ | ----------------- | --------------------------------------------------------------- |
| `SOURCE`                 | None              | Instance containing the activated ability                       |
| `EVENT_OBJECT`           | None              | Exact object involved in the current supported contextual event |
| `PLAYER`                 | `role`            | Player reference                                                |
| `AREA`                   | `player`, `name`  | Area belonging to a player                                      |
| `SELECTED_OBJECT`        | `selection`       | Earlier singular selection                                      |
| `SELECTED_SET`           | `selection`       | Earlier set-producing selection or REMAINDER                    |
| `TEXT_DESIGNATED_OBJECT` | `designation`     | Single instance designated by text/rules                        |
| `ATTACKER`               | None              | Attacker in battle context                                      |
| `ATTACK_TARGET`          | None              | Target in battle context                                        |
| `BATTLE`                 | None              | Engine battle context                                           |
| `TOP_CARD`               | `player`, `area`  | Current top card of an area                                     |
| `CANDIDATE`              | None              | Object under filter evaluation                                  |

`PLAYER`:

```
{ kind = "PLAYER", role = "YOU" }
```

`AREA`:

```
{ kind = "AREA", player = { kind = "PLAYER", role = "YOU" }, name = "HAND" }
```

### 14.3 Player Roles

- `YOU`
- `OPPONENT`
- `TURN_PLAYER`
- `NON_TURN_PLAYER`
- `CONTEXT_PLAYER`
- `OWNER`
- `CONTROLLER`
- `ACTIVATOR`
- `CHOOSER`
- `EXECUTOR`
- `AFFECTED`

`YOU` is not a universal substitute for owner, controller, Turn Player, or activator.

`OWNER` and `CONTROLLER` are relative to the contextual object. For operations returning cards to their owners' Hands, resolve the owner separately for each selected instance.

For `ON_OPPONENT_CHARACTER_KO`, `YOU` and `OPPONENT` are determined using the controller of the source ability at event occurrence. The engine retains this contextual relationship for the pending event.

`TEXT_DESIGNATED_OBJECT.designation` must reproduce an exact expression in `abilities.source_text` and identify a single instance.

### 14.4 Selection and Candidate References

`SELECTED_OBJECT` must reference an earlier singular `SELECT`.

`SELECTED_SET` must reference an earlier set-producing `SELECT` or `REMAINDER`.

All result IDs are local to one effect and must reference earlier compatible results. No global variable, implicit last-card reference, or arbitrary instance-ID access exists.

`CANDIDATE` is permitted only in:

- `PROPERTY_COMPARE.subject` within selection filters, continuous scope filters, `target_scope.filter`, `allowed_targets.filter`, `OBJECT_COUNT.filter`, and inline payment filters.
- `COLOR_SET_COMPARE.subject` within selection filters.

`ATTACKER` is permitted in `PREVENT_BATTLE_KO.attacker_condition`.

`ATTACK_TARGET` and `BATTLE` are valid only in explicitly authorized structures.

`EVENT_OBJECT` is permitted only in the effect associated with `ON_OPPONENT_CHARACTER_KO`, wherever a compatible singular reference is accepted. It must identify the original triggering Character, not a substitute or a newly selected object.

`OWNER`/`CONTROLLER` must refer to the contextual card object, not automatically the ability owner.

When returning selected cards to owner Hands, resolve each destination separately.

## 15. Quantities, Selections, and Choices

### 15.1 Quantity

Supported forms:

```
{ mode = "FIXED", value = { literal = 2 } }
```

```
{ mode = "UP_TO", maximum = { literal = 2 } }
```

```
{ mode = "DYNAMIC", value = { count_area = { area = { kind = "AREA", player = { kind = "PLAYER", role = "YOU" }, name = "TRASH" } } } }
```

Rules:

- `FIXED` requires a positive quantity.
- `UP_TO` requires a non-negative maximum and allows zero when text/CR permits.
- `DYNAMIC` uses only §13 dynamic values and cannot produce negative quantities.

There is no `PROCEDURE` quantity mode. `DRAW` with `UP_TO` follows the official procedure of choosing whether to continue before each draw.

Fixed, `UP_TO`, and dynamic quantities are not interchangeable.

For payments, quantity choices occur during payment processing before resolution.

### 15.2 Selection

A `SELECT` step declares:

- `id`, unique within the effect.
- `object`: `CARD` or `DON`.
- `areas`: one or more AREA references.
- `filter`, when specified.
- `quantity`.
- `chooser`.
- `information`.
- `information_step`, when based on LOOK/REVEAL.
- `order_chooser`, when authorized by text/CR.

`information` accepts:

- `PUBLIC_INFORMATION`
- `OWNER_PRIVATE_INFORMATION`
- `LOOKED_BY_CHOOSER`
- `REVEALED_TO_CHOOSER`
- `FACE_DOWN_UNVERIFIABLE`

`information_step` is mandatory only for `LOOKED_BY_CHOOSER` and `REVEALED_TO_CHOOSER`.

For `LOOKED_BY_CHOOSER`, reference an earlier LOOK by the same chooser; candidates must remain in the examined area.

For `REVEALED_TO_CHOOSER`, reference an earlier REVEAL that included the chooser or was public; candidates must remain in the indicated areas.

`OWNER_PRIVATE_INFORMATION` requires the chooser to own the secret area.

`FACE_DOWN_UNVERIFIABLE` does not bypass CR restrictions.

Selection does not move, reveal, K.O., discard, or otherwise change an object.

Singular cardinality produces `SELECTED_OBJECT`; plural or set-producing selection produces `SELECTED_SET`.

`order_chooser` appears only when the text/CR assigns that choice. Simultaneous movements must not be rewritten as arbitrary sequential movements.

### 15.3 LOOK Remainder

`REMAINDER` identifies cards from an earlier LOOK result that remain after compatible selections are excluded.

It does not perform selection, revelation, movement, or reordering.

Fields:

| Field       | Type   | Requirement                    |
| ----------- | ------ | ------------------------------ |
| `id`        | string | Required, unique within effect |
| `kind`      | enum   | `REMAINDER`                    |
| `look_step` | string | Earlier LOOK step              |
| `excluding` | array  | Optional earlier SELECT IDs    |

Example:

```
{ id = "R1", kind = "REMAINDER", look_step = "S1", excluding = ["S2"] }
```

The remainder contains only cards that:

1. Belonged to the referenced LOOK result.
2. Remain in the original examined area when REMAINDER is evaluated.
3. Are not included in the excluded selections.

Each `excluding` ID must identify an earlier SELECT whose `information_step` points to the same LOOK and whose candidates are restricted to that result.

If omitted, no previous selection is explicitly excluded.

Cards moved out of the examined area are excluded even without an explicit exclusion ID.

A selected card listed in `excluding` remains excluded even if it has not moved.

Remainder order is not an instruction for sequential movement. Subsequent positioning must obey simultaneous-processing and ordering rules.

`REMAINDER` can be referenced as `SELECTED_SET` and by operations accepting set selections.

It does not grant access to cards outside the original LOOK result.

A remainder generated by a mechanism other than LOOK is unsupported unless another explicit construct covers it.

## 16. Replacement Effects

Every `REPLACEMENT` declaration requires `[abilities.effects.replacement]`.

`target` and `application` are descendant tables. `costs` and `procedure` are ordered arrays.

### 16.1 Structure

```
[[abilities.effects]]
id = "E1"
category = "REPLACEMENT"

[abilities.effects.replacement]
point = "BEFORE_PROCESSING"
replaced_part = "ACTIVATION_COST_PAYMENT"
costs = []

[abilities.effects.replacement.target]
procedure = "ACTIVATION_COST_PAYMENT"
rule_ref = "CR-8-3-1-7"
subject = { kind = "SOURCE" }

[abilities.effects.replacement.application]
chooser = { kind = "PLAYER", role = "YOU" }
```

Fields:

- `target`: original procedure, normative reference, and context.
- `point`: `BEFORE_PROCESSING` or `DURING_PROCESSING`.
- `replaced_part`: exact part replaced.
- `available_when`: optional additional condition.
- `duration`: optional availability duration.
- `application`: application decision.
- `costs`: ordered payment array; empty when none.
- `procedure`: ordered replacement sequence.

`target` requires `procedure`, `rule_ref`, and `subject`.

`rule_ref` must reference an existing normative source.

`subject` identifies the processing being replaced.

The engine retains the originating card, ability, effect, and source owner.

### 16.2 Target Catalog

`target.procedure` accepts:

- `ACTIVATION_COST_PAYMENT` — CR 8-3-1-7.
- `LEADER_DAMAGE_PROCESSING` — CR 4-6 and/or 7-1-4.
- `CHARACTER_KO_PROCESSING` — CR 10-2-1.
- `EFFECT_REMOVAL_FROM_FIELD` — removal from the Field by an effect, only when text expresses this class.
- `ORDINARY_CARD_MOVEMENT` — ordinary area movement excluding K.O., damage, draw, Trigger, play, and DON!! −X.

`replaced_part` accepts:

- `ACTIVATION_COST_PAYMENT`
- `LEADER_DAMAGE`
- `LIFE_TO_HAND_DURING_DAMAGE`
- `CHARACTER_KO`
- `FIELD_REMOVAL_BY_EFFECT`
- `ORDINARY_CARD_MOVEMENT`

Compatibility:

| Target procedure            | Permitted replaced part                       |
| --------------------------- | --------------------------------------------- |
| `ACTIVATION_COST_PAYMENT`   | `ACTIVATION_COST_PAYMENT`                     |
| `LEADER_DAMAGE_PROCESSING`  | `LEADER_DAMAGE`, `LIFE_TO_HAND_DURING_DAMAGE` |
| `CHARACTER_KO_PROCESSING`   | `CHARACTER_KO`                                |
| `EFFECT_REMOVAL_FROM_FIELD` | `FIELD_REMOVAL_BY_EFFECT`                     |
| `ORDINARY_CARD_MOVEMENT`    | `ORDINARY_CARD_MOVEMENT`                      |

These values are not interchangeable.

`EFFECT_REMOVAL_FROM_FIELD` does not mean arbitrary area changes.

### 16.3 Application

`application` contains exactly `chooser`, a PLAYER reference.

The designated player decides whether to apply the replacement.

Every available replacement may be declined. If declined, its procedure does not resolve and original processing continues.

This choice is different from an optional resolution action, optional cost, or `UP_TO` selection.

### 16.4 Engine Procedures

The engine applies CR 8-1-3-4:

1. An available replacement may be declined.
2. Priority follows the related card's replacement, then Turn Player replacements in chosen order, then Non-Turn Player replacements in chosen order.
3. A replacement cannot be repeated on the same individual processing already replaced.
4. It substitutes the entire identified part, including during another effect's resolution; remaining text continues in CR order.
5. An impossible replacement procedure cannot be applied.
6. A replacement may enable another, subject to priority and non-repetition.
7. The replaced processing is treated as an effect activated by the replacement's owner.

Priority, chaining, and non-repetition belong to the engine.

A new independent event may offer the same replacement again.

### 16.5 Activation Cost Replacement

For `ACTIVATION_COST_PAYMENT`:

- The replaced Activation Cost must be explicitly identified.
- The replacement procedure occurs instead of the original payment.
- If the replacement payment is not made as specified, the text after `:` does not resolve.
- Original cost, replacement procedure, and subsequent resolution remain distinct.

Never move the payment into resolution.

### 16.6 Collective Costs

There is no generic single-payment rule for multiple replacement effects.

A simultaneous event affecting multiple objects, multiple individual removals, and one textual cost are distinct situations.

Preserve the payment unit established by the card and normative sources.

When unresolved, use `BLOCKED_NORMATIVE`. Never assume a single payment regardless of quantity.

### 16.7 Boundary

Replacement cannot be omitted, converted into `PERMANENT`, represented as a `REPLACE` resolution operation, or reduced to generic movement.

Unsupported cases use `BLOCKED_CAPABILITY` or `BLOCKED_NORMATIVE`, according to the cause.

## 17. Costs and Payments

`activation_costs` and `replacement.costs` are ordered arrays.

Each payment is an inline table discriminated by `kind`. Undefined fields are rejected.

### 17.1 Optionality

All payments accept `optional`:

- Omitted or `false`: mandatory.
- `true`: payer may decline the entire payment.

Payment-level optionality is distinct from quantity.

If an optional payment is declined, the effect following `:` is not activated (CR 8-3-1-4).

If accepted, the payment proceeds according to its quantity and selection rules.

An accepted `UP_TO` payment may still permit a zero quantity when text/CR allows. Choosing zero is different from declining the payment entirely.

### 17.2 Payment Catalog

| `kind`                           | Required fields                                  | Semantics                                                                           |
| -------------------------------- | ------------------------------------------------ | ----------------------------------------------------------------------------------- |
| `REST_DON`                       | `payer`, `selection`                             | Select and rest specified active DON!! in payer's Cost Area                         |
| `DON_MINUS`                      | `payer`, `selection`                             | Return specified DON!! from payer's Leader, Character, and Cost Areas to DON!! Deck |
| `REST_CARD`                      | `payer`, exactly one of `selection` or `subject` | Rest selected card(s) or exact SOURCE                                               |
| `TRASH_HAND_CARD`                | `payer`, `selection`                             | Trash one selected card from payer's Hand                                           |
| `RETURN_CHARACTER_TO_OWNER_HAND` | `payer`, `selection`                             | Return one selected Character from payer's Character Area to its owner's Hand       |
| `ADD_LIFE_TO_HAND`               | `payer`, `quantity`                              | Move top Life card(s) to payer's Hand                                               |
| `PLACE_HAND_CARD_BOTTOM_DECK`    | `payer`, `selection`                             | Place one selected Hand card at bottom of payer's Deck                              |

All payment types may also contain `optional`.

For payment-selection types, `selection` is a payment-selection table, not a prior resolution step ID.

Payment-selection fields:

- `object`: `CARD` or `DON`.
- `areas`: non-empty AREA_REF array.
- `quantity`: §15.1.
- `chooser`: PLAYER reference.
- `information`: §15.2 vocabulary.
- `filter`: optional Condition using `CANDIDATE`.

Specific requirements:

REST_DON

- Areas contain only payer's Cost Area.
- `object = "DON"`.
- Filter restricts `STATE = ACTIVE`.

DON_MINUS

\- Areas identify the payer's permitted Leader, Character, and Cost Areas.

REST_CARD using selection

- `object = "CARD"`.
- Filter restricts `STATE = ACTIVE`.
- In versions 1.0.0 through 1.4.0, quantity must be `FIXED 1`.
- In version 1.5.0 and later, quantity must be `FIXED` with any positive integer.
- `UP_TO` and `DYNAMIC` are not supported for this payment-selection form.
- Selected objects must be distinct eligible card instances.

REST_CARD using subject

- Uses only the direct SOURCE form in §17.7.
- It cannot be combined with `selection`.

TRASH_HAND_CARD

- Area must be payer's Hand.
- Quantity must be `FIXED 1`.

RETURN_CHARACTER_TO_OWNER_HAND

- Area must be only payer's Character Area.
- Object must be `CARD`.
- Quantity must be `FIXED 1`.
- Chooser must be payer.
- Information must be `PUBLIC_INFORMATION`.
- Filter must restrict `CATEGORY = CHARACTER`.

PLACE_HAND_CARD_BOTTOM_DECK

- Area must be payer's Hand.
- Object must be `CARD`.
- Quantity must be `FIXED 1`.
- Chooser must be payer.
- Information must be `OWNER_PRIVATE_INFORMATION`.

The chooser is the player making the selection. Unknown chooser roles require blocking.

### 17.3 Multiple-Card REST_CARD Payment — FCL 1.5.0

Version 1.5.0 extends `REST_CARD.selection` to allow a fixed quantity greater than one.

Example:

```
{
  kind = "REST_CARD",
  payer = { kind = "PLAYER", role = "YOU" },
  selection = {
    object = "CARD",
    areas = [
      { kind = "AREA", player = { kind = "PLAYER", role = "YOU" }, name = "CHARACTER_AREA" }
    ],
    filter = { all = [
      { test = "PROPERTY_COMPARE", subject = { kind = "CANDIDATE" }, property = "CATEGORY", operator = "EQ", value = { literal = "CHARACTER" } },
      { test = "PROPERTY_COMPARE", subject = { kind = "CANDIDATE" }, property = "STATE", operator = "EQ", value = { literal = "ACTIVE" } }
    ] },
    quantity = { mode = "FIXED", value = { literal = 2 } },
    chooser = { kind = "PLAYER", role = "YOU" },
    information = "PUBLIC_INFORMATION"
  },
  optional = true
}
```

Semantics:

1. One payment represents the entire required group.
2. The payer chooses exactly the fixed number of distinct eligible card instances.
3. Every selected card must satisfy the filter and be active when its payment is processed.
4. The selected cards are rested as part of that payment, following the normal engine/CR procedure.
5. The number is not an `UP_TO` quantity. `FIXED 2` requires exactly two cards when the optional payment is accepted.
6. The payment is not equivalent to two independent optional payments.
7. Do not encode the same group as multiple `REST_CARD` entries merely to represent its quantity.
8. `subject = SOURCE` remains a singular direct payment and does not support multiple objects.

If the accepted payment cannot be made before activation, the associated effect cannot activate.

If payment becomes impossible during processing after activation, pay as much as possible according to the CR, but do not resolve the text after `:`.

Declining an optional payment is a separate decision from selecting its required quantity.

This extension does not establish generic multi-object costs for other payment types.

### 17.4 Life-to-Hand Payment

`ADD_LIFE_TO_HAND` contains:

- `kind`
- `payer`
- `quantity`
- optional `optional`

It does not contain `selection`, `areas`, `chooser`, or `information`.

Under CR 3-10-2:

- Movement from Life uses the current top Life card.
- The payer cannot choose another card.
- The payer cannot inspect or reorder face-down Life.
- Multiple cards move individually, each time using the new top card.
- This is not damage.
- This does not offer Trigger.
- Trigger is offered only during official damage processing.

Use `UP_TO` when the text permits up to a quantity.

Example:

```
{ kind = "ADD_LIFE_TO_HAND", payer = { kind = "PLAYER", role = "YOU" }, quantity = { mode = "UP_TO", maximum = { literal = 1 } }, optional = true }
```

The optional payment may be declined entirely. If accepted, any permitted `UP_TO` quantity choice is separate.

### 17.5 Hand-to-Bottom-of-Deck Payment

`PLACE_HAND_CARD_BOTTOM_DECK` moves exactly one selected card from payer's Hand to the bottom of that payer's Deck.

It uses a payment-selection table with exactly one CARD from HAND, payer as chooser, and `OWNER_PRIVATE_INFORMATION`.

Example:

```
{ kind = "PLACE_HAND_CARD_BOTTOM_DECK", payer = { kind = "PLAYER", role = "YOU" }, selection = { object = "CARD", areas = [{ kind = "AREA", player = { kind = "PLAYER", role = "YOU" }, name = "HAND" }], quantity = { mode = "FIXED", value = { literal = 1 } }, chooser = { kind = "PLAYER", role = "YOU" }, information = "OWNER_PRIVATE_INFORMATION" }, optional = true }
```

The card moves during payment processing, before resolution.

This is not `POSITION_CARD`, `MOVE_CARD`, or a later operation.

Do not reveal the card to the opponent unless another rule/effect requires it.

Do not shuffle the Deck.

If the payer cannot select an eligible card, the payment cannot be made.

### 17.6 Character-to-Owner-Hand Payment

`RETURN_CHARACTER_TO_OWNER_HAND` represents an optional or mandatory payment returning exactly one selected Character from the payer's Character Area to its owner's Hand.

Its payment selection must contain:

- `object = "CARD"`.
- Only payer's `CHARACTER_AREA`.
- `quantity = FIXED 1`.
- Payer as chooser.
- `PUBLIC_INFORMATION`.
- A filter restricting `CATEGORY = CHARACTER`.

Example:

```
{ kind = "RETURN_CHARACTER_TO_OWNER_HAND", payer = { kind = "PLAYER", role = "YOU" }, selection = { object = "CARD", areas = [{ kind = "AREA", player = { kind = "PLAYER", role = "YOU" }, name = "CHARACTER_AREA" }], filter = { test = "PROPERTY_COMPARE", subject = { kind = "CANDIDATE" }, property = "CATEGORY", operator = "EQ", value = { literal = "CHARACTER" } }, quantity = { mode = "FIXED", value = { literal = 1 } }, chooser = { kind = "PLAYER", role = "YOU" }, information = "PUBLIC_INFORMATION" }, optional = true }
```

The selected Character goes to the Hand of its owner, not automatically to the payer's Hand.

Payer and owner may be different players.

This is ordinary movement during payment processing, not K.O., Trash, or resolution movement. It does not create `ON_KO`.

Use `optional = true` for "You may return 1..."; retain `FIXED 1` for the accepted payment. Never replace it with `UP_TO 1`.

If the payer cannot select one eligible Character, the payment cannot be made.

The returned card can be used by subsequent resolution only if it actually belongs to the payer's Hand and satisfies the later selection.

### 17.7 Direct Source-Instance REST_CARD Payment

The direct form is:

```
{ kind = "REST_CARD", payer = { kind = "PLAYER", role = "YOU" }, subject = { kind = "SOURCE" } }
```

When using `subject`:

- `selection` is forbidden.
- `subject` must be exactly SOURCE.
- SOURCE must be legally restable.
- SOURCE must be controlled by payer.
- SOURCE must be active when payment is processed.
- The exact SOURCE instance is rested.
- No substitute card may be chosen.

The direct form does not use `object`, `areas`, `quantity`, `chooser`, `information`, or `filter`.

It is not a general arbitrary-instance reference.

If SOURCE cannot be rested, payment cannot be made.

### 17.8 General Payment Rules

- DON!! is attached only through `GIVE_DON`.
- Payments do not transfer DON!! already attached to another card.
- `REST_DON` uses active DON!!.
- `GIVE_DON.source_state = "RESTED"` requires explicit text/FAQ authorization. EB05-057 is limited to its own case.
- Attached DON!! has neither active nor rested state.
- Card-play Cost is distinct from `activation_costs`.
- Payments occur before resolution and preserve their specified order.
- Costs added by effects follow the normative ordering of the effect that added them.
- If the entire cost is impossible before activation, the effect cannot activate.
- If payment becomes impossible after activation during processing, pay as much as possible and do not resolve the text after `:` (CR 8-3-1-3-1).
- Declining optional payment prevents activation of the associated effect.
- Alternative costs are not mode choices. Never assume alternative and additional costs may be combined.
- Activation Cost replacements use §16.
- Undefined payment types remain `BLOCKED_CAPABILITY`.

## 18. Resolution

`resolution` is an ordered array. Every step has a unique `id` within the effect, including nested branches, and a `kind`.

| Kind           | Required fields besides `id` and `kind`                 | Optional fields                               |
| -------------- | ------------------------------------------------------- | --------------------------------------------- |
| `SELECT`       | `object`, `areas`, `quantity`, `chooser`, `information` | `filter`, `information_step`, `order_chooser` |
| `REMAINDER`    | `look_step`                                             | `excluding`                                   |
| `OPERATION`    | `operation` and §19 required parameters                 | §19 optional parameters                       |
| `IF`           | `condition`, `steps`                                    | None                                          |
| `THEN`         | `steps`                                                 | None                                          |
| `IF_YOU_DO`    | `depends_on`, `steps`                                   | None                                          |
| `OPTIONAL`     | `chooser`, `steps`                                      | None                                          |
| `CHOOSE_MODE`  | `chooser`, `options`                                    | None                                          |
| `SIMULTANEOUS` | `steps`                                                 | None                                          |
| `REPEAT`       | `quantity`, `steps`                                     | None                                          |

`SELECT` follows §15.2. `REMAINDER` follows §15.3.

Operations use references valid at execution time.

### 18.1 IF

The Condition is evaluated when the step is reached.

Only steps inside the branch depend on it.

### 18.2 THEN

`THEN` contains the steps belonging to the subsequent textual clause.

It marks a clause boundary but does not create a universal dependency on complete success of the previous step.

When the text explicitly introduces a subsequent clause with `Then`, preserve the boundary using `THEN`, unless another more specific rule/construct applies.

It is not interchangeable with `IF`, `IF_YOU_DO`, `OPTIONAL`, or `SIMULTANEOUS`.

### 18.3 IF_YOU_DO

`depends_on` identifies a previous step.

Its success semantics follow the text and CR. Do not assume success means all, at least one, or atomic completion without normative support.

### 18.4 OPTIONAL

Represents a player-declinable action during resolution.

It is not an optional cost, `UP_TO` selection, mode choice, or declined Replacement Effect.

### 18.5 CHOOSE_MODE

`options` is an ordered non-empty array. Each option has a unique `id` and its own ordered `steps`.

Only instructions belonging to the selected mode appear in that branch.

### 18.6 SIMULTANEOUS

Groups steps that must occur simultaneously.

Never rewrite the group as an arbitrary sequence.

Player choice order and card ordering remain subject to the CR.

### 18.7 REPEAT

Only finite repetition is allowed:

- Fixed quantity.
- Supported dynamic quantity.
- `UP_TO`, when authorized.

No free loops, arbitrary stopping conditions, or recursion.

The engine handles generated events, Trigger, `rule processing`, and general loops.

## 19. Supported Operations

Only the following identifiers are permitted.

| Operation               | Semantics                                                                               |
| ----------------------- | --------------------------------------------------------------------------------------- |
| `DRAW`                  | Draw from Deck using official procedure; `UP_TO` permits stopping before each draw      |
| `TRASH_HAND`            | Move selected Hand card to Trash; not K.O.                                              |
| `TRASH_DECK`            | Trash selected Deck card under authorized selection/information rules; not draw         |
| `KO_CHARACTER`          | Apply official Character K.O. procedure and event                                       |
| `MOVE_CARD`             | Ordinary area movement, not K.O., damage, draw, Trigger, playing, or DON!! −X           |
| `REST`                  | Active to rested; already rested remains rested                                         |
| `SET_ACTIVE`            | Rested to active; already active remains active                                         |
| `GIVE_DON`              | Attach DON!! from Cost Area to eligible Leader/Character                                |
| `ADD_DON`               | Move DON!! from DON!! Deck into Cost Area with specified state                          |
| `DON_MINUS`             | Official DON!! −X procedure                                                             |
| `PLAY_CARD`             | Official procedure for playing by effect; not ordinary Field movement                   |
| `REVEAL`                | Reveal subject/selection/LOOK result without moving it                                  |
| `LOOK`                  | Inspect top cards of an ordered area without movement, revelation to others, or reorder |
| `SHUFFLE_DECK`          | Official Deck shuffle                                                                   |
| `MODIFY_POWER`          | Modify effective Power, not base Power                                                  |
| `SET_BASE_POWER`        | Set base Power under normative precedence                                               |
| `MODIFY_COST`           | Modify effective Cost                                                                   |
| `SET_BASE_COST`         | Set base Cost                                                                           |
| `MODIFY_CHARACTERISTIC` | Modify/grant only NAME, TYPE, or ATTRIBUTE                                              |
| `GRANT_KEYWORD_EFFECT`  | Grant recognized Keyword Effect/Keyword with duration                                   |
| `DEAL_LEADER_DAMAGE`    | Official Leader damage procedure, including Life, Trigger, replacements, and defeat     |
| `MOVE_LIFE_CARD`        | Ordinary Life movement, not damage and no Trigger by itself                             |
| `POSITION_CARD`         | Position a card/set at an authorized stack position                                     |
| `RETURN_CARD`           | Return to Hand, Deck, or Trash, preserving owner/destination                            |
| `END_GAME`              | Grant victory/defeat only when text/rules authorize it                                  |

Attack-target permission, attack-target restriction, and battle K.O. prevention are continuous consequences, not resolution operations.

### 19.1 Operation Parameters

`subject` is a singular §14 reference.

`selection` is an earlier compatible `SELECT` or `REMAINDER` result.

Where the matrix says `subject` or `selection`, exactly one must appear.

Area, player, quantity, and duration use their defined structures. No additional fields are permitted.

| Operation               | Required fields                                                       | Optional fields                              |
| ----------------------- | --------------------------------------------------------------------- | -------------------------------------------- |
| `DRAW`                  | `player`, `quantity`                                                  | None                                         |
| `TRASH_HAND`            | `player`, `selection`                                                 | None                                         |
| `TRASH_DECK`            | `player`, `selection`                                                 | None                                         |
| `KO_CHARACTER`          | `subject` or `selection`                                              | None                                         |
| `MOVE_CARD`             | `subject` or `selection`, `source_area`, `destination_area`           | `face`, `state`, `position`, `order_chooser` |
| `REST` / `SET_ACTIVE`   | `subject` or `selection`                                              | None                                         |
| `GIVE_DON`              | recipient `subject` or `selection`, `quantity`, `source_state`        | None                                         |
| `ADD_DON`               | `player`, `quantity`, `arrival_state`                                 | None                                         |
| `DON_MINUS`             | `player`, `quantity`                                                  | None                                         |
| `PLAY_CARD`             | card `subject` or `selection`, `source_area`, `player`                | `state`                                      |
| `REVEAL`                | Exactly one of `subject`, `selection`, `look_step`                    | `recipients`                                 |
| `LOOK`                  | `area`, `player`, `quantity`                                          | None                                         |
| `SHUFFLE_DECK`          | `player`                                                              | None                                         |
| `MODIFY_POWER`          | `subject` or `selection`, `operator`, `value`, `duration`             | None                                         |
| `SET_BASE_POWER`        | `subject` or `selection`, `value`, `duration`                         | None                                         |
| `MODIFY_COST`           | `subject` or `selection`, `operator`, `value`, `duration`             | None                                         |
| `SET_BASE_COST`         | `subject` or `selection`, `value`, `duration`                         | None                                         |
| `MODIFY_CHARACTERISTIC` | `subject` or `selection`, `property`, `operator`, `value`, `duration` | None                                         |
| `GRANT_KEYWORD_EFFECT`  | `subject` or `selection`, `keyword_effect`, `duration`                | None                                         |
| `DEAL_LEADER_DAMAGE`    | `player`, `quantity`                                                  | None                                         |
| `MOVE_LIFE_CARD`        | `subject` or `selection`, `source_area`, `destination_area`           | `position`, `face`, `order_chooser`          |
| `POSITION_CARD`         | `subject` or `selection`, `destination_area`, `position`              | `order_chooser`                              |
| `RETURN_CARD`           | `subject` or `selection`, `destination_area`                          | `position`                                   |
| `END_GAME`              | `player`, `outcome` (`WIN` or `LOSE`)                                 | None                                         |

`source_area`, `destination_area`, and `area` are AREA_REF, except `AREA_COUNT.area`, which uses an area name.

`recipients` is a non-empty array of PLAYER references. Omission means public information.

`LOOK` has no `recipients`; information remains private to the designated player.

`LOOK` creates a temporary identified set, without changing order or area.

`REVEAL.look_step` identifies an earlier LOOK in the same resolution. Otherwise, `REVEAL` uses `subject` or `selection`.

`POSITION_CARD` referencing a `REMAINDER` positions the entire set as a group.

When the text grants ordering choice, `order_chooser` identifies the player. The engine preserves the chosen final order; TOML array order does not determine it.

For multiple cards placed at the bottom of a Deck in a chosen order, `order_chooser` is required.

`face`: `FACE_UP` or `FACE_DOWN`.

`state`, `source_state`, `arrival_state`: `ACTIVE` or `RESTED`.

`position`: `TOP`, `BOTTOM`, or authorized `NORMAL`.

`GIVE_DON.source_state = "ACTIVE"` is normal. `RESTED` requires explicit official authorization, such as the specific EB05-057 FAQ case.

Attached DON!! has neither active nor rested state.

`GIVE_DON` uses Cost Area DON!! only. `ADD_DON` uses DON!! Deck.

`MODIFY_POWER.operator` and `MODIFY_COST.operator`: `ADD` or `SUBTRACT`.

`MODIFY_CHARACTERISTIC.property`: `NAME`, `TYPE`, or `ATTRIBUTE`.

Its `operator` accepts `SET` for replacing values, or `ADD`/`REMOVE` for TYPE/ATTRIBUTE.

`NAME` accepts only `SET`. NAME/TYPE values are strings; ATTRIBUTE is a §6 enum.

Changing color/category is unsupported.

When a selected card moves to an area whose player is `OWNER`, resolve the destination separately for each card.

### 19.2 Semantics That Must Not Be Collapsed

- K.O. generates the official K.O. procedure; ordinary Trash does not create `ON_KO` or `ON_OPPONENT_CHARACTER_KO`.
- `MOVE_LIFE_CARD` is neither damage nor automatic Trigger.
- `ADD_LIFE_TO_HAND` is a payment, not damage or resolution movement.
- `PLACE_HAND_CARD_BOTTOM_DECK` is a payment, not resolution movement or shuffle.
- `RETURN_CHARACTER_TO_OWNER_HAND` is a payment, not `RETURN_CARD`.
- `REST_CARD.subject = SOURCE` rests the exact source instance.
- `PLAY_CARD` performs official play legality, cost/waiver, destination, and entry timing.
- DON!! in DON!! Deck, Cost Area, and attached to cards are distinct.
- `MODIFY_POWER` does not modify base Power.
- Setting Power to zero follows the CR 4-12 procedure; FCL has no specific operation for it, so block it.
- Power may become negative without moving a card.
- Negative effective Cost is zero outside calculations, but may participate in calculations according to CR.
- Generic effect negation, arbitrary ability granting, generic action prohibition, and arbitrary procedure modification remain unsupported.
- `BLOCKER` follows its own engine procedure, not generic redirection.
- `REMAINDER` does not create a new LOOK, change visibility, or grant information.
- Positioning a remainder set preserves group ordering.
- `GRANT_ATTACK_TARGET_PERMISSION` only adds its explicitly specified targets.
- `RESTRICT_ATTACK_TARGETS` only narrows the legal target set for the specified player's attacks.
- `PREVENT_BATTLE_KO` only prevents the specified battle K.O. result.
- None of these continuous constructs is interchangeable with another.

## 20. Continuous Effects — `continuous`

`continuous` represents a continuing consequence, never a sequence of actions.

Normally exactly one of `subject` or `scope` identifies the object affected.

Both are inside `continuous`, never sibling fields of the effect.

Permitted kinds:

| Kind                                             | Required additional fields                                | Optional fields      |
| ------------------------------------------------ | --------------------------------------------------------- | -------------------- |
| `MODIFY_POWER` / `MODIFY_COST`                   | `operator`, `value`, Auto/Activate `duration`             | None                 |
| `SET_BASE_POWER` / `SET_BASE_COST`               | `value`, Auto/Activate `duration`                         | None                 |
| `MODIFY_CHARACTERISTIC`                          | `property`, `operator`, `value`, Auto/Activate `duration` | None                 |
| `GRANT_KEYWORD_EFFECT` / `REMOVE_KEYWORD_EFFECT` | `keyword_effect`, Auto/Activate `duration`                | None                 |
| `GRANT_ATTACK_TARGET_PERMISSION`                 | `target_scope`, Auto/Activate `duration`                  | None                 |
| `RESTRICT_ATTACK_TARGETS`                        | `attacking_player`, `allowed_targets`                     | None                 |
| `PREVENT_BATTLE_KO`                              | Auto/Activate `duration`                                  | `attacker_condition` |

`scope`:

```
{ object = "CARD", areas = [AREA_REF], player = PLAYER_REF, filter = CONDITION }
```

`player` and `filter` are optional. Without `player`, scope includes all players.

Scope filters use `CANDIDATE`.

`subject` is a singular reference.

`property` accepts `NAME`, `TYPE`, `ATTRIBUTE`.

`keyword_effect` accepts only §10.1 values.

In AUTO/ACTIVATE, every continuous effect requires `duration`.

PERMANENT uses optional `valid_when` and does not include `duration` inside `continuous`.

Operators for characteristic/Power/Cost modification accept the operations defined in §19.

`MODIFY_POWER` can yield negative Power. `MODIFY_COST` follows CR handling of negative Cost.

`SET_BASE_POWER` and `SET_BASE_COST` modify base values, with normative precedence.

### 20.1 GRANT_ATTACK_TARGET_PERMISSION

Grants additional attack-target eligibility to the card identified by `continuous.subject` or each card in `continuous.scope`.

It is permission, not an attack instruction.

Example:

```
[abilities.effects.continuous]
kind = "GRANT_ATTACK_TARGET_PERMISSION"
subject = { kind = "SOURCE" }
target_scope = {
  object = "CARD",
  areas = [{ kind = "AREA", player = { kind = "PLAYER", role = "OPPONENT" }, name = "CHARACTER_AREA" }],
  states = ["ACTIVE"]
}
```

`target_scope` is an inline table:

| Field    | Type              | Requirement                                                |
| -------- | ----------------- | ---------------------------------------------------------- |
| `object` | enum              | Required; only `CARD`                                      |
| `areas`  | array of AREA_REF | Required, non-empty                                        |
| `states` | array             | Optional; non-empty when present; `ACTIVE` and/or `RESTED` |
| `filter` | Condition         | Optional; uses `CANDIDATE`                                 |

Areas identify where additional targets can be found.

States restrict the target cards, when present, and are valid only for areas with those states.

Filter evaluates each candidate using its effective characteristics.

The engine combines these additional targets with the normally legal targets under the CR.

The source attacker must independently satisfy all attack-declaration rules.

The permission does not:

- Declare or execute an attack.
- Select an actual target.
- Grant `RUSH` or `RUSH_CHARACTER`.
- Allow an otherwise prohibited attack on the turn a Character is played.
- Override a separate restriction on the source attacker.
- Authorize targets outside `target_scope`.
- Change battle resolution or K.O.

"Can also attack your opponent's active Characters" uses opponent's `CHARACTER_AREA` and `states = ["ACTIVE"]`.

DON!! requirements are encoded by validity conditions, normally `valid_when` for PERMANENT, not attack operations.

The OP01-021 FAQ confirms additional target permission does not grant Rush or allow attacking on the turn the source Character was played.

### 20.2 PREVENT_BATTLE_KO

Prevents K.O. caused by a battle result for `continuous.subject` or the cards within `continuous.scope`, subject to optional `attacker_condition`.

Example:

```
[abilities.effects.continuous]
kind = "PREVENT_BATTLE_KO"
subject = { kind = "SOURCE" }
attacker_condition = { all = [
  { test = "PROPERTY_COMPARE", subject = { kind = "ATTACKER" }, property = "CATEGORY", operator = "EQ", value = { literal = "CHARACTER" } },
  { test = "PROPERTY_COMPARE", subject = { kind = "ATTACKER" }, property = "ATTRIBUTE", operator = "EQ", value = { literal = "STRIKE" } }
] }
```

Without `attacker_condition`, protection applies to battle K.O. regardless of attacker characteristics.

When present, it evaluates the actual attacker in the relevant battle, using effective characteristics.

For "cannot be K.O.'d in battle by Characters with Strike attribute", identify both CHARACTER and STRIKE.

It prevents only the specified battle K.O.

It does not:

- Prevent attack declaration.
- Cancel or redirect the attack.
- End or negate battle.
- Prevent Leader battle damage.
- Prevent K.O. by effects.
- Prevent ordinary Trash/movement.
- Negate effects.
- Grant unrelated immunity.
- Create an optional Replacement Effect.

DON!! conditions belong in `valid_when` for PERMANENT effects.

`PREVENT_BATTLE_KO` is not a Replacement Effect.

The OP01-024 FAQ establishes the distinction between battle K.O. and K.O. caused by effects.

### 20.3 RESTRICT_ATTACK_TARGETS — FCL 1.5.0

`RESTRICT_ATTACK_TARGETS` declares a narrow continuous restriction on the attack targets available to one player.

It is not a generic action-prohibition mechanism.

It is intended for text such as:

- "Your opponent cannot attack any card other than [specified Character]."
- "Your opponent can attack only the designated class of Characters."

Required structure:

- `subject`: exactly one SOURCE reference.
- `attacking_player`: PLAYER reference identifying whose attacks are restricted.
- `allowed_targets`: inline target-class definition.

`scope` is forbidden for this kind.

Example:

```
[abilities.effects.continuous]
kind = "RESTRICT_ATTACK_TARGETS"
subject = { kind = "SOURCE" }
attacking_player = { kind = "PLAYER", role = "OPPONENT" }
allowed_targets = {
  object = "CARD",
  areas = [{ kind = "AREA", player = { kind = "PLAYER", role = "YOU" }, name = "CHARACTER_AREA" }],
  filter = {
    test = "PROPERTY_COMPARE",
    subject = { kind = "CANDIDATE" },
    property = "NAME",
    operator = "EQ",
    value = { literal = "Eustass\"Captain\"Kid" }
  }
}
```

`allowed_targets` accepts:

| Field    | Type              | Requirement                                   |
| -------- | ----------------- | --------------------------------------------- |
| `object` | enum              | Required; only `CARD`                         |
| `areas`  | array of AREA_REF | Required; non-empty                           |
| `states` | array of enums    | Optional; non-empty; `ACTIVE` and/or `RESTED` |
| `filter` | Condition         | Optional; uses `CANDIDATE`                    |

Semantics:

1. The effect restricts attack declarations made by `attacking_player`.
2. The engine first determines targets otherwise legal under the CR and currently applicable attack permissions.
3. An attack target must additionally belong to `allowed_targets`.
4. The target must still satisfy every ordinary rule and other applicable requirement for an attack.
5. The construct never adds new target classes or makes normally illegal targets legal.
6. When no legal target remains, the affected player cannot declare an attack against a target outside the permitted set.
7. Multiple applicable `RESTRICT_ATTACK_TARGETS` effects combine as restrictions: a target must satisfy all of them. Their allowed sets are therefore intersected, not united.
8. Restrictions remain effective even when another effect grants additional attack-target permission.
9. Evaluation occurs when attack targets are being declared, under the current continuous-effect validity and rules.
10. The `subject` identifies the source instance establishing the restriction and its provenance; it is not itself the restricted attack target.

`allowed_targets.filter` evaluates each candidate card using `CANDIDATE` and effective characteristics.

When text specifies a card name, use `PROPERTY_COMPARE` with `property = "NAME"` and `operator = "EQ"`.

Do not use `TEXT_DESIGNATED_OBJECT` when the official effect allows any matching card instance rather than only one exact instance.

The official OP01-051 FAQ confirms that when multiple copies of the card establish the restriction, the opponent may attack any of the affected player's Characters with the specified card name.

The official OP11-010 FAQ confirms that an independent effect granting attacks against active Characters does not override the OP01-051 restriction.

`RESTRICT_ATTACK_TARGETS` does not:

- Prevent an attacker from existing or becoming eligible to attack.
- Declare, select, or execute an attack.
- Cancel an attack already declared.
- Redirect an attack after declaration.
- Change battle resolution.
- Modify Blocker or other battle procedures.
- Grant additional target permissions.
- Prohibit unrelated game actions.
- Introduce generic attack cancellation or arbitrary procedure modification.

The restriction applies only to target eligibility at the applicable declaration point. All other attack and battle procedures remain under the CR.

### 20.4 Continuous-Effect Structure and Boundaries

For ordinary continuous effects, exactly one of `subject` or `scope` is required.

- `subject` identifies one singular affected object.
- `scope` identifies a set of affected objects.
- `GRANT_ATTACK_TARGET_PERMISSION.target_scope` identifies additional attack targets.
- `RESTRICT_ATTACK_TARGETS.allowed_targets` identifies the permitted attack-target set under a restriction.
- `PREVENT_BATTLE_KO.attacker_condition` identifies qualifying battle attackers.

For `RESTRICT_ATTACK_TARGETS`, `subject` is mandatory and `scope` is forbidden.

Auto/Activate continuous effects require duration.

Permanent effects use `valid_when` and do not declare continuous duration.

The following are outside the profile:

- `GRANT_ABILITY`
- `NEGATE_EFFECT`
- Generic `PROHIBIT_ACTION`
- Generic `MODIFY_PROCEDURE`

Use `BLOCKED_CAPABILITY` when the actual text requires these unrestricted capabilities.

The three supported target/battle constructs are narrowly defined exceptions and cannot be repurposed outside their contracts.

Replacement Effects remain exclusively `category = "REPLACEMENT"`.

## 21. Supported Durations

`duration` uses an inline object with `kind` and required conditional fields.

Permitted kinds:

- `THIS_TURN`
- `UNTIL_END_OF_TURN`
- `UNTIL_END_OF_END_PHASE`
- `THIS_BATTLE`
- `UNTIL_START_OF_NEXT_TURN`, with `player`
- `WHILE_VALID`, with `condition`
- `PERMANENT`, only when text/rules establish that duration without ordinary temporal expiration

Do not normalize `THIS_TURN` and `UNTIL_END_OF_TURN`.

Expiration occurs at official engine checkpoints.

Do not invent counters or expiration events such as "next attack" or "next draw".

A continuous effect created by Auto/Activate does not automatically disappear because its source leaves the Field. Its validity and duration follow text/CR.

Permanent Effects follow their validity and permitted area exceptions.

The engine performs Permanent Effect stabilization in Turn Player, then Non-Turn Player order, repeating when the state changes (CR 8-1-3-3-5).

## 22. Usage Limits

`usage_limit` accepts only:

```
usage_limit = { kind = "ONCE_PER_TURN" }
```

Application is per card instance under CR 10-2-13.

Leaving an area and returning may create a new instance.

When activation reaches payment and payment becomes impossible during processing, `[Once Per Turn]` remains consumed for that turn (CR 10-2-13-5).

Reset, instance identity, and consumption timing belong to the engine.

Do not invent `ONCE_PER_BATTLE`, `ONCE_PER_GAME`, `ONCE_PER_PHASE`, or `ONCE_PER_ATTACK`.

## 23. Validation Rules

All validation layers are mandatory.

### 23.1 Syntactic

- Valid UTF-8 TOML 1.0.0.
- Parser errors terminate validation.

### 23.2 Structural

Validate:

- Authorized tables and keys only.
- Correct types and required/prohibited fields.
- Valid enumerations.
- Unique IDs.
- Correct category-specific fields.
- Ordered arrays wherever order matters.
- Operation/test/payment/step parameter compatibility.
- `editorial.extra` as the only extensible non-executable metadata area.
- Valid printing overrides and matching `art_id` values.
- Required `default_printing_id` when overrides exist.
- Boolean `optional`, only in payment tables.
- No nulls, duplicate keys, or unsupported TOML types.
- Correct nesting of continuous `subject`/`scope`.
- Exactly one of subject/scope for ordinary continuous kinds.
- `continuous.kind` field restrictions.
- Valid `GRANT_ATTACK_TARGET_PERMISSION.target_scope`.
- Valid `RESTRICT_ATTACK_TARGETS` structure, including SOURCE, attacking player, and allowed target class.
- Valid `PREVENT_BATTLE_KO.attacker_condition`.
- Kind-specific fields forbidden from other continuous-effect types.
- Exactly one `REST_CARD` form: `selection` or `subject`.
- `REST_CARD.subject` exactly SOURCE.
- Version-appropriate `REST_CARD.selection.quantity`.
- `PLACE_HAND_CARD_BOTTOM_DECK` payment structure.
- `RETURN_CHARACTER_TO_OWNER_HAND` payment structure from §17.6.
- Valid `REMAINDER` structure.
- Compatible earlier `REMAINDER.excluding` selections.
- Compatible `POSITION_CARD` references.
- No blocked ability presented as executable.
- `ON_OPPONENT_CHARACTER_KO` is accepted only as an `AUTO` timing.
- `EVENT_OBJECT` is used only in its authorized event context.
- `EVENT_OBJECT` is not used as an unrestricted instance identifier or global variable.

For `REST_CARD.selection`, validators must enforce the exact version rules:

- Up to 1.4.0: `FIXED 1`.
- 1.5.0 and later: positive `FIXED` quantity.

### 23.3 Referential

Validate:

- All references use §14 forms.
- `source_refs`, replacement rule references, and blocker references resolve.
- Selection references identify compatible earlier results in the same effect.
- `depends_on` identifies a compatible preceding step.
- Areas identify valid players and names.
- Properties and states apply to the referenced object.
- `EVENT_ACTIVATED_BY` occurs only in the specified timing context.
- Owner destinations resolve separately for each selected card.
- Color comparisons reference compatible card objects.
- LOOK and REMAINDER references point to earlier compatible steps.
- REMAINDER exclusions use compatible earlier SELECT steps.
- `ATTACKER` appears only in valid battle context.
- `PREVENT_BATTLE_KO.attacker_condition` evaluates that attacker.
- `target_scope.filter` and `allowed_targets.filter` use `CANDIDATE`.
- Attack restrictions contain no arbitrary instance-ID access.
- `EVENT_OBJECT` points to the exact Character associated with the corresponding `ON_OPPONENT_CHARACTER_KO` occurrence.
- `EVENT_OBJECT` cannot reference an unrelated event, card, or effect.
- No global or implicit selection reference exists.

### 23.4 Semantic

Validate:

- Correct category, timing, and activation window.
- Event-resolution timing occurs only after actual Event-card resolution.
- Trigger activation is not confused with Event activation.
- Correct condition context and evaluation point.
- `ON_KO` refers to the source instance's own K.O.
- `ON_OPPONENT_CHARACTER_KO` requires an actual opponent Character K.O., not ordinary Trash or movement.
- `ON_OPPONENT_CHARACTER_KO` uses the opponent relative to the source controller at event occurrence.
- `EVENT_OBJECT` identifies the actual K.O.'d Character and is not confused with SOURCE.
- Conditions for `ON_OPPONENT_CHARACTER_KO` use the state at the triggering event, not a later state.
- The applicable OP01-061 FAQ interpretation is preserved: a missing `[DON!! x1]` requirement at the K.O. event prevents activation, even if other state changes occur during the same processing.
- Normal Auto Effect source-area restrictions remain in force.
- Cost and resolution remain separate.
- Optional payment and quantity choice remain separate.
- Life payment follows top-card rules and does not create damage/Trigger.
- Hand-to-bottom payment occurs before resolution and does not shuffle.
- Character-to-owner-hand payment uses the payer's Character Area and its owner's Hand.
- Optional Character return uses `optional = true` plus `FIXED 1`, not `UP_TO 1`.
- Direct SOURCE payment rests the exact source instance.
- Multiple-card REST_CARD selection uses distinct eligible active cards and exact fixed cardinality.
- Multiple-card REST_CARD remains one payment, not multiple optional payments.
- Optional payment decline and accepted fixed quantity are separate decisions.
- LOOK preserves identified card instances.
- REMAINDER contains only the proper remaining LOOK cards.
- Excluded or already-moved cards are not included.
- Mode choice and selection are separate.
- Chooser and executor are separate.
- `IF`, `THEN`, and `IF_YOU_DO` remain distinct.
- Explicit textual THEN clauses are preserved.
- Operations correspond to official procedures.
- Ordering and simultaneity remain correct.
- Multicolor comparisons use complete color sets.
- Printing overrides apply before property evaluation.
- Continuous effects have correct subject/scope, validity, and duration.
- Replacement Effects satisfy §16.
- K.O., Trash, damage, Life, Trigger, play, and DON!! −X remain distinct.
- Type arrays preserve official type boundaries.
- Every gameplay ability and clause has semantic coverage.
- No independent Trigger or activation is omitted.
- Partial translations cannot be `SUPPORTED`.
- `GRANT_ATTACK_TARGET_PERMISSION` only adds declared target eligibility.
- `RESTRICT_ATTACK_TARGETS` only narrows the target set of the specified attacking player.
- Attack restrictions combine cumulatively and are not overridden by added attack permissions.
- Restrictions do not cancel already declared attacks.
- `PREVENT_BATTLE_KO` only prevents the specified battle K.O.
- `ATTACKER` uses correct battle context and effective characteristics.
- No unsupported semantics are approximated.

### 23.5 Normative

- Interpretations from CR, FAQ, and errata must be traceable using `source_refs` or blocker references.
- FAQs must not be generalized beyond their cases.
- Errata must be applied before ability decomposition.
- `raw_text` never overrides current official text.
- An applicable errata reference is mandatory when it changes the gameplay wording used.
- Physical printing cannot revive obsolete text when errata applies to the logical card identity.
- Printing overrides require reliable evidence.
- Provider-specific names and artwork labels are not gameplay differences.
- `SUPPORTED` cannot rely on assumed text, characteristics, or rules.
- Complete gameplay text must be accounted for.
- Official names and type boundaries must not be inferred from provider suffixes, concatenation, internal identifiers, or artwork labels.
- The OP01-061 event-time requirement must follow the applicable official FAQ and cannot be replaced by a generic delayed-condition evaluation.

## 24. Blocking Policy

The translator must block when the text and applicable official sources do not establish:

- Player, owner, controller, chooser, executor, or affected player.
- Timing, window, condition, or evaluation point.
- Designated object, selection, cardinality, or information access.
- Random selection without an applicable construct.
- Order, simultaneity, or dependency.
- Cost, payment, or optionality.
- Replaced processing, application point, replaced part, or procedure.
- Validity, scope, or duration.
- Meaning of `if you do`.
- Instance identity or applicable text version.
- Characteristics of conflicting physical printings.
- Printed type boundaries.
- Exact movement/result set.
- Additional attack-target class.
- Restricted attack-target class.
- Protected object or qualifying battle attacker.

### 24.1 Blocker Meanings

`BLOCKED_NORMATIVE`: official normative support or clarification is missing.

`BLOCKED_CAPABILITY`: the semantics are known, but the declared language version lacks the construct.

`INVALID_SOURCE`: text, errata, language/region, or source information is insufficient or inconsistent.

### 24.2 Existing Capabilities That Must Not Be Blocked

Do not use `BLOCKED_CAPABILITY` when the behavior is already expressible through:

- `AFTER_EVENT_CARD_RESOLUTION`.
- `EVENT_ACTIVATED_BY`.
- `ON_OPPONENT_CHARACTER_KO`.
- `EVENT_OBJECT` in its authorized context.
- `ADD_LIFE_TO_HAND`.
- `PLACE_HAND_CARD_BOTTOM_DECK`.
- `RETURN_CHARACTER_TO_OWNER_HAND`.
- `REST_CARD.subject = SOURCE`.
- `REST_CARD.selection` with version-appropriate fixed quantity.
- `REMAINDER`.
- `COLOR_SET_COMPARE`.
- `card.printing_overrides`.
- `GRANT_ATTACK_TARGET_PERMISSION`.
- `RESTRICT_ATTACK_TARGETS`.
- `PREVENT_BATTLE_KO`.

Do not block merely because an ability involves battle, attacks, target restrictions, attacker characteristics, K.O., or continuous protection. Inspect the actual constructs first.

Do not use one construct to approximate behavior belonging to another.

Never convert a blocker into an empty operation, default value, approximated branch, or omitted text.

Never invent an enumeration, opcode, property, player, or exception.

## 26. Reference Examples

Examples demonstrate structures and do not replace checking actual text, CR, FAQ, or errata.

### 26.1 Simple Permanent Effect

```
[[abilities.effects]]
id = "E1"
category = "PERMANENT"
valid_when = { all = [
  { test = "DON_GIVEN_TO_SOURCE", operator = "GTE", value = { literal = 1 } },
  { test = "PLAYER_TURN", value = "YOUR_TURN" }
] }
continuous = {
  scope = {
    object = "CARD",
    areas = [{ kind = "AREA", player = { kind = "PLAYER", role = "YOU" }, name = "CHARACTER_AREA" }],
    player = { kind = "PLAYER", role = "YOU" }
  },
  kind = "MODIFY_POWER",
  operator = "ADD",
  value = { literal = 1000 }
}
```

### 26.2 Ordered Auto Effect

```
[[abilities.effects]]
id = "E1"
category = "AUTO"
timing = "ON_PLAY"
effect_form = "ONE_SHOT"
resolution = [
  { id = "S1", kind = "OPERATION", operation = "DRAW", player = { kind = "PLAYER", role = "YOU" }, quantity = { mode = "FIXED", value = { literal = 1 } } }
]
```

### 26.3 Resolution Condition

```
resolution = [
  { id = "S1", kind = "OPERATION", operation = "TRASH_HAND", player = { kind = "PLAYER", role = "YOU" }, selection = "S0" },
  { id = "S2", kind = "IF", condition = { test = "AREA_COUNT", area = "TRASH", player = { kind = "PLAYER", role = "YOU" }, operator = "GTE", value = { literal = 5 } }, steps = [
    { id = "S2A", kind = "OPERATION", operation = "SET_BASE_POWER", subject = { kind = "SOURCE" }, value = { literal = 5000 }, duration = { kind = "UNTIL_END_OF_TURN" } }
  ] }
]
```

The Condition is evaluated when the IF step is reached, not frozen at the start of resolution.

### 26.4 Trigger and Event

Non-Event Trigger:

```
[[abilities]]
id = "A1"
source_text = "[Trigger] Draw 1 card."
status = "SUPPORTED"
keyword_effects = ["TRIGGER"]

[[abilities.effects]]
id = "E1"
category = "AUTO"
timing = "TRIGGER"
effect_form = "ONE_SHOT"
resolution = [
  { id = "S1", kind = "OPERATION", operation = "DRAW", player = { kind = "PLAYER", role = "YOU" }, quantity = { mode = "FIXED", value = { literal = 1 } } }
]
```

Event Trigger activating the card:

```
event_activation = { window = "TRIGGER" }
```

The Event's normal effect remains a separate `ACTIVATE` with `EVENT_CARD_RESOLUTION`.

### 26.5 Optional Life Payment

```
activation_costs = [
  { kind = "ADD_LIFE_TO_HAND",
    payer = { kind = "PLAYER", role = "YOU" },
    quantity = { mode = "UP_TO", maximum = { literal = 1 } },
    optional = true
  }
]
```

The payment is not damage or Trigger.

### 26.6 LOOK Remainder

```
resolution = [
  { id = "S1", kind = "OPERATION", operation = "LOOK", area = { kind = "AREA", player = { kind = "PLAYER", role = "YOU" }, name = "DECK" }, player = { kind = "PLAYER", role = "YOU" }, quantity = { mode = "FIXED", value = { literal = 5 } } },
  { id = "S2", kind = "SELECT", object = "CARD", areas = [{ kind = "AREA", player = { kind = "PLAYER", role = "YOU" }, name = "DECK" }], quantity = { mode = "UP_TO", maximum = { literal = 1 } }, chooser = { kind = "PLAYER", role = "YOU" }, information = "LOOKED_BY_CHOOSER", information_step = "S1" },
  { id = "S3", kind = "THEN", steps = [
    { id = "S3A", kind = "REMAINDER", look_step = "S1", excluding = ["S2"] },
    { id = "S3B", kind = "OPERATION", operation = "POSITION_CARD", selection = "S3A", destination_area = { kind = "AREA", player = { kind = "PLAYER", role = "YOU" }, name = "DECK" }, position = "BOTTOM", order_chooser = { kind = "PLAYER", role = "YOU" } }
  ] }
]
```

REMAINDER identifies only the unselected cards from the earlier LOOK result that remain in the examined area.

### 26.7 Returning a Card to Its Owner

```
{ id = "S1", kind = "OPERATION", operation = "RETURN_CARD", selection = "S0", destination_area = { kind = "AREA", player = { kind = "PLAYER", role = "OWNER" }, name = "HAND" } }
```

The engine resolves the owner separately for each card.

### 26.8 Attack-Target Permission

```
[[abilities.effects]]
id = "E1"
category = "PERMANENT"
valid_when = { test = "DON_GIVEN_TO_SOURCE", operator = "GTE", value = { literal = 1 } }

[abilities.effects.continuous]
kind = "GRANT_ATTACK_TARGET_PERMISSION"
subject = { kind = "SOURCE" }
target_scope = {
  object = "CARD",
  areas = [{ kind = "AREA", player = { kind = "PLAYER", role = "OPPONENT" }, name = "CHARACTER_AREA" }],
  states = ["ACTIVE"]
}
```

This adds target eligibility. It does not grant Rush or override attack restrictions.

### 26.9 Battle K.O. Prevention

```
[[abilities.effects]]
id = "E1"
category = "PERMANENT"
valid_when = { test = "DON_GIVEN_TO_SOURCE", operator = "GTE", value = { literal = 2 } }

[abilities.effects.continuous]
kind = "PREVENT_BATTLE_KO"
subject = { kind = "SOURCE" }
attacker_condition = { all = [
  { test = "PROPERTY_COMPARE", subject = { kind = "ATTACKER" }, property = "CATEGORY", operator = "EQ", value = { literal = "CHARACTER" } },
  { test = "PROPERTY_COMPARE", subject = { kind = "ATTACKER" }, property = "ATTRIBUTE", operator = "EQ", value = { literal = "STRIKE" } }
] }
```

The condition applies to the actual attacker and only to K.O. caused by battle.

### 26.10 OP01-051 — Eustass "Captain" Kid

This card demonstrates errata, restricted attack targets, multiple-source interaction, direct SOURCE payment, and an independent Activate Effect.

The complete translation must contain:

- An official `ERRATA` reference dated February 17, 2023.
- The corresponding OP01-051 FAQ reference.
- The OP11-010 FAQ reference.
- A `PERMANENT` effect with `valid_when` conditions for `DON_GIVEN_TO_SOURCE >= 1`, `OPPONENT_TURN`, and the source being RESTED.
- `RESTRICT_ATTACK_TARGETS`, with SOURCE as `subject`, OPPONENT as `attacking_player`, and Characters named `Eustass"Captain"Kid` as `allowed_targets`.
- A separate `ACTIVATE` effect using `MAIN_PHASE` and `ONCE_PER_TURN`.
- An optional `REST_CARD.subject = SOURCE` payment.
- A subsequent `PLAY_CARD` using a selection of up to one Character costing 3 or less from the player's Hand.

The OP01-051 FAQ permits attacking any of the affected player's Characters with the specified card name when multiple copies establish the restriction. It does not restrict attacks to the exact source instance.

The OP11-010 FAQ confirms that another effect granting additional attack targets does not override this restriction.

The erratum changes both the attack-restriction wording and the subsequent `Play up to 1` instruction. The annotation about errata is not gameplay text.

The Activate Effect is independent. Its optional source-resting payment is `REST_CARD.subject = SOURCE`, and its subsequent selection is `UP_TO 1`, allowing zero when the text/CR permits.

### 26.11 OP01-055 — You Can Be My Samurai!!

This card demonstrates a multiple-Character resting payment, Event activation, optional payment, and the FAQ distinction between activating an Event card and resolving its effect.

Its translation must contain:

- An Event card with `event_activation.window = "MAIN"`.
- A separate `ACTIVATE` effect with `activation_window = "EVENT_CARD_RESOLUTION"`.
- A single optional `REST_CARD.selection` payment using `FIXED 2`.
- A filter identifying distinct active Characters in the player's Character Area.
- A subsequent `DRAW 2` operation.

The payment is optional as a whole. If accepted, exactly two distinct eligible Characters must be selected and rested.

Characters played during the current turn are eligible, as confirmed by the official FAQ.

The Event card may be activated without resting the two Characters, but its effect does not resolve. This is not equivalent to paying a zero or partial quantity.

The payment is one `REST_CARD` entry with `FIXED 2`, not two independent optional costs.

### 26.12 OP01-061 — Kaido

This card demonstrates the FCL 1.6.0 timing, event-time activation conditions, `EVENT_OBJECT`, `DON!! x1`, `[Your Turn]`, `[Once Per Turn]`, and `ADD_DON`.

```
format = "FL_CARD_LANGUAGE"
language_version = "1.6.0"

[card]
number = "OP01-061"
name = "Kaido"
category = "LEADER"
colors = ["BLUE", "PURPLE"]
types = ["Animal Kingdom Pirates", "The Four Emperors"]
attributes = ["STRIKE"]
power = 5000
life = 4

[source]
raw_text = "[DON!! x1] [Your Turn] [Once Per Turn] When your opponent's Character is K.O.'d, add up to 1 DON!! card from your DON!! deck and set it as active. This card has been officially errata'd."
current_text = "[DON!! x1] [Your Turn] [Once Per Turn] When your opponent's Character is K.O.'d, add up to 1 DON!! card from your DON!! deck and set it as active."
language = "en"
region = "EN_NA_EU_OC_LATAM_ME"

[[source.references]]
id = "CR-1.2.1"
kind = "CR"
document = "ONE_PIECE_CARD_GAME_COMPREHENSIVE_RULES"
version = "1.2.1"
url = "https://en.onepiece-cardgame.com/pdf/rule_comprehensive.pdf?20260828="

[[source.references]]
id = "OP01-061-FAQ"
kind = "FAQ"
document = "OFFICIAL_OP01_FAQ"
section = "OP01-061"
date = "2024-04-05"
url = "https://en.onepiece-cardgame.com/pdf/qa_op01.pdf?20240405="

[translation]
status = "SUPPORTED"

[[abilities]]
id = "A1"
source_text = "[DON!! x1] [Your Turn] [Once Per Turn] When your opponent's Character is K.O.'d, add up to 1 DON!! card from your DON!! deck and set it as active."
status = "SUPPORTED"
source_refs = ["CR-1.2.1", "OP01-061-FAQ"]

[[abilities.effects]]
id = "E1"
category = "AUTO"
timing = "ON_OPPONENT_CHARACTER_KO"
usage_limit = { kind = "ONCE_PER_TURN" }
effect_form = "ONE_SHOT"

activation_conditions = { all = [
  { test = "PLAYER_TURN", value = "YOUR_TURN" },
  { test = "DON_GIVEN_TO_SOURCE", operator = "GTE", value = { literal = 1 } }
] }

resolution = [
  { id = "S1",
    kind = "OPERATION",
    operation = "ADD_DON",
    player = { kind = "PLAYER", role = "YOU" },
    quantity = { mode = "UP_TO", maximum = { literal = 1 } },
    arrival_state = "ACTIVE"
  }
]
```

The API annotation in `raw_text` is not a gameplay instruction and is excluded from `current_text`.

The official OP01-061 FAQ establishes that the `[DON!! x1]` condition must be satisfied when the opponent's Character is K.O.'d. If the Leader loses the required DON!! at that event point, the Auto Effect does not activate.

This example does not require a separate `ERRATA` reference merely because the original API input contains an annotation claiming that the card has been errata'd. An actual applicable official correction must be independently established before an `ERRATA` reference or altered `current_text` is declared.

## 27. Conformance

A document conforms to FCL when:

1. It is valid TOML 1.0.0.
2. It declares `format = "FL_CARD_LANGUAGE"` and an explicitly supported `language_version`.
3. It follows every authorized structure, vocabulary, and semantic invariant.
4. It passes syntactic, structural, referential, semantic, and normative validation.
5. It contains no blocked gameplay abilities.
6. It contains no unauthorized fields outside `[editorial.extra]`.
7. Every instruction in `source.current_text` is represented.
8. Every 1.3.0/1.4.0 construct retains its original semantics.
9. Every `RETURN_CHARACTER_TO_OWNER_HAND` payment follows §17.6.
10. Every `RESTRICT_ATTACK_TARGETS` declaration follows §20.3.
11. Every multi-card `REST_CARD.selection` follows §§17.2 and 17.3.
12. Every `ON_OPPONENT_CHARACTER_KO` declaration and associated `EVENT_OBJECT` usage follows §§11.1, 12, and 14.
13. Every `scale` expression follows the count-source, multiplier, numeric-value, and evaluation rules of §13.

`SUPPORTED` is a translation conclusion, not proof that the VM implements every game rule.

Program conformance, compiler coverage, and engine coverage are separate checks.

Older documents keep their declared versions and semantics.

## 28. Version 1.6.0 Freeze

FCL 1.6.0 freezes the complete 1.5.0 profile and adds:

- `ON_OPPONENT_CHARACTER_KO`, a distinct Auto Effect timing.
- Explicit distinction between a source's own K.O. and an opponent Character's K.O.
- `EVENT_OBJECT` semantics for the opponent Character that fulfilled the new timing.
- Contextual `YOU`/`OPPONENT` resolution relative to the source controller at event occurrence.
- Event-time activation-condition evaluation.
- Preservation of the relevant event state, consistent with the OP01-061 FAQ.
- Validation, referential, and semantic rules for the extension.
- OP01-061 as a normative reference example.

The complete frozen profile retains:

- TOML 1.0.0.
- One logical card file per official card identity.
- Identity, gameplay data, source, translation, editorial, and printing separation.
- Auto, Activate, Permanent, and Replacement Effects.
- Conditions, references, values, selections, choices, and information.
- Supported activation costs and payments, including optional payments.
- `ADD_LIFE_TO_HAND`.
- `PLACE_HAND_CARD_BOTTOM_DECK`.
- `RETURN_CHARACTER_TO_OWNER_HAND`.
- Direct `REST_CARD.subject = SOURCE`.
- Multiple-card `REST_CARD.selection`.
- `REMAINDER` for LOOK results.
- Explicit `THEN` clause boundaries.
- Printing-specific gameplay characteristic overrides.
- Complete source-text and ability-coverage requirements.
- Errata traceability.
- Multicolor-aware color comparisons.
- Trigger and Event-card activation handling.
- The complete Replacement Effect structure.
- `AFTER_EVENT_CARD_RESOLUTION`.
- `EVENT_ACTIVATED_BY`.
- `ON_OPPONENT_CHARACTER_KO`.
- `EVENT_OBJECT` in its defined context.
- `GRANT_ATTACK_TARGET_PERMISSION`.
- `RESTRICT_ATTACK_TARGETS`.
- `PREVENT_BATTLE_KO`.
- Battle-context attacker evaluation.
- Strict validation and blocking.

Versions 1.0.0–1.5.0 remain frozen with their original meanings.

Version 1.6.0 is ready for real card-translation testing. It does not imply complete card coverage or a completed VM.

Cases outside the profile remain blocked.

Changes to this version require a new versioned specification, not silent semantic modifications.

## 29. Engine Boundaries and Additional Normative Semantics

The following rules define responsibilities of the execution environment rather than additional card instructions.

### 29.1 Identity and Roles

A printed card is not the same as its in-game instance.

An instance has:

- Area.
- Owner.
- Controller/context.
- State.
- DON!! relationships.
- Printing identity.
- Regulatory history.

Area changes may create new instances according to the applicable procedures.

Effective characteristics derive from base `[card]` values with the relevant printing overrides applied.

Accepted references are those in §14.

`OWNER` and `CONTROLLER` are relative to their contextual object.

`TEXT_DESIGNATED_OBJECT.designation` reproduces literal text and identifies one instance.

`NONE` is not an area.

Leader, Character, Stage, and Cost Areas make up the Field.

DON!! in DON!! Deck, Cost Area, and attached to cards are different positions.

Attached DON!! is neither active nor rested, is not available in Cost Area, and cannot be transferred directly between cards.

When the host leaves the Field, returning attached DON!! follows the engine's official procedure.

### 29.2 Events, Activation, and Ordering

An event is a contextual occurrence; a timing is the regulatory point at which an ability may activate.

The engine generates events and timings. FCL declares only supported timings.

`YOUR_TURN` and `OPPONENT_TURN` are Conditions, not events.

Auto Effect separates satisfied timing, pending effect, activation, and resolution.

If the source leaves its required area before activation, the pending Auto Effect does not activate under CR 8-1-3-1-3.

An already activated effect creating a continuous consequence follows its declared duration and validity. Leaving the Field does not automatically erase it.

`AFTER_EVENT_CARD_RESOLUTION` is created only after an actual Event card activation has resolved. The engine retains the activating player.

Event Trigger activation is different and does not create that timing.

`ON_KO` occurs only through official K.O. processing involving the source instance, not arbitrary Trash movement.

`ON_OPPONENT_CHARACTER_KO` is generated by an actual K.O. of a Character in the opponent's Character Area. It is not generated by ordinary Trash or area movement.

For `ON_OPPONENT_CHARACTER_KO`, the engine retains the exact event object, source-controller context, and event-time condition state. Event-time conditions must not be evaluated against later state changes.

Trigger is offered only during official Life damage processing, instead of adding the card to Hand.

If declined, normal processing continues. If accepted, the official Trigger procedure applies.

If an effect/replacement prevents the card from being added to the Hand, Trigger cannot activate (CR 4-6-3-1).

Ordinary Life movement/payment does not offer Trigger.

Double Attack processes damage/Trigger point by point.

Event activation differs from activating a card ability.

Event `[Main]`/`[Counter]` windows authorize card activation; the effect after activation is represented separately.

Blocker, Counter, battle, phases, and priority belong to the engine.

### 29.3 Conditions, Selection, and Information

Activation conditions are checked before activation, at the timing-defined evaluation point.

Validity conditions are evaluated continuously.

Resolution conditions are evaluated at the required point and may observe changes caused by previous steps.

Never assume the game state is frozen at the beginning of resolution.

Selection identifies objects; it does not move, reveal, K.O., or affect them.

Fixed and `UP_TO` quantities are different.

`DRAW UP TO X` allows stopping before each draw.

An absent text-designated object is not equivalent to an empty selection.

Secret areas follow official information rules. LOOK, REVEAL, and selection remain distinct.

Life movement uses the top-card procedure under CR 3-10-2 unless otherwise specified.

Life-to-hand payment is not damage or Trigger.

Multicolor comparison uses complete effective color sets.

`DISJOINT` means no shared color, not merely unequal arrays.

When both players choose simultaneously, Turn Player chooses first.

When cards are simultaneously placed in an area, ordering follows the card owner and CR unless otherwise specified.

LOOK members retain their identities during the same resolution.

REMAINDER is evaluated when its step is reached. Already moved cards and explicitly excluded compatible selections are omitted.

REMAINDER does not grant new information, create an area, or authorize access outside the original LOOK.

`GIVE_DON.source_state` records the pre-attachment state. Rested DON!! requires explicit text/FAQ authorization. EB05-057 is not general authorization.

Printing overrides apply before property evaluation; continuous modifications follow their normative processing order.

### 29.4 Resolution and Impossibility

Resolution follows textual order.

Impossible actions do not occur, and there is no general rollback.

Consequences for subsequent steps depend on the text/CR.

`IF`, `THEN`, and `IF_YOU_DO` are distinct.

Independent instructions continue when the text/CR establishes independence.

Playing Cost and Activation Cost are distinct.

If the entire payment is impossible before activation, the effect cannot activate.

If payment becomes impossible during processing after activation, pay as much as possible; the subsequent resolution is not performed, and applicable `[Once Per Turn]` usage remains consumed.

Declining an optional payment prevents its associated effect from activating.

Accepting optional payment and choosing its quantity are separate decisions.

Mode selection, `UP_TO` selection, and optional resolution actions are not costs.

`THEN` marks a textual clause; it does not invent a dependency on complete success of a previous clause.

### 29.5 Continuous Effects, Negation, and Prohibition

Permanent Effects remain valid while their conditions and the rules permit.

Effects explicitly permitted to apply in secret areas remain valid there only when text/CR establishes the exception.

Continuous effects stabilize in Turn Player, then Non-Turn Player order, repeating when state changes (CR 8-1-3-3-5).

Action prohibition, effect negation, and operation impossibility are different phenomena.

FCL 1.6.0 does not provide arbitrary action prohibition, general effect negation, or unrestricted game-procedure modification.

Do not simulate those capabilities through Keyword removal or movement.

The supported exceptions are:

- `GRANT_ATTACK_TARGET_PERMISSION`: adds only the target eligibility described by its target scope.
- `RESTRICT_ATTACK_TARGETS`: restricts the affected player's target choices to the described allowed set.
- `PREVENT_BATTLE_KO`: prevents only the described battle K.O. consequence.

These are not interchangeable and do not create generic rule modification.

### 29.6 Keywords and Procedures

Recognized Keyword Effects:

- Rush.
- Double Attack.
- Banish.
- Blocker.
- Trigger.
- Rush: Character.
- Unblockable.

Each invokes its official procedure.

Other Keywords are represented semantically through:

- `ON_PLAY`
- `ON_KO`
- `ON_OPPONENT_CHARACTER_KO`
- `WHEN_ATTACKING`
- `ON_BLOCK`
- `ON_OPPONENT_ATTACK`
- `MAIN`
- `COUNTER`
- `DON!! xX`
- `DON!! −X`
- `YOUR_TURN`
- `OPPONENT_TURN`
- `ONCE_PER_TURN`
- `TRASH`

The language does not provide generic control transfer, arbitrary effect copying, attack redirection, attack/battle cancellation, unrestricted rule modification, or indefinite repetition.

`GRANT_ATTACK_TARGET_PERMISSION` only adds targets and never grants Rush.

`RESTRICT_ATTACK_TARGETS` only narrows legal targets. It does not cancel or redirect an attack.

`PREVENT_BATTLE_KO` does not create immunity or negate effects.

Blocker is not generic redirection; ending a battle is a consequence of the battle procedure.

### 29.7 Incorporated FAQ Cases and Boundaries

Only the specific cases below are incorporated. Do not generalize them beyond their official scope.

- OP01-004: An ability triggered by an opponent activating an Event activates after that Event resolves. Event Trigger activation is not Event activation.
- OP01-021: Franky's additional target permission does not permit attacking on the turn it is played; it does not grant Rush.
- OP01-022: Brook cannot select the same Character twice for two Power reductions.
- OP01-024: The protection applies to K.O. caused by battle results, not effects.
- OP01-047: The returned Character may be played by the same effect when it enters the payer's Hand and satisfies the conditions.
- OP01-051: When multiple copies establish the restriction, any of the affected player's Characters with the card name Eustass"Captain"Kid may be attacked. This is not limited to the exact source instance.
- OP01-051 errata: The February 17, 2023 correction changes the attack restriction from "any Character other than" to "any card other than the Character" with the specified name and changes `Play 1` to `Play up to 1`.
- OP01-055: Characters played during the current turn may be rested for the payment. The Event may be activated without resting two Characters, but nothing happens.
- OP01-061: The `[DON!! x1]` condition is checked when the opponent's Character is K.O.'d. If the Leader simultaneously loses the DON!! required by this condition, the Auto Effect does not activate. This is the normative basis for event-time condition evaluation under `ON_OPPONENT_CHARACTER_KO`.
- OP11-010: A separate effect granting attacks against active Characters does not override the OP01-051 attack-target restriction.
- EB05-006: Looking at the top Life card before a mode choice is permitted in the specific case addressed.
- EB05-008: A Counter Effect with `up to 1` may activate when no Character is present.
- EB05-011: The specific On Play condition is unsatisfied when the top Life card is already face-down in the case addressed.
- EB05-028: The opponent of the ability's activator chooses the four cards from their own Hand.
- EB05-033: The Leader type requirement and difference in Field DON!! are mandatory.
- EB05-034: A subsequent condition is evaluated after DON!! −2 and may no longer be satisfied.
- EB05-045: On Play can K.O. its source; in the case addressed, it may choose not to K.O. a card.
- EB05-046: Trash count is evaluated after the previous discard.
- EB05-052: Gloriosa's replacement covers the specified battle damage, Double Attack damage, and effect-damage cases.
- EB05-057: Rested DON!! attachment is allowed only in the specific multiple-Attribute scenario, including Special or Wisdom. `GIVE_DON.source_state` captures the pre-attachment state; attached DON!! then has neither state.
- EB05-059: Battle K.O. immunity does not apply in the specific scenario addressed.
- EB05-061: Nami's replacement may be used when its own source would be removed, but not when no Life exists for the consequence.
- EB05-010: Negation of printed, continuously granted, or Trigger-granted Blocker follows the specific FAQ treatment.
- EB05-021: On Play of a Character played by an effect does not automatically chain into another On Play in the addressed case.

OP01-021 establishes the specific additional-target permission. OP01-024 establishes the specific battle K.O. protection. OP01-051 and OP11-010 establish the specific attack-target restriction semantics and its interaction with added permissions. OP01-061 establishes the event-time condition requirement for the opponent Character K.O. timing.

These cases do not create generic attack modification, immunity, negation, or procedure modification.

Unverified historical FAQs, complete errata reconciliation, collective replacement costs, complex replacement interactions, unclear evaluation points, secret-area effects, loops, and regional/language conflicts remain outside the profile.

A specific case requiring an unverified ruling must be `BLOCKED_NORMATIVE`.

Collective replacement costs deliberately have no generic rule.

## Official Normative References

### Comprehensive Rules

ONE PIECE CARD GAME Comprehensive Rules v1.2.1 — August 28, 2026

`https://en.onepiece-cardgame.com/pdf/rule_comprehensive.pdf?20260828=`

Relevant sections:

- CR 2-3-5 — multicolor cards.
- CR 3-10-2 — Life Area secrecy and top-card movement.
- CR 4-6-3-1 — Trigger during damage processing.
- CR 4-12 — Power-related rules.
- CR 8-1-3-1 — Auto Effects.
- CR 8-1-3-3-5 — continuous-effect stabilization.
- CR 8-1-3-4 — Replacement Effects.
- CR 8-3-1-3-1 — payment becoming impossible during processing.
- CR 8-3-1-4 — optional Activation Costs.
- CR 8-3-1-7 — Activation Cost replacement.
- CR 8-6 — activation and resolution ordering.
- CR 10-2-1 — K.O.
- CR 10-2-13 — Once Per Turn usage.

### Official Card Errata

`https://en.onepiece-cardgame.com/rules/errata_card/`

Relevant entries:

- February 17, 2023 — OP01-047 Trafalgar Law: changes its payment to return one of your Characters to its owner's Hand and changes `Play 1` to `Play up to 1`.
- February 17, 2023 — OP01-051 Eustass"Captain"Kid: changes the attack restriction from other Characters to any other card except the specified Character and changes `Play 1` to `Play up to 1`.

The official corrected "After" text takes precedence over the original wording.

### Official OP01 FAQ

`https://en.onepiece-cardgame.com/pdf/qa_op01.pdf?20240405=`

Relevant entries:

- OP01-004 — Event-resolution timing.
- OP01-021 — additional target permission does not grant Rush.
- OP01-022 — distinct Character selection.
- OP01-024 — battle K.O. protection.
- OP01-047 — returned Character may be played.
- OP01-051 — multiple copies of the attack-target restriction.
- OP01-055 — resting Characters played during the current turn and optional payment behavior.
- OP01-061 — DON!! condition evaluation when the opponent's Character is K.O.'d.

### Official OP11 FAQ

`https://en.onepiece-cardgame.com/pdf/qa_op11.pdf?20250523=`

Relevant entry:

\- OP11-010 — an additional attack-target permission does not override the OP01-051 restriction.

These sources establish only the specific rules and FAQ interpretations listed. They do not certify that every published card, FAQ, erratum, printing, or interaction has been audited.
