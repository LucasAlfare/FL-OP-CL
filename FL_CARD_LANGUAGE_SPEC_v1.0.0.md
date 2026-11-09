# FL Card Language (FCL) — Specification 1.0.0

Single Normative Document: `FL_CARD_LANGUAGE_SPEC_v1.0.0.md`\
Textual Format: TOML 1.0.0\
Language and Semantic Version: 1.0.0\
Freeze Date: November 8, 2026\
Status: `FROZEN — READY FOR CARD-TRANSLATION TESTING`

## 1. Purpose and Authority

The FL Card Language (FCL) is a declarative language for describing One Piece Card Game card abilities and effects as data programs interpretable by a Rules Engine/VM.

The card declares its specific behavior; the official rules define the procedures and consequences; the engine executes those procedures. FCL is not a general-purpose language. It does not contain Kotlin code, user-defined functions, callbacks, mutable variables, unrestricted loops, recursion, or arbitrary access to game state.

Version 1.0.0 defines, in a single document, the canonical TOML format, the closed semantic profile, vocabularies, Replacement Effect structure, validation rules, and blocking policy. This document supersedes any previous specification, semantic contract, or addendum for FCL 1.0.0. No external document needs to be combined with it.

The external lexical and syntactic grammar is TOML 1.0.0. FCL adds a closed profile of tables, fields, types, enumerations, and semantic invariants. `language_version = "1.0.0"` identifies both the structure and the meaning of the constructs. There is no separate semantic-profile field.

`FROZEN` means that the contract for this version is closed and ready for card translation and testing to begin. It does not mean that every published card is supported, that the compiler/VM already exists, or that every game interaction has been implemented. Any behavior outside this profile must be blocked rather than approximated. Structural or semantic changes require a new version.

## 2. Scope and Responsibility Boundaries

The normative foundation is the ONE PIECE CARD GAME Comprehensive Rules v1.2.1 (August 28, 2026), together with the currently applicable official card text, errata, and FAQs. Official sources and the audit boundary are listed at the end of this document.

### 2.1 Included

- printed game identity and separately maintained editorial data;
- Auto, Activate, Permanent, and Replacement Effects;
- recognized Keyword Effects and their official procedures;
- activation, validity, and resolution conditions, filters, and predicates;
- selections, quantities, mode choices, and information;
- activation costs and supported payments;
- ordered resolution, dependencies, optional choices, simultaneity, and finite repetition;
- game operations enumerated in this profile;
- continuous effects, scope, validity, and duration;
- source references, traceability, and translation blocking.

### 2.2 Excluded

- complete rules for turns, phases, battles, Blocker, Counter, damage, Life, Trigger, K.O., priority, `rule processing`, defeat, and general loops;
- implementation of Rules Engine/VM procedures, bytecode, and binary layout;
- format legality, banlists, and deck construction;
- operations, conditions, timings, durations, or properties not enumerated in this document;
- arbitrary behavior, card-specific code, or instructions exclusive to a particular card.

Deck construction restrictions and effects that modify deck construction rules are outside the FCL 1.0.0 gameplay program. Editorial legality data may be stored outside executable instructions.

### 2.3 Mandatory Semantic Principles

1. Category and duration are different dimensions. The four categories are `AUTO`, `ACTIVATE`, `PERMANENT`, and `REPLACEMENT`. `ONE_SHOT` and `CONTINUOUS` describe the temporal form of an effect, not a fifth category. Permanent Effect is a category; Continuous Effect is a duration/consequence form.
2. Activation, cost, choice, selection, and resolution are distinct. The timing at which an event occurs, eligibility, activation, payment, mode choice, object selection, and resolution must not be conflated.
3. Impossibility is not a universal failure state. An unsatisfied condition, impossible cost, impossible action, missing target, empty selection, `up to 0` choice, declined replacement, and invalidated effect are different situations. There is no universal rollback, atomicity, or continuation rule.
4. Do not infer equivalence. K.O. is not Trash; damage is not movement from Life; drawing is not selecting from the Deck; playing a card is not moving it to the Field; Trigger is not any movement involving Life; effective Power is not base Power; effective Cost is not base Cost; owner is not controller; chooser is not executor.
5. Syntax does not create semantics. When the text/rules/FAQ do not determine the subject, object, player, timing, quantity, order, cost, scope, or consequence, translation must be blocked.
6. The card does not reimplement the engine. Priority, phases, battle, damage processing, K.O., Trigger, permanent-effect stabilization, information rules, competing replacements, and loops are engine procedures.

A structurally valid TOML document may still be semantically incorrect. `SUPPORTED` requires structural validation and normative justification, not merely successful parsing.

## 3. Authoring Unit and External Organization

### 3.1 One Logical Unit per Card

The canonical authoring unit is one TOML file per logical card identity (`card number`). Artwork variations and printings do not duplicate programs when they share the same identity and gameplay text/rules.

A card unit has five conceptual sections, all within the same TOML document:

1. game identity (`[card]`);
2. abilities and programs (`[[abilities]]` and descendant structures);
3. normative sources (`[source]` and descendant structures);
4. translation results and blockers (`[translation]`);
5. editorial metadata and printings (`[editorial]`).

This separation is semantic and structural. It does not require physically separate files.

### 3.2 Non-Instructions

Game identity, provenance, and editorial data are not executable instructions. The compiler validates and classifies them separately.

Editorial metadata must not be sent to the VM as instructions. Normative source information is required for auditing but does not need to be included in the binary execution catalog.

## 4. Mandatory TOML Rules

The implementation must accept TOML 1.0.0 and reject extensions from later versions.

| Rule           | Requirement                                                                  |
| -------------- | ---------------------------------------------------------------------------- |
| Encoding       | UTF-8                                                                        |
| Keys           | `snake_case`, case-sensitive                                                 |
| Enumerations   | Strings in `UPPER_SNAKE_CASE`                                                |
| Local IDs      | Stable strings, unique within their defined scope                            |
| Arrays         | Order is significant when representing sequences, alternatives, or printings |
| Tables         | Physical table/key order has no semantic meaning                             |
| Missing values | Omit inapplicable keys; TOML does not use `null`                             |
| Unknown keys   | Reject                                                                       |
| Duplicate keys | Reject through the TOML parser                                               |
| Dates          | ISO `YYYY-MM-DD` strings, not TOML datetime values                           |
| Game numbers   | Decimal integers, unless the textual type requires a string                  |
| Comments       | Allowed, with no gameplay meaning                                            |

Do not use YAML anchors, aliases, custom tags, or other YAML features. Do not use free-form text strings as substitutes for required semantic structures.

### 4.1 Ordering

Resolution order must be represented in TOML arrays, never by the physical order of tables or properties. Condition arrays represent conjunction only when the field containing them explicitly defines that semantics; in all other contexts, interpretation is determined by the field's type.

### 4.2 Inline Tables

Inline tables are used for references, dynamic values, conditions, and compact nodes in nested structures. Each inline table must follow TOML 1.0.0 syntax. Extensive resolution structures may be organized as arrays of tables (`[[...]]`).

## 5. Top-Level Structure

The only permitted top-level keys/tables are:

- `format`;
- `language_version`;
- `[card]`;
- `[source]`;
- `[translation]`;
- `[editorial]`;
- `[[abilities]]`.

Minimal structural example:

```
format = "FL_CARD_LANGUAGE"
language_version = "1.0.0"

[card]
number = "OP01-001"
name = "Roronoa Zoro"
category = "LEADER"
colors = ["RED"]

[source]
raw_text = "..."
current_text = "..."
language = "en"
region = "EN_NA_EU_OC_LATAM_ME"

[translation]
status = "SUPPORTED"

[editorial]

[[abilities]]
id = "A1"
source_text = "..."
status = "SUPPORTED"
```

This is a structural example. Required fields, applicability, and vocabularies are specified in the following sections.

## 6. Game Identity — `[card]`

Permitted fields:

| Field        | Type         | Rule                                                     |
| ------------ | ------------ | -------------------------------------------------------- |
| `number`     | string       | Required; official card number                           |
| `name`       | string       | Required                                                 |
| `category`   | enum         | Required: `LEADER`, `CHARACTER`, `EVENT`, `STAGE`, `DON` |
| `colors`     | enum array   | Required; one or more printed colors                     |
| `types`      | string array | Printed types; empty only if officially typeless         |
| `attributes` | enum array   | Applicable only to Leader/Character                      |
| `cost`       | integer      | Applicable to Character/Event/Stage                      |
| `power`      | integer      | Applicable to Leader/Character                           |
| `life`       | integer      | Applicable to Leader                                     |
| `counter`    | integer      | Applicable to Character                                  |

### 6.1 Enumerations

`category`: `LEADER`, `CHARACTER`, `EVENT`, `STAGE`, `DON`.

`colors`: `RED`, `GREEN`, `BLUE`, `PURPLE`, `BLACK`, `YELLOW`.

`attributes`: `SLASH`, `STRIKE`, `RANGED`, `SPECIAL`, `WISDOM`, `QUESTION_MARK`.

`types` are literal printed data, not closed language enumerations. Preserve the spelling and official content used by the selected source.

### 6.2 Applicability

- `life` may exist only for `LEADER`.
- `power` may exist only for `LEADER` and `CHARACTER`.
- `counter` may exist only for `CHARACTER`.
- `cost` may exist only for `CHARACTER`, `EVENT`, and `STAGE`.
- `attributes` may exist only for `LEADER` and `CHARACTER`.
- Do not populate inapplicable fields with zero or empty strings.

The absence of an inapplicable field is different from a numeric value of zero.

## 7. Normative Sources — `[source]`

Required fields: `raw_text`, `current_text`, `language`, and `region`. `edition` is optional. `raw_text` preserves the source as received; `current_text` contains the actual current official text being translated. Applicable errata must be reflected in `current_text`; `raw_text` is never authoritative when it differs from the currently applicable official source.

All sources used in interpretation — Comprehensive Rules, FAQs, errata, and official card text — belong in a single `[[source.references]]` list. There is no separate errata table.

| Field      | Type   | Rule                                                        |
| ---------- | ------ | ----------------------------------------------------------- |
| `id`       | string | Required; unique within the file                            |
| `kind`     | enum   | `CR`, `FAQ`, `ERRATA`, or `OFFICIAL_CARD_TEXT`              |
| `document` | string | Required; identifiable source name                          |
| `url`      | string | Required; official URL                                      |
| `version`  | string | Optional; source version/edition                            |
| `section`  | string | Optional; relevant section/rule/question                    |
| `date`     | string | Optional; ISO `YYYY-MM-DD` date, especially for errata/FAQs |

```
[[source.references]]
id = "CR-8-1-3-4"
kind = "CR"
document = "ONE_PIECE_CARD_GAME_COMPREHENSIVE_RULES"
version = "1.2.1"
section = "8-1-3-4"
url = "https://en.onepiece-cardgame.com/pdf/rule_comprehensive.pdf?20260828="

[[source.references]]
id = "EB05-057"
kind = "FAQ"
document = "OFFICIAL_EB05_FAQ"
section = "EB05-057"
date = "2026-11-06"
url = "https://en.onepiece-cardgame.com/pdf/qa_eb05.pdf?20261106="
```

`source_refs`, `replacement.target.rule_ref`, and `translation.blockers.references` must point to existing IDs in this list. An FAQ only establishes the case it addresses; its outcome must not be generalized.

## 8. Translation Result — `[translation]`

`[translation]` contains `status` (required) and `notes` (optional, with no executable semantics). `status` accepts `SUPPORTED`, `BLOCKED_NORMATIVE`, `BLOCKED_CAPABILITY`, `INVALID_SOURCE`, or `NON_GAME_TEXT`.

Blockers use `[[translation.blockers]]`:

| Field            | Type         | Requirement                                                              |
| ---------------- | ------------ | ------------------------------------------------------------------------ |
| `code`           | enum         | Required: `BLOCKED_NORMATIVE`, `BLOCKED_CAPABILITY`, or `INVALID_SOURCE` |
| `scope`          | enum         | Required: `CARD`, `ABILITY`, or `EFFECT`                                 |
| `ability_id`     | string       | Required for `ABILITY`/`EFFECT`; forbidden for `CARD`                    |
| `effect_id`      | string       | Required only for `EFFECT`; forbidden otherwise                          |
| `source_excerpt` | string       | Required; literal excerpt from `current_text`                            |
| `reason`         | string       | Required; objective description of the gap                               |
| `references`     | string array | Optional; IDs from `[source.references]`                                 |

```
# Fragment: assumes [source.references] contains the ID used in references.
[translation]
status = "BLOCKED_CAPABILITY"

[[translation.blockers]]
code = "BLOCKED_CAPABILITY"
scope = "ABILITY"
ability_id = "A2"
source_excerpt = "..."
reason = "The text requires an operation not defined in the 1.0.0 profile."
references = ["CR-8-1-3-4"]
```

### 8.1 Consistency

- Only `SUPPORTED` may be compiled into gameplay.
- Any blocked gameplay ability blocks the entire card; do not partially compile it.
- Status precedence is `INVALID_SOURCE` > `BLOCKED_NORMATIVE` > `BLOCKED_CAPABILITY` > `SUPPORTED`.
- A `SUPPORTED` ability must be fully represented; `NON_GAME_TEXT` contains no executable instructions.
- `translation.status = "NON_GAME_TEXT"` is used only when there are no translatable gameplay abilities (including cards with no abilities).
- `SUPPORTED` and `NON_GAME_TEXT` must not have blockers. `BLOCKED_*` and `INVALID_SOURCE` states require at least one blocker.
- If at least one gameplay ability exists, `translation.status` must be the highest-severity status among gameplay abilities and card-scope blockers: `INVALID_SOURCE` > `BLOCKED_NORMATIVE` > `BLOCKED_CAPABILITY` > `SUPPORTED`. `NON_GAME_TEXT` abilities are excluded from this comparison. Use `translation.status = "NON_GAME_TEXT"` only when the card has no gameplay abilities.
- Every blocker must correspond to an existing ability/effect when its scope is `ABILITY`/`EFFECT`; statuses and blockers must agree.
- `notes` does not replace `reason` or alter execution.

## 9. Editorial Metadata — `[editorial]`

`[editorial]` is optional and is never sent to the VM. Permitted fields: `rarity`, `set_id`, `set_name`, `illustrator`, `thumbnail_url`, `rights`, `block_number`; `[[editorial.printings]]`; and `[editorial.extra]`.

Each printing accepts only `set_id`, `rarity`, `art_id`, `illustrator`, `thumbnail_role`, `thumbnail_url`, and `rights`. `thumbnail_role`: `SAMPLE`, `CUSTOM`, or `FINAL`. `rights` is an annotation, not a license.

`[editorial.extra]` is the only extensible area: it accepts TOML scalar values (`string`, `integer`, `float`, `boolean`) and homogeneous arrays of those scalars. It does not accept tables, arrays of tables, or executable data. All other unknown keys are rejected.

```
[[editorial.printings]]
set_id = "OP-01"
rarity = "L"
art_id = "OP01-001-ALT"
illustrator = "Illustrator Name"
thumbnail_role = "SAMPLE"
thumbnail_url = "https://example.invalid/card.png"
rights = "UNVERIFIED"

[editorial.extra]
source_api = "optcgapi"
collector_note = "Alternate art"
```

Artwork/printing variations do not duplicate the program when they share the same logical identity and current gameplay text.

## 10. Abilities — `[[abilities]]`

Each `[[abilities]]` represents one logical unit of gameplay text. Fields:

- `id`: required and unique within the card;
- `source_text`: required; exact excerpt of the current official text corresponding to the ability;
- `status`: required; `SUPPORTED`, `BLOCKED_NORMATIVE`, `BLOCKED_CAPABILITY`, `INVALID_SOURCE`, or `NON_GAME_TEXT`;
- `source_refs`: optional; IDs from `[source.references]` supporting the interpretation;
- `keyword_effects`: optional; recognized Keyword Effects present on the card;
- `event_activation`: optional; activation mode for an Event card, distinct from the effect category;
- `effects`: zero or more declarations in `[[abilities.effects]]`.

A `SUPPORTED` ability must be fully represented by `effects`, `keyword_effects`, or `event_activation`. `NON_GAME_TEXT` must not contain executable instructions. A blocked ability may preserve the portion that was understood, but it blocks the entire card and requires a corresponding blocker.

Do not merge distinct abilities or split an ability in a way that changes its cost, timing, condition, choice, order, or dependency.

### 10.1 Keyword Effects

`keyword_effects` accepts only `RUSH`, `DOUBLE_ATTACK`, `BANISH`, `BLOCKER`, `TRIGGER`, `RUSH_CHARACTER`, and `UNBLOCKABLE`. Each term invokes the engine's official procedure; FCL does not reimplement that procedure.

A `TRIGGER` occurrence must correspond to exactly one representation:

(a) For a non-Event card, `keyword_effects = ["TRIGGER"]` together with `AUTO/timing = "TRIGGER"`.

(b) For an Event card with the text `[Trigger] Activate this card.`, `keyword_effects = ["TRIGGER"]` together with `event_activation.window = "TRIGGER"`.

Do not declare both for the same occurrence, and do not register only the keyword.

Other Keywords are not stored in an undifferentiated list. They are represented according to their semantic role: timing, condition, cost, limit, operation, or restriction.

### 10.2 Event Card Activation

`event_activation` describes the authorization/procedure for activating an Event card, not its Activation Cost or a movement operation. Format:

```
event_activation = { window = "MAIN" }
```

`window` accepts `MAIN`, `COUNTER`, or `TRIGGER`. `TRIGGER` is used only when the Event's `[Trigger]` text instructs the player to activate the card itself. If the Trigger contains its own effect (rather than “Activate this card”), that effect is represented as `AUTO` with `timing = "TRIGGER"`, not as `event_activation.window = "TRIGGER"`.

The Event activation window and procedure belong to the engine. The Event's normal effect is declared separately as `ACTIVATE` with `activation_window = "EVENT_CARD_RESOLUTION"`. Activating an Event card must not be confused with activating an ability of another card.

## 11. Effect Declarations — `[[abilities.effects]]`

Each `[[abilities.effects]]` contains:

- `id`: required and unique within the ability;
- `category`: required: `AUTO`, `ACTIVATE`, `PERMANENT`, or `REPLACEMENT`;
- `source_refs`: optional; IDs of relevant normative sources;
- the fields specific to its category.

The category is normative and must not be inferred from the operation name. Uncertain classification blocks translation.

### 11.1 Auto Effect

Fields:

- `timing`: required;
- `activation_conditions`: optional; one Condition expression;
- `activation_costs`: optional; ordered array;
- `usage_limit`: optional;
- `effect_form`: required: `ONE_SHOT` or `CONTINUOUS`;
- `resolution`: required when `ONE_SHOT`;
- `continuous`: required when `CONTINUOUS`.

Supported timings: `ON_PLAY`, `ON_KO`, `WHEN_ATTACKING`, `ON_OPPONENT_ATTACK`, `ON_BLOCK`, `TRIGGER`, `START_OF_GAME`, `START_OF_TURN`, `START_OF_MAIN_PHASE`, `END_OF_YOUR_TURN`, and `END_OF_OPPONENT_TURN`.

`TRIGGER` is a specific window within Life damage processing. When a card with Trigger would be added to its owner's hand from Life during this procedure, the engine offers the player the option to activate it instead of adding it to their hand. If accepted, the declared instruction resolves through the Trigger procedure; if declined, normal processing continues. If an effect or Replacement Effect prevents the card from being added to the hand, Trigger cannot be activated (CR 4-6-3-1). This is not an `OPTIONAL` step within resolution.

`START_OF_GAME` is accepted only when the official game-start procedure supports the case. If the timing event occurred but the source card left its area before the Auto Effect could be activated, the effect does not activate (CR 8-1-3-1-3).

`[On K.O.]` is special: the engine identifies the K.O. on the Field, processes the timing and activation according to the CR, moves the card to the Trash at the prescribed point, and resolves the effect with its source in the area applicable at that time. K.O. must not be replaced with generic Trash.

### 11.2 Activate Effect

Fields:

- `activation_window`: required;
- `activation_conditions`: optional; one Condition expression;
- `activation_costs`: optional; ordered array;
- `usage_limit`: optional;
- `effect_form`: required: `ONE_SHOT` or `CONTINUOUS`;
- `resolution`: required when `ONE_SHOT`;
- `continuous`: required when `CONTINUOUS`.

`activation_window` accepts `MAIN_PHASE` for an Activate Effect authorized during the Main Phase, or `EVENT_CARD_RESOLUTION` for an Event's text after the card has been activated. The `MAIN`/`COUNTER`/`TRIGGER` authorization for activating the Event card itself remains in `event_activation`.

### 11.3 Permanent Effect

Required fields: `scope` and `continuous`. `valid_when` is optional; when omitted, the effect is valid whenever the card and the rules permit it. When present, it is a Condition evaluated continuously. Do not use `resolution` to convert it into a sequence of one-time actions.

### 11.4 Replacement Effect

Required fields: `replacement`; `available_when` is optional when availability already follows from the target processing and the rules; `duration` appears only when the text/rules delimit availability.

Replacement is its own category, not an operation or a subtype of Continuous. It has no `effect_form`. `available_when` and `duration` delimit availability; application replaces part of a specific processing procedure. If the replacement procedure creates a continuous effect, that effect declares its own duration.

## 12. Conditions and Predicates

A Condition is an inline table with exactly one form: `all = [CONDITION, ...]`, `any = [CONDITION, ...]`, `not = CONDITION`, or `test = "TEST_NAME"` together with that test's fields. `all` and `any` require at least one item.

`activation_conditions`, `valid_when`, `available_when`, `filter`, and `IF.condition` accept a Condition. Conditions combined through `all` are simultaneous. The evaluation point depends on the field: before activation for activation conditions; continuously for validity; and when the step is reached for resolution.

### 12.1 Tests and Fields

| `test`                | Fields in addition to `test`                                                                                          |
| --------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `PROPERTY_COMPARE`    | `subject`, `property`, `operator`, and `value`; for `INCLUSIVE_RANGE`, use `minimum` and `maximum` instead of `value` |
| `PLAYER_TURN`         | `value` (`YOUR_TURN` or `OPPONENT_TURN`)                                                                              |
| `DON_GIVEN_TO_SOURCE` | `operator`, `value`                                                                                                   |
| `DON_IN_FIELD`        | `player`, `operator`, `value`                                                                                         |
| `AREA_COUNT`          | `area`, `player`, `operator`, `value`                                                                                 |
| `OBJECT_COUNT`        | `object`, `areas`, `player`, `operator`, `value`                                                                      |
| `SUM_COST`            | `selection`, `operator`, `value`                                                                                      |

`PROPERTY_COMPARE` properties: `CATEGORY`, `NAME`, `COLOR`, `TYPE`, `ATTRIBUTE`, `POWER`, `BASE_POWER`, `COST`, `BASE_COST`, `LEADER_LIFE`, `COUNTER`, `STATE`, `AREA`, `PLAYER_CONTEXT`, `OWNER`, `CONTROLLER`, `KEYWORD_EFFECT`, `KEYWORD`, and `DON_GIVEN_TO_CARD`. The type of `value` must match the property: numeric for Power/Cost/Life/Counter and `DON_GIVEN_TO_CARD`; string for Name/Type/Keyword; enum for Category/Color/Attribute/State/Area/Keyword Effect; PLAYER reference for Owner/Controller/Player Context. Combinations without a defined comparison are blocked.

`DON_GIVEN_TO_SOURCE` queries DON!! attached to the source instance itself; `[DON!! xX]` corresponds to quantity `GTE X`. `DON_IN_FIELD` counts DON!! in the areas that make up the Field, not DON!! attached to cards. `AREA_COUNT.area` is an area from the §14 catalog. `OBJECT_COUNT.areas` contains AREA references and `filter` is optional (if absent, count all eligible objects); when present, the filter uses `CANDIDATE` as its subject. `SUM_COST` sums the Cost of the selected set; it does not compare individual Cost values.

Operators: `EQ`, `GT`, `GTE`, `LT`, `LTE`, and `INCLUSIVE_RANGE`. For `INCLUSIVE_RANGE`, `minimum` and `maximum` must be numeric Value objects, `minimum` must not exceed `maximum`, and `value` is forbidden. For all other operators, `value` is required and `minimum`/`maximum` are forbidden. There is no `NE` or general arithmetic. Fields not listed for the test are rejected. Do not freeze the entire state at the beginning of resolution or continuously reevaluate a one-time condition.

## 13. Values

Literal and dynamic values use discriminated inline objects. Each object contains exactly one form:

- `literal`: integer, string, boolean, or enum literal;
- `property = { subject = REF, name = "BASE_POWER" }`;
- `count_area = { area = AREA_REF }`;
- `count_objects = { object = "CARD", areas = [...], player = PLAYER_REF, filter = CONDITION }`; `filter` is optional;
- `count_selection = "selection_id"`;
- `don_count = { player = PLAYER_REF, scope = "FIELD" }`;
- `don_given = { subject = REF }`;
- `sum_cost = { selection = "selection_id" }`.

`don_count.scope` accepts only `FIELD`, `COST_AREA`, or `DON_DECK`. `property.name` uses the queryable properties from §12.1. Filters use `CANDIDATE` as the subject when evaluating each examined object. `count_selection` and `sum_cost.selection` must reference a compatible earlier selection within the same resolution. `count_area.area` is an `AREA` reference. `count_objects` requires `object`, `areas`, and `player`; `filter` is optional. Dynamic values may use only these sources; there are no free variables, functions, arbitrary ID access, or general arithmetic.

## 14. References

Valid areas: `DECK`, `DON_DECK`, `HAND`, `TRASH`, `LEADER_AREA`, `CHARACTER_AREA`, `STAGE_AREA`, `COST_AREA`, `LIFE_AREA`. `NONE` is not an area.

Card states in areas with state: `ACTIVE`, `RESTED`. DON!! attached to a card has neither of these states (CR 4-4-2). Faces: `FACE_UP`, `FACE_DOWN`.

Stack positions: `TOP`, `BOTTOM`, `NORMAL` when the official procedure permits normal positioning.

References identify instances, roles, or results defined by the engine or by an earlier selection. Use exactly one of the following forms:

| `kind`                   | Additional fields | Meaning                                                           |
| ------------------------ | ----------------- | ----------------------------------------------------------------- |
| `SOURCE`                 | none              | Instance containing the ability in the activation context         |
| `EVENT_OBJECT`           | none              | Object involved in the contextual event                           |
| `PLAYER`                 | `role`            | Player role                                                       |
| `AREA`                   | `player`, `name`  | Area belonging to a player                                        |
| `SELECTED_OBJECT`        | `selection`       | Singular object obtained from an earlier selection                |
| `SELECTED_SET`           | `selection`       | Set obtained from an earlier selection                            |
| `TEXT_DESIGNATED_OBJECT` | `designation`     | Singular object designated by the text/rules                      |
| `ATTACKER`               | none              | Attacker in the battle context                                    |
| `ATTACK_TARGET`          | none              | Attack target in the battle context                               |
| `BATTLE`                 | none              | Battle context provided by the engine                             |
| `TOP_CARD`               | `player`, `area`  | Card at the top of the specified area at that moment              |
| `CANDIDATE`              | none              | Candidate object evaluated by a selection, scope, or count filter |

A `PLAYER` reference uses `{ kind = "PLAYER", role = ROLE }`; `AREA` uses `{ kind = "AREA", player = PLAYER_REF, name = AREA_NAME }`. `AREA_NAME` accepts only the areas listed above. `SELECTED_OBJECT`/`SELECTED_SET` use `selection` equal to the ID of a previous `SELECT` step. The other `kind` values accept no additional fields.

Player roles:

`YOU`, `OPPONENT`, `TURN_PLAYER`, `NON_TURN_PLAYER`, `CONTEXT_PLAYER`, `OWNER`, `CONTROLLER`, `ACTIVATOR`, `CHOOSER`, `EXECUTOR`, `AFFECTED`.

`TEXT_DESIGNATED_OBJECT.designation` must reproduce an exact expression present in `abilities.source_text` and correspond to a single instance in the indicated context. It must not be a label invented by the LLM. `CANDIDATE` may appear only in `PROPERTY_COMPARE.subject` within `SELECT.filter`, `continuous.scope.filter`, `OBJECT_COUNT.filter`, or an inline payment filter; elsewhere it is invalid.

`CONTROLLER` may be used only when the distinction is supported by the text/rules. `YOU` is not a universal substitute for `OWNER`, `CONTROLLER`, `TURN_PLAYER`, or `ACTIVATOR`.

`NONE` is not an area. A card temporarily without an area, such as during the Trigger cycle, is handled by the engine procedure, not by an artificial `NONE` area.

Selection IDs are local to an effect. A selection reference may point only to an earlier selection within the same resolution. There is no reference to a global variable, an implicit "last card", or an arbitrary instance ID.

## 15. Quantities, Selections, and Choices

### 15.1 Quantity

Supported forms:

- `{ mode = "FIXED", value = { literal = 2 } }`;
- `{ mode = "UP_TO", maximum = { literal = 2 } }`;
- `{ mode = "DYNAMIC", value = { count_area = { area = { kind = "AREA", player = { kind = "PLAYER", role = "YOU" }, name = "TRASH" } } } }`.

`FIXED` requires a positive quantity. `UP_TO` requires a non-negative limit and allows zero when permitted by the CR/text. `DYNAMIC` uses only the dynamic values from §13 and must not produce a negative quantity. There is no `PROCEDURE` mode: `DRAW` with `UP_TO` executes the official procedure of choosing to stop before each draw up to the limit. Fixed quantities, `up to`, and dynamic counts are not interchangeable.

### 15.2 Selection

A `SELECT` step declares:

- `id`, unique within the effect;
- `object`: `CARD` or `DON`;
- `areas`: one or more area references;
- `filter` when the text restricts eligibility;
- `quantity`;
- `chooser`;
- `information`;
- `information_step` when the information comes from a previous LOOK/REVEAL;
- `order_chooser` when the rules/text assign a player the choice of order.

`information` accepts `PUBLIC_INFORMATION`, `OWNER_PRIVATE_INFORMATION`, `LOOKED_BY_CHOOSER`, `REVEALED_TO_CHOOSER`, or `FACE_DOWN_UNVERIFIABLE`. `information_step` is required only for `LOOKED_BY_CHOOSER` and `REVEALED_TO_CHOOSER`, and forbidden otherwise. For `LOOKED_BY_CHOOSER`, it must reference an earlier `LOOK` whose `player` is the chooser themselves; candidates are restricted to cards from that result that remain in the examined area. For `REVEALED_TO_CHOOSER`, it must reference an earlier `REVEAL` that included the chooser in `recipients` (or omitted `recipients`, making the information public); candidates are restricted to revealed cards that remain in the indicated areas. `OWNER_PRIVATE_INFORMATION` requires the chooser to be the owner of the secret area being examined. `FACE_DOWN_UNVERIFIABLE` does not waive the CR restrictions on selecting from secret areas.

Selection does not move, reveal, K.O., discard, or otherwise alter the object. Subsequent operations reference the selection ID. A selection with singular cardinality produces `SELECTED_OBJECT`; a plural or `UP_TO` selection produces `SELECTED_SET`. IDs are local to the effect and do not survive resolution.

`order_chooser` is a PLAYER reference and appears only when the CR/text grants a player the choice. When multiple cards are placed simultaneously into an area, preserve any ordering choice required by the CR. Never rewrite simultaneous movement as an arbitrary sequence.

### 15.3 Mode Choice

`CHOOSE_MODE` contains `id`, `chooser`, and an ordered `options` array. Each option contains `id` and `steps`. Each option represents only a branch actually offered by the text. Choosing a mode is not selecting a card; chooser and executor are separate roles.

## 16. Replacement Effects — Mandatory Structure

This section fully incorporates the relevant normative content of the former Replacement Effects v1.1.1 addendum. That addendum no longer exists as a separate normative document. In the event of a conflict with older versions, this section takes precedence for FCL 1.0.0.

Every declaration with `category = "REPLACEMENT"` contains `[abilities.effects.replacement]`. `target` and `application` are descendant tables; `costs` and `procedure` are ordered arrays.

```
[[abilities]]
id = "A3"
source_text = "You may trash 1 card from your hand instead of paying the activation cost."
status = "SUPPORTED"

[[source.references]]
id = "CR-8-3-1-7"
kind = "CR"
document = "ONE_PIECE_CARD_GAME_COMPREHENSIVE_RULES"
version = "1.2.1"
section = "8-3-1-7"
url = "https://en.onepiece-cardgame.com/pdf/rule_comprehensive.pdf?20260828="

[[abilities.effects]]
id = "E3"
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

[[abilities.effects.replacement.procedure]]
id = "R1"
kind = "SELECT"
object = "CARD"
areas = [{ kind = "AREA", player = { kind = "PLAYER", role = "YOU" }, name = "HAND" }]
quantity = { mode = "FIXED", value = { literal = 1 } }
chooser = { kind = "PLAYER", role = "YOU" }
information = "OWNER_PRIVATE_INFORMATION"

[[abilities.effects.replacement.procedure]]
id = "R2"
kind = "OPERATION"
operation = "TRASH_HAND"
player = { kind = "PLAYER", role = "YOU" }
selection = "R1"
```

The example demonstrates structural form; it is `SUPPORTED` only if the current text, rules, and card context substantiate the replacement.

### 16.1 Required Fields

- `target`: the procedure that would occur, normative reference, and subject/context;
- `point`: `BEFORE_PROCESSING` or `DURING_PROCESSING`, according to the actual application point;
- `replaced_part`: the exact part being replaced;
- `available_when`: additional condition, if needed;
- `duration`: availability duration, only when defined by text/rules;
- `application`: application decision;
- `costs`: ordered payment array, empty if there are no costs;
- `procedure`: ordered sequence replacing the identified part.

`target` contains `procedure`, `rule_ref`, and `subject`. `rule_ref` must point to an ID in `[source.references]` identifying the applicable rule/FAQ. `subject` is the reference for the processing that would otherwise occur; it cannot be omitted. The declaration's location itself identifies the card, ability, effect, and source owner; the engine preserves this provenance for the normative attribution required by CR 8-1-3-4-7.

Closed catalog of `target.procedure`:

- `ACTIVATION_COST_PAYMENT` — Activation Cost payment (CR 8-3-1-7);
- `LEADER_DAMAGE_PROCESSING` — Leader damage processing (CR 4-6 and/or 7-1-4, depending on the cause);
- `CHARACTER_KO_PROCESSING` — Character K.O. procedure (CR 10-2-1);
- `EFFECT_REMOVAL_FROM_FIELD` — removal of a card from the Field by an effect, only when this broad class is explicitly expressed in the text, for example, “would be removed from the field by your opponent's effect”;
- `ORDINARY_CARD_MOVEMENT` — ordinary movement between areas that does not replace K.O., damage, drawing, Trigger, playing a card, or DON!! −X.

`replaced_part` accepts `ACTIVATION_COST_PAYMENT`, `LEADER_DAMAGE`, `LIFE_TO_HAND_DURING_DAMAGE`, `CHARACTER_KO`, `FIELD_REMOVAL_BY_EFFECT`, or `ORDINARY_CARD_MOVEMENT`, according to the following matrix:

| `target.procedure`          | Permitted `replaced_part`                     |
| --------------------------- | --------------------------------------------- |
| `ACTIVATION_COST_PAYMENT`   | `ACTIVATION_COST_PAYMENT`                     |
| `LEADER_DAMAGE_PROCESSING`  | `LEADER_DAMAGE`, `LIFE_TO_HAND_DURING_DAMAGE` |
| `CHARACTER_KO_PROCESSING`   | `CHARACTER_KO`                                |
| `EFFECT_REMOVAL_FROM_FIELD` | `FIELD_REMOVAL_BY_EFFECT`                     |
| `ORDINARY_CARD_MOVEMENT`    | `ORDINARY_CARD_MOVEMENT`                      |

These values are not interchangeable. `EFFECT_REMOVAL_FROM_FIELD` does not mean any change of area: it corresponds only to the event defined by text describing removal from the Field caused by an effect. If the text does not identify the class, block translation.

`application` contains exactly `chooser` (PLAYER_REF), the player who decides whether to apply or decline the replacement according to CR 8-1-3-4 priority. Any available replacement may be declined; if declined, the replacement does not resolve and the original processing continues without it. This choice is inherent to the Replacement Effect procedure and is not represented by an `optional` field. Do not confuse it with `OPTIONAL` during resolution, optional costs, or `UP_TO` selections.

`costs` uses the payment structure from §17. `procedure` uses steps from §§18 and 19. Step IDs are unique within the Replacement Effect. The replacement procedure must be fully executable and supported; if it cannot be performed, the replacement cannot be applied.

### 16.2 Rules Belonging to the Engine

The card declares the replacement; the engine applies CR 8-1-3-4:

1. If available and declined, the replacement does not resolve.
2. Competing replacements follow normative priority: first, the replacement associated with the card that generated the effect; next, those of the Turn Player in the order chosen; then those of the Non-Turn Player in the order chosen.
3. The same replacement cannot be applied repeatedly to the same individual processing that it has already replaced.
4. The replacement exchanges the entire indicated part, even during another resolution; afterward, the remaining text continues in the order prescribed by the CR.
5. An impossible replacement procedure cannot be applied.
6. One replacement may enable another, subject to priority and non-repetition rules.
7. The replaced part is treated as an effect activated by the owner of the Replacement Effect.

The card does not declare `priority`, a chaining algorithm, or permission to reapply the same replacement. The engine controls these procedures. The identity of the replaced event/processing is maintained by the engine; a new independent event may offer the same replacement again.

### 16.3 Activation Cost Replacement — CR 8-3-1-7

When `target.procedure = "ACTIVATION_COST_PAYMENT"`:

1. The declaration explicitly identifies the Activation Cost payment being replaced.
2. The replacement procedure is processed instead of the original payment.
3. If the Activation Cost is not paid as described in the text, the effect after `:` is not processed.
4. The original cost, replacement procedure, and resolution after `:` remain distinct elements.

The replacement does not authorize moving the payment into resolution or assuming that the text after `:` always resolves.

### 16.4 Collective Costs

There is no generic rule for making a single payment across multiple replacements. A simultaneous event affecting multiple objects, multiple individual removals, and a textual cost are distinct situations. Translation must preserve the payment quantity/unit determined by the card and official sources. When that cannot be determined, use `BLOCKED_NORMATIVE`; do not encode "pay once regardless of quantity."

### 16.5 Catalog Boundary

Replacement must not be omitted, converted into `PERMANENT`, represented as a `REPLACE` operation during resolution, or converted into generic movement without preserving the replaced processing. Cases outside this section's catalog remain `BLOCKED_CAPABILITY` or `BLOCKED_NORMATIVE`, depending on the cause.

## 17. Costs and Payments

`activation_costs` and `replacement.costs` are ordered arrays. Each payment is a discriminated inline table using `kind`; fields not defined for the given type are rejected.

| `kind`            | Required fields              | Rule                                                                                                                                                                       |
| ----------------- | ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `REST_DON`        | `kind`, `payer`, `selection` | Select and rest the indicated quantity of active DON!! in the payer's Cost Area.                                                                                           |
| `DON_MINUS`       | `kind`, `payer`, `selection` | Select the indicated quantity of DON!! in the payer's Leader, Character, and Cost Areas and return them to the DON!! Deck, according to CR 8-3-1-6; any state is eligible. |
| `REST_CARD`       | `kind`, `payer`, `selection` | Select and rest an eligible active card in the payer's context.                                                                                                            |
| `TRASH_HAND_CARD` | `kind`, `payer`, `selection` | Select a card from the payer's hand and send it to the Trash.                                                                                                              |

For payments, `selection` is its own inline table, not a `SELECT` step ID. Required fields: `object` (`CARD` or `DON`), `areas` (non-empty array of AREA_REF), `quantity` (§15.1), `chooser` (PLAYER_REF), and `information` (the §15.2 vocabulary). `filter` (Condition) is optional and, when present, uses `CANDIDATE` as its subject.

For `REST_DON`, `areas` must point only to the payer's Cost Area, `object = "DON"`, and `filter` must restrict `STATE = ACTIVE`.

For `DON_MINUS`, `areas` must point to the payer's three permitted areas.

For `REST_CARD` and `TRASH_HAND_CARD`, `quantity` must be FIXED 1. `REST_CARD.filter` must restrict `STATE = ACTIVE`. For `TRASH_HAND_CARD`, `areas` must point to the payer's Hand.

The `chooser` is the player who makes the selection; if the text/rules do not determine that player, block translation.

DON!! may only be attached through the `GIVE_DON` operation; payments do not transfer DON!! attached to another card. `REST_DON` uses active DON!!. `GIVE_DON.source_state = "RESTED"` is permitted only when official text/FAQ authorizes selecting rested DON!! from the Cost Area; EB05-057 confirms only its specific case. Once attached, DON!! is neither active nor rested (CR 4-4-2).

Mandatory rules:

- A cost to play a card is not `activation_costs`.
- Payments occur before resolution and preserve textual order.
- Costs added by other effects are processed in the normative order of the effect that added them.
- If it is impossible to pay the entire cost before activation, the effect cannot be activated.
- If payment becomes impossible during processing after activation, pay as much as possible, but the text after `:` does not resolve (CR 8-3-1-3-1).
- If an optional cost is declined, the associated effect does not activate.
- An alternative cost path is not a mode choice; do not assume alternative/additional costs are combinable.
- Activation Cost replacement is declared as a Replacement Effect and follows §16.3.

Other payment types remain `BLOCKED_CAPABILITY` until a versioned extension is introduced.

## 18. Resolution — `resolution` and `steps`

`resolution` is an ordered array of steps. Every step has an `id` unique within the effect (including nested branches) and a `kind`. Fields are closed by `kind`:

| `kind`         | Required fields beyond `id`/`kind`                      | Optional fields                               |
| -------------- | ------------------------------------------------------- | --------------------------------------------- |
| `SELECT`       | `object`, `areas`, `quantity`, `chooser`, `information` | `filter`, `information_step`, `order_chooser` |
| `OPERATION`    | `operation` and the required fields from §19.1          | Only the optional fields from §19.1           |
| `IF`           | `condition`, `steps`                                    | —                                             |
| `THEN`         | `steps`                                                 | —                                             |
| `IF_YOU_DO`    | `depends_on`, `steps`                                   | —                                             |
| `OPTIONAL`     | `chooser`, `steps`                                      | —                                             |
| `CHOOSE_MODE`  | `chooser`, `options`                                    | —                                             |
| `SIMULTANEOUS` | `steps`                                                 | —                                             |
| `REPEAT`       | `quantity`, `steps`                                     | —                                             |

`SELECT` follows §15.2. `OPERATION` uses only references valid at execution time. In `IF`, `steps` contains exactly the portion dependent on the condition; steps outside that block do not implicitly inherit the condition. `THEN` contains the steps of the corresponding textual clause and does not itself create a dependency on the previous step's success. `IF_YOU_DO.depends_on` points to a previous step that performs the cited action; the dependency is the result defined by the text/CR, not a universal rule requiring total success. If that result cannot be determined, block translation.

### 18.1 `SELECT`

Creates a selection according to §15.2. The step ID names the result and may only be referenced afterward, within the same resolution.

### 18.2 `OPERATION`

Contains `operation` and the required parameters from the §19.1 matrix. It uses only references valid at execution time.

### 18.3 `IF`

The condition is evaluated when the step is reached. Impossible actions and their consequences for subsequent steps follow the text and CR; there is no general rollback.

### 18.4 `THEN`

Contains `steps`. It marks the subsequent textual clause; it does not create a universal dependency on the immediately preceding step's success.

### 18.5 `IF_YOU_DO`

Contains `depends_on` (ID of a previous step) and `steps`. Do not define success as "all", "at least one", or "atomic action" without specific normative support.

### 18.6 `OPTIONAL`

Contains `chooser` and `steps`. Represents only an action during resolution that the text allows the player to decline. It is not an optional cost, `UP_TO` selection, mode choice, or refusal of a Replacement Effect.

### 18.7 `CHOOSE_MODE`

Contains `chooser` and a non-empty `options` array. Each option contains an `id` unique within the `CHOOSE_MODE` and `steps`, in textual order. Each branch contains only instructions belonging to that option.

### 18.8 `SIMULTANEOUS`

Contains `steps` grouped as simultaneous. The group must not be rewritten as a sequence. Ordering choices and the order of decisions between players remain determined by the CR.

### 18.9 `REPEAT`

Contains `quantity` and `steps`. Only finite repetition is permitted: a fixed quantity, an explicitly supported dynamic quantity, or `UP_TO` when authorized. There are no unrestricted loops, arbitrary stopping conditions, or recursion. The engine handles generated events, Trigger, `rule processing`, and general loops.

## 19. Supported Operations

`operation` accepts only the families below. The identifier is a semantic instruction; concrete execution is an engine procedure.

| Identifier              | Semantics                                                                                                                                                                                                                                                |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `DRAW`                  | Draw from the top of the Deck through the official procedure. `UP_TO` uses the choice to stop before each draw.                                                                                                                                          |
| `TRASH_HAND`            | Send a selected card from the Hand to the Trash; this is not K.O.                                                                                                                                                                                        |
| `TRASH_DECK`            | Send a card from the Deck to the Trash through an authorized selection/information procedure; this is not drawing.                                                                                                                                       |
| `KO_CHARACTER`          | Apply the official K.O. procedure to an eligible Character; generates a K.O. timing/event.                                                                                                                                                               |
| `MOVE_CARD`             | Ordinary movement between areas; not equivalent to K.O., damage, drawing, Trigger, playing a card, or DON!! −X.                                                                                                                                          |
| `REST`                  | Change active to rested; an already rested object remains rested.                                                                                                                                                                                        |
| `SET_ACTIVE`            | Change rested to active; an already active object remains active.                                                                                                                                                                                        |
| `GIVE_DON`              | Attach DON!! from the Cost Area to an eligible Leader/Character; `source_state` declares its state before attachment. Attached DON!! is neither active nor rested (CR 4-4-2).                                                                            |
| `ADD_DON`               | Move DON!! from the DON!! Deck to the Cost Area with a defined arrival state.                                                                                                                                                                            |
| `DON_MINUS`             | Execute the official DON!! −X procedure, selecting DON!! from the permitted Leader, Character, and Cost Areas.                                                                                                                                           |
| `PLAY_CARD`             | Invoke the official procedure for playing a card by an effect; this is not simple movement to the Field. The procedure for playing by an effect does not charge the printed Cost unless an expressly applicable additional cost applies.                 |
| `REVEAL`                | Reveal without moving `subject`, `selection`, or all cards viewed through `look_step`; `recipients` identifies who receives the information. The engine maintains the history according to the official procedure.                                       |
| `LOOK`                  | Look at the `quantity` cards from the top of the indicated ordered area, without moving them, revealing them to others, or changing their order. Produces a temporary set identified by the step's own ID and available only within the same resolution. |
| `SHUFFLE_DECK`          | Shuffle a Deck as authorized by the procedure.                                                                                                                                                                                                           |
| `MODIFY_POWER`          | Modify effective Power, not base Power; accepts negative values according to the CR.                                                                                                                                                                     |
| `SET_BASE_POWER`        | Set base Power with normative precedence; does not adjust effective Power.                                                                                                                                                                               |
| `MODIFY_COST`           | Modify effective Cost; the final Cost used by the game cannot be below zero.                                                                                                                                                                             |
| `SET_BASE_COST`         | Set base Cost with normative precedence; does not modify effective Cost.                                                                                                                                                                                 |
| `MODIFY_CHARACTERISTIC` | Modify/grant only `NAME`, `TYPE`, or `ATTRIBUTE` in this profile.                                                                                                                                                                                        |
| `GRANT_KEYWORD_EFFECT`  | Grant a recognized Keyword Effect/Keyword, with validity and duration.                                                                                                                                                                                   |
| `DEAL_LEADER_DAMAGE`    | Invoke the official procedure for dealing damage to a Leader by an effect, including Life, Trigger, replacements, and defeat.                                                                                                                            |
| `MOVE_LIFE_CARD`        | Ordinary movement involving Life; it is not damage and does not trigger Trigger by itself.                                                                                                                                                               |
| `POSITION_CARD`         | Position a card at the top/bottom of an authorized stack.                                                                                                                                                                                                |
| `RETURN_CARD`           | Return a card to the Hand, Deck, or Trash, preserving destination and owner/player.                                                                                                                                                                      |
| `END_GAME`              | Grant victory/defeat only when explicitly authorized by the text/rules.                                                                                                                                                                                  |

### 19.1 Parameters by Operation

`subject` is a singular reference from §14; `selection` is the ID of a previous `SELECT` in the same effect. When the matrix specifies `subject` or `selection`, exactly one must be present. Area references are AREA_REF, `player` is PLAYER_REF, quantities follow §15.1, and durations follow §21. No additional fields are permitted.

| Operation               | Required fields                                                       | Permitted optional fields                    |
| ----------------------- | --------------------------------------------------------------------- | -------------------------------------------- |
| `DRAW`                  | `player`, `quantity`                                                  | —                                            |
| `TRASH_HAND`            | `player`, `selection`                                                 | —                                            |
| `TRASH_DECK`            | `player`, `selection`                                                 | —                                            |
| `KO_CHARACTER`          | `subject` or `selection`                                              | —                                            |
| `MOVE_CARD`             | `subject` or `selection`, `source_area`, `destination_area`           | `face`, `state`, `position`, `order_chooser` |
| `REST` / `SET_ACTIVE`   | `subject` or `selection`                                              | —                                            |
| `GIVE_DON`              | recipient `subject` or `selection`, `quantity`, `source_state`        | —                                            |
| `ADD_DON`               | `player`, `quantity`, `arrival_state`                                 | —                                            |
| `DON_MINUS`             | `player`, `quantity`                                                  | —                                            |
| `PLAY_CARD`             | card `subject` or `selection`, `source_area`, `player`                | `state`                                      |
| `REVEAL`                | Exactly one of: `subject`, `selection`, or `look_step`                | `recipients`                                 |
| `LOOK`                  | `area`, `player`, `quantity`                                          | —                                            |
| `SHUFFLE_DECK`          | `player`                                                              | —                                            |
| `MODIFY_POWER`          | `subject` or `selection`, `operator`, `value`, `duration`             | —                                            |
| `SET_BASE_POWER`        | `subject` or `selection`, `value`, `duration`                         | —                                            |
| `MODIFY_COST`           | `subject` or `selection`, `operator`, `value`, `duration`             | —                                            |
| `SET_BASE_COST`         | `subject` or `selection`, `value`, `duration`                         | —                                            |
| `MODIFY_CHARACTERISTIC` | `subject` or `selection`, `property`, `operator`, `value`, `duration` | —                                            |
| `GRANT_KEYWORD_EFFECT`  | `subject` or `selection`, `keyword_effect`, `duration`                | —                                            |
| `DEAL_LEADER_DAMAGE`    | `player`, `quantity`                                                  | —                                            |
| `MOVE_LIFE_CARD`        | `subject` or `selection`, `source_area`, `destination_area`           | `position`, `face`, `order_chooser`          |
| `POSITION_CARD`         | `subject` or `selection`, `destination_area`, `position`              | `order_chooser`                              |
| `RETURN_CARD`           | `subject` or `selection`, `destination_area`                          | `position`                                   |
| `END_GAME`              | `player`, `outcome` (`WIN` or `LOSE`)                                 | —                                            |

`source_area`, `destination_area`, and `area` are AREA_REF, except for `AREA_COUNT.area` in §12.1, which is AREA_NAME. `recipients` is a non-empty array of PLAYER_REF; if omitted, the information is public. `LOOK` does not change the visibility of the card: information remains restricted to the player specified by the procedure; `LOOK` has no `recipients` field. `LOOK` examines the first `quantity` cards of an ordered area, creates a temporary set, and does not change the area or its order. `REVEAL.look_step` is the ID of a previous `LOOK` step and reveals all cards in that set; if `REVEAL` uses `subject` or `selection`, it reveals only the indicated object/set. `look_step` may point only to a previous `LOOK` in the same resolution.

`face` accepts `FACE_UP` or `FACE_DOWN`; `state`/`source_state`/`arrival_state` accept `ACTIVE` or `RESTED`; `position` accepts `TOP`, `BOTTOM`, or `NORMAL` when permitted by the procedure. `order_chooser` is a PLAYER_REF.

`GIVE_DON.source_state = "ACTIVE"` is the normal case; `source_state = "RESTED"` is supported only when official text/FAQ authorizes that selection. EB05-057 confirms only the specific scenario addressed. Once attached, DON!! no longer has an active/rested state, according to CR 4-4-2. `GIVE_DON` always uses DON!! from the Cost Area, never DON!! attached to another card. `ADD_DON` moves DON!! from the DON!! Deck to the Cost Area.

`MODIFY_POWER.operator` and `MODIFY_COST.operator` accept `ADD` or `SUBTRACT`. `MODIFY_CHARACTERISTIC.property` accepts only `NAME`, `TYPE`, or `ATTRIBUTE`. `operator` accepts `SET` to replace the value, `ADD` to add a TYPE/ATTRIBUTE, or `REMOVE` to remove a TYPE/ATTRIBUTE. `NAME` accepts only `SET`; `NAME`/`TYPE` values are strings and `ATTRIBUTE` is an enum from §6.1. Changing color/category is blocked.

`END_GAME` does not represent concession: concession cannot be forced by a card or replaced (CR 1-2-4). It represents only an explicit victory/defeat effect (CR 1-2-5).

`GRANT_ABILITY`, `NEGATE_EFFECT`, `PROHIBIT_ACTION`, and `MODIFY_PROCEDURE` are not operations in this version. If the text requires one of them, use `BLOCKED_CAPABILITY`.

### 19.2 Semantics That Must Not Be Collapsed

- K.O. generates the K.O. procedure/event; ordinary Trash does not generate `[On K.O.]`.
- `MOVE_LIFE_CARD` does not cause damage or offer Trigger. Trigger occurs only during the official damage procedure.
- `PLAY_CARD` processes play legality, cost/waiver, destination, and entry timing; it is not `MOVE_CARD`.
- DON!! attached to a card is different from DON!! in the Cost Area and DON!! in the DON!! Deck.
- `MODIFY_POWER` does not alter base Power. The wording "set Power to 0" has its own semantics under CR 4-12 and is neither `SET_BASE_POWER` nor `MODIFY_POWER`; because FCL 1.0.0 has no specific instruction for this procedure, it is `BLOCKED_CAPABILITY`.
- Power may become negative without moving the card. Negative effective Cost is treated as zero outside calculations; the negative value may participate in calculations according to the CR.
- Generic effect negation, granting abilities, prohibiting actions, and modifying procedures are blocked in this version; they must not be approximated through Keyword removal or `MOVE_CARD`.
- `BLOCKER` is not generic redirection; the engine executes the Keyword's procedure.

## 20. Continuous Effects — `continuous`

`continuous` represents a continuous consequence, not a sequence of actions. Exactly one of `subject` or `scope` must identify where the effect applies. Remaining fields depend on `kind`:

| `kind`                                           | Required additional fields                                   | Optional fields |
| ------------------------------------------------ | ------------------------------------------------------------ | --------------- |
| `MODIFY_POWER` / `MODIFY_COST`                   | `operator`, `value`, `duration` in Auto/Activate             | —               |
| `SET_BASE_POWER` / `SET_BASE_COST`               | `value`, `duration` in Auto/Activate                         | —               |
| `MODIFY_CHARACTERISTIC`                          | `property`, `operator`, `value`, `duration` in Auto/Activate | —               |
| `GRANT_KEYWORD_EFFECT` / `REMOVE_KEYWORD_EFFECT` | `keyword_effect`, `duration` in Auto/Activate                | —               |

`scope` accepts `{ object = "CARD"|"DON", areas = [AREA_REF, ...], player = PLAYER_REF, filter = CONDITION }`; `player` and `filter` are optional. Without `player`, the scope includes all players. In filters, `subject = { kind = "CANDIDATE" }` represents each examined object. `subject` is a singular reference from §14. `property` accepts `NAME`, `TYPE`, or `ATTRIBUTE`. `keyword_effect` accepts only a Keyword Effect listed in §10.1.

In `AUTO`/`ACTIVATE`, `duration` is required for every `continuous` effect. A `PERMANENT` effect uses optional `valid_when` (omitted = always valid, subject to area rules) and does not declare `duration` inside `continuous`. `operator` accepts `ADD` or `SUBTRACT`. `MODIFY_POWER` may produce negative Power; `MODIFY_COST` follows the CR's treatment of negative Cost. `SET_BASE_POWER` and `SET_BASE_COST` modify base values and follow normative precedence. When present for `PERMANENT`, `valid_when` is not repeated inside `continuous`.

`GRANT_ABILITY`, `NEGATE_EFFECT`, `PROHIBIT_ACTION`, and `MODIFY_PROCEDURE` are outside the executable 1.0.0 profile. If the text requires any of these capabilities, use `BLOCKED_CAPABILITY`. Replacement Effects remain exclusively `category = "REPLACEMENT"` (§16).

## 21. Supported Durations

`duration` uses an inline object with `kind` and conditional fields:

- `THIS_TURN`;
- `UNTIL_END_OF_TURN`;
- `UNTIL_END_OF_END_PHASE`;
- `THIS_BATTLE`;
- `UNTIL_START_OF_NEXT_TURN` with `player`;
- `WHILE_VALID` with `condition`;
- `PERMANENT`, only when the text/rules establish a duration without ordinary temporal expiration.

Do not normalize `THIS_TURN` and `UNTIL_END_OF_TURN`. The engine processes expirations at official checkpoints. Do not invent counters or expiration events (`next attack`, `next draw`, etc.) that are not in the catalog.

A continuous consequence created by an Auto/Activate Effect does not automatically disappear because the source card left the Field; its validity and duration are evaluated according to the text and CR. Permanent Effects depend on the ability's validity, with area exceptions only when permitted by the CR/text. The engine applies permanent-effect stabilization in the following order: Turn Player, then Non-Turn Player, repeating when the game state changes as prescribed by CR 8-1-3-3-5.

## 22. Usage Limits

`usage_limit` accepts only:

```
usage_limit = { kind = "ONCE_PER_TURN" }
```

Application is per card/instance, according to CR 10-2-13. Leaving an area and returning may create a new instance for this rule. If activation reaches the payment stage and the cost becomes impossible during processing, `[Once Per Turn]` is still consumed and cannot be activated again that turn (CR 10-2-13-5). The engine controls reset, instance identity, and the point at which the limit is consumed.

Do not admit `ONCE_PER_BATTLE`, `ONCE_PER_GAME`, `ONCE_PER_PHASE`, or `ONCE_PER_ATTACK` as generic variants.

## 23. Validation Rules

All validation layers are mandatory for compilation.

### 23.1 Syntactic

The file must be valid UTF-8 TOML 1.0.0. A parser error terminates validation.

### 23.2 Structural

- only tables/keys authorized by this document;
- correct value types; required keys present and prohibited keys absent;
- recognized enumerations;
- unique IDs within their defined scope;
- category-specific fields not mixed;
- arrays used wherever order matters;
- parameters compatible with the operation, test, payment, and step;
- `editorial.extra` is the only area for free-form, non-executable metadata;
- no `null`, duplicate keys, or TOML types outside the profile.

### 23.3 Referential

- every reference uses an authorized form from §14;
- `source_refs`, `replacement.target.rule_ref`, and `translation.blockers.references` point to existing IDs in `[source.references]`;
- selection references point to a compatible earlier `SELECT` in the same effect;
- `depends_on` points to a compatible previous step;
- area identifies a valid player and name;
- property/state is applicable to the object;
- no arbitrary instance-ID access.

### 23.4 Semantic

- correct category, timing, and window;
- conditions evaluated in the correct context and at the correct time;
- cost separated from resolution;
- selection preserves object, area, filter, cardinality, chooser, and information;
- mode choice separated from selection;
- executor separated from chooser;
- `if`, `then`, and `if you do` remain distinct;
- operations correspond to official procedures;
- ordering and simultaneity preserved;
- continuous effects have subject/scope, validity, and duration;
- Replacement Effects contain every element required by §16;
- K.O., Trash, damage, Life, Trigger, playing a card, and DON!! −X remain distinct;
- any semantics that cannot be substantiated result in blocking.

### 23.5 Normative

Interpretations that depend on the CR, FAQ, or errata must be traceable through `source_refs` or `translation.blockers.references`. A specific FAQ must not be generalized beyond the case it addresses. `raw_text` never takes precedence over the current official text in `current_text`.

## 24. Blocking Policy

Possible states are defined in §8. The translating LLM must block whenever it cannot establish, from the current text and applicable sources:

- player, owner, controller, chooser, executor, or affected player;
- timing, window, condition, or evaluation point;
- designated object, selection, cardinality, or available information;
- random selection, when the profile does not provide a specific construct;
- order, simultaneity, or dependency;
- cost, payment, or optionality;
- replaced processing, application point, replaced part, or replacement procedure;
- validity, scope, or duration;
- the consequence of `if you do`;
- card/instance identity or applicable text version.

`BLOCKED_NORMATIVE` means that normative support or official clarification is missing. `BLOCKED_CAPABILITY` means that the semantics are known, but the 1.0.0 profile has no construct to represent them. `INVALID_SOURCE` means that the text, errata, language/region, or source data are insufficient or inconsistent.

No blocker may be converted into an empty operation, default value, approximated branch, or omitted text. The LLM must never invent an enumeration, opcode, property, player, or exception.

## 25. Versioning and Compatibility

- `language_version` identifies FCL's textual structure, catalog, and semantics. The version of this specification and the language is `1.0.0`; there is no independent semantic version.
- Unknown fields are rejected, except for scalar keys in `[editorial.extra]`.
- Incompatible changes to structure, enumeration, type, or meaning require a new major version.
- A compatible extension that does not change existing meanings requires a new minor version.
- A textual correction that does not change semantics may receive a patch version of the document, but does not change `language_version`.
- A change that alters the meaning of a construct requires a new language version.
- The compiler rejects versions it does not declare support for; it must never silently reinterpret them.

## 26. Reference Examples

The following examples demonstrate TOML structure. They are syntactic examples and do not replace auditing the card and its rules.

### 26.1 Simple Permanent Effect

```
format = "FL_CARD_LANGUAGE"
language_version = "1.0.0"

[card]
number = "EXAMPLE-001"
name = "Example Leader"
category = "LEADER"
colors = ["RED"]
types = ["Example Crew"]
attributes = ["SLASH"]
power = 5000
life = 5

[source]
raw_text = "[DON!! x1] [Your Turn] All of your Characters gain +1000 power."
current_text = "[DON!! x1] [Your Turn] All of your Characters gain +1000 power."
language = "en"
region = "EN_NA_EU_OC_LATAM_ME"

[translation]
status = "SUPPORTED"

[[abilities]]
id = "A1"
source_text = "[DON!! x1] [Your Turn] All of your Characters gain +1000 power."
status = "SUPPORTED"

[[abilities.effects]]
id = "E1"
category = "PERMANENT"
valid_when = { all = [{ test = "DON_GIVEN_TO_SOURCE", operator = "GTE", value = { literal = 1 } }, { test = "PLAYER_TURN", value = "YOUR_TURN" }] }
scope = { object = "CARD", areas = [{ kind = "AREA", player = { kind = "PLAYER", role = "YOU" }, name = "CHARACTER_AREA" }], player = { kind = "PLAYER", role = "YOU" } }
continuous = { kind = "MODIFY_POWER", operator = "ADD", value = { literal = 1000 } }
```

### 26.2 Auto Effect with Ordered Resolution (Fragment)

```
[[abilities]]
id = "A2"
source_text = "On Play: Draw 1 card."
status = "SUPPORTED"

[[abilities.effects]]
id = "E2"
category = "AUTO"
timing = "ON_PLAY"
effect_form = "ONE_SHOT"
resolution = [
  { id = "S1", kind = "OPERATION", operation = "DRAW", player = { kind = "PLAYER", role = "YOU" }, quantity = { mode = "FIXED", value = { literal = 1 } } }
]
```

### 26.3 Resolution Condition After a Previous Step

```
resolution = [
  { id = "S1", kind = "SELECT", object = "CARD", areas = [{ kind = "AREA", player = { kind = "PLAYER", role = "YOU" }, name = "HAND" }], quantity = { mode = "FIXED", value = { literal = 1 } }, chooser = { kind = "PLAYER", role = "YOU" }, information = "OWNER_PRIVATE_INFORMATION" },
  { id = "S2", kind = "OPERATION", operation = "TRASH_HAND", player = { kind = "PLAYER", role = "YOU" }, selection = "S1" },
  { id = "S3", kind = "IF", condition = { test = "AREA_COUNT", area = "TRASH", player = { kind = "PLAYER", role = "YOU" }, operator = "GTE", value = { literal = 5 } }, steps = [
    { id = "S3A", kind = "OPERATION", operation = "SET_BASE_POWER", subject = { kind = "SOURCE" }, value = { literal = 5000 }, duration = { kind = "UNTIL_END_OF_TURN" } }
  ] }
]
```

The condition is evaluated when the `IF` node is processed. This example does not authorize freezing the game state at the beginning of resolution.

### 26.4 Activation Cost Replacement Effect

```
[[abilities]]
id = "A3"
source_text = "You may trash 1 card from your hand instead of paying the activation cost."
status = "SUPPORTED"

[[source.references]]
id = "CR-8-3-1-7"
kind = "CR"
document = "ONE_PIECE_CARD_GAME_COMPREHENSIVE_RULES"
version = "1.2.1"
section = "8-3-1-7"
url = "https://en.onepiece-cardgame.com/pdf/rule_comprehensive.pdf?20260828="

[[abilities.effects]]
id = "E3"
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

[[abilities.effects.replacement.procedure]]
id = "R1"
kind = "SELECT"
object = "CARD"
areas = [{ kind = "AREA", player = { kind = "PLAYER", role = "YOU" }, name = "HAND" }]
quantity = { mode = "FIXED", value = { literal = 1 } }
chooser = { kind = "PLAYER", role = "YOU" }
information = "OWNER_PRIVATE_INFORMATION"

[[abilities.effects.replacement.procedure]]
id = "R2"
kind = "OPERATION"
operation = "TRASH_HAND"
player = { kind = "PLAYER", role = "YOU" }
selection = "R1"
```

The example may be used only when the current text, rules, and card context actually support this replacement. CR 8-3-1-7 determines the consequence for the effect whose Activation Cost was replaced. The selection and operation constitute the replacement procedure; they are not declared as an additional Activation Cost.

### 26.5 Trigger Auto Effect

Structural example for a non-Event card whose Trigger text executes its own effect:

```
[[abilities]]
id = "A4"
source_text = "[Trigger] Draw 1 card."
status = "SUPPORTED"
keyword_effects = ["TRIGGER"]

[[abilities.effects]]
id = "E4"
category = "AUTO"
timing = "TRIGGER"
effect_form = "ONE_SHOT"
resolution = [
  { id = "T1", kind = "OPERATION", operation = "DRAW", player = { kind = "PLAYER", role = "YOU" }, quantity = { mode = "FIXED", value = { literal = 1 } } }
]
```

The engine offers activation during Life damage processing. This is a fragment; card/source metadata and translation status must be present in the complete file.

### 26.6 Event with a Trigger That Activates the Card Itself

For `[Trigger] Activate this card.`, use `keyword_effects = ["TRIGGER"]` and `event_activation = { window = "TRIGGER" }`. The Event's normal ability is declared separately as `ACTIVATE` with `activation_window = "EVENT_CARD_RESOLUTION"`. Do not create a `MOVE_CARD` operation to simulate activation.

## 27. Conformance

A program conforms to FCL 1.0.0 only when:

1. it is valid TOML 1.0.0;
2. it uses `format = "FL_CARD_LANGUAGE"` and `language_version = "1.0.0"`;
3. it fully complies with the structure, vocabularies, and invariants in this document;
4. it passes syntactic, structural, referential, semantic, and normative validation;
5. it contains no blocked gameplay abilities;
6. it contains no unknown fields outside `[editorial.extra]`.

`SUPPORTED` is a translation conclusion, not proof that the VM implements every game rule. Program conformance, compiler coverage, and engine coverage are three distinct checks.

## 28. Version 1.0.0 Freeze

FCL 1.0.0 freezes:

- TOML 1.0.0 as the textual format;
- one file per logical card identity;
- separation of identity, normative sources, translation, editorial data, and abilities;
- the four normative categories, including first-class Replacement Effects;
- the conditions, references, values, selections, choices, costs, resolution structures, operations, continuous effects, durations, and usage limits enumerated herein;
- Trigger and Event card activation handling;
- the complete Replacement Effect structure incorporated into §16;
- strict validation and blocking of unsupported constructs.

This version is frozen and ready for real card translation testing. It does not declare that every card in the game can be translated or that the Rules Engine is ready. Cases outside the profile remain blocked; they do not silently change the frozen version.

To change semantics or structure, open an extension proposal, document the card/text/rule requiring the capability, demonstrate why existing constructs are insufficient, define its interaction with the current profile, and publish a new version. Do not edit the meaning of FCL 1.0.0 during card translation.

## 29. Engine Boundaries and Additional Normative Semantics

This section consolidates responsibilities belonging to the engine rather than FCL instructions.

### 29.1 Identity and Roles

The printed card (card number/current text) is not the in-game instance. An instance has an area, owner, controller/context, state, DON!! relationships, and regulatory history. Changing areas may create a new instance for specific effects; the card number does not replace instance identity.

Accepted references: `SOURCE`, `EVENT_OBJECT`, `PLAYER`, `AREA`, `SELECTED_OBJECT`, `SELECTED_SET`, `TEXT_DESIGNATED_OBJECT`, `ATTACKER`, `ATTACK_TARGET`, `BATTLE`, and `TOP_CARD`. Roles: `YOU`, `OPPONENT`, `TURN_PLAYER`, `NON_TURN_PLAYER`, `CONTEXT_PLAYER`, `OWNER`, `CONTROLLER`, `ACTIVATOR`, `CHOOSER`, `EXECUTOR`, and `AFFECTED`. `YOU` is never a universal substitute for owner, controller, Turn Player, or activator. `OWNER`/`CONTROLLER` are relative to the contextual object. `TEXT_DESIGNATED_OBJECT.designation` reproduces a literal expression from `source_text` and must identify a single instance. `NONE` is not an area.

Areas: `DECK`, `DON_DECK`, `HAND`, `TRASH`, `LEADER_AREA`, `CHARACTER_AREA`, `STAGE_AREA`, `COST_AREA`, `LIFE_AREA`. Leader, Character, Stage, and Cost Areas make up the Field. States: `ACTIVE`, `RESTED`; faces: `FACE_UP`, `FACE_DOWN`; positions: `TOP`, `BOTTOM`, `NORMAL` when authorized.

DON!! in the DON!! Deck, in the Cost Area, and attached to a card are distinct positions. Attached DON!! is neither active nor rested (CR 4-4-2), is not available in the Cost Area, and cannot be transferred directly between cards. When the host card leaves the Field, the DON!! return procedure belongs to the engine.

### 29.2 Events, Activation, and Ordering

An event is a contextual game occurrence; timing is the regulatory point at which an ability may activate. The engine generates events and timings. FCL declares only timings from the §11 catalog. Turn conditions such as `YOUR_TURN`/`OPPONENT_TURN` are conditions, not events.

Auto Effect distinguishes satisfied timing, pending effect, actual activation, and resolution. If the source that satisfied the timing changes areas before activation, the Auto Effect does not activate (CR 8-1-3-1-3). An already activated/resolved effect that created a continuous consequence follows its validity/duration; leaving the area does not automatically erase it.

`[On K.O.]` is generated only by the K.O. procedure defined by the CR. Sending a card to the Trash by another means does not automatically generate this timing.

`[Trigger]` is offered only during official Life damage processing, in place of adding the card to the hand. If the player does not activate it, normal processing continues; if activated, Trigger resolution occurs at the prescribed point and the card follows the destination specified by the procedure. If an effect or Replacement Effect prevents the card from being added to the hand, Trigger cannot be activated (CR 4-6-3-1). Moving a card to/from Life through an ordinary effect does not offer Trigger. With Double Attack, damage/Trigger is processed point by point according to the CR.

Activating an Event card and activating an effect of a card are different actions. `[Main]`/`[Counter]` on an Event are activation windows for the card; text that resolves after activation is represented separately. Blocker, Counter, battle, phases, and priority are engine procedures.

### 29.3 Conditions, Selection, and Information

Activation conditions, continuous validity conditions, resolution conditions, and selection filters are distinct contexts. Activation conditions are checked before activation; resolution conditions are evaluated at the point required by the text/CR and may observe changes made by previous steps. Do not assume a snapshot of the state at the beginning of resolution.

Selection identifies eligible objects; it does not move, reveal, or affect them. A fixed quantity is not `UP_TO`; `UP_TO` allows zero when authorized. The `DRAW UP TO X` procedure allows the player to stop before each draw. An object designated by the text but absent is not the same as an empty selection. Secret areas follow the CR's information procedures; looking, revealing, and choosing are distinct actions.

When multiple players must make choices simultaneously, the Turn Player chooses first. When multiple cards are simultaneously placed in an area, the owner determines their order under the CR, unless a different rule/text applies.

### 29.4 Resolution and Impossibility

Resolution follows textual order. An impossible action does not occur; there is no general rollback of previous actions. Consequences for subsequent steps depend on the structure and the CR. `IF`, `THEN`, and `IF_YOU_DO` remain distinct. Independent instructions continue when the text/CR makes them independent.

The cost to play a card is distinct from Activation Cost. If a cost is impossible before activation, the effect does not activate. If payment becomes impossible during processing after activation, pay as much as possible, resolution does not occur, and `[Once Per Turn]` is consumed when applicable. Declining a voluntary cost prevents activation. Mode choice, `UP_TO` selection, and optional actions during resolution are not costs.

### 29.5 Continuous Effects, Negation, and Prohibition

A Permanent Effect remains valid while its condition and the rules permit it. Some effects phrased as "according to/under the rules" may remain valid in secret areas; this applies only when the text/CR identifies the exception. Continuous effects are applied/stabilized by the engine in Turn Player, Non-Turn Player order, repeating if the game state changes (CR 8-1-3-3-5).

Prohibiting an action, negating/invalidating an effect, and an operation being impossible are different phenomena. FCL 1.0.0 has no generic instructions for the first two; when the text requires these behaviors, block translation. The VM must not simulate them through Keyword removal or movement operations.

### 29.6 Keywords and Their Own Procedures

Keyword Effects in the profile: Rush, Double Attack, Banish, Blocker, Trigger, Rush: Character, and Unblockable. Each term invokes its official procedure. Other Keywords are encoded according to their function: `ON_PLAY`, `ON_KO`, `WHEN_ATTACKING`, `ON_BLOCK`, `ON_OPPONENT_ATTACK`, `MAIN`, `COUNTER`, `DON!! xX`, `DON!! −X`, `YOUR_TURN`, `OPPONENT_TURN`, `ONCE_PER_TURN`, and `TRASH`.

The language does not provide a generic operation to transfer control, copy any effect, redirect attacks, cancel attacks/battles, arbitrarily modify engine rules, or repeat resolution indefinitely. Blocker is not generic redirection; ending a battle is a consequence of the battle procedure.

### 29.7 Incorporated FAQ Cases and Boundary

The following FAQs have been incorporated only for the cases they address; they must not be generalized beyond those cases:

- `EB05-006`: Looking at the top Life card before choosing one of the modes is permitted in the specific case addressed.
- `EB05-008`: A Counter Effect with `up to 1` may be activated even when there is no Character on the Field.
- `EB05-011`: The specific On Play condition is not satisfied when the top Life card is already face-down in the case addressed.
- `EB05-028`: The activator's opponent chooses the four cards from their own hand to Trash.
- `EB05-033`: The Leader type requirement and the difference in DON!! in the Field are mandatory conditions.
- `EB05-034`: The subsequent condition is evaluated after DON!! −2 and may no longer be satisfied.
- `EB05-045`: On Play may K.O. its own source and, in the case addressed, may choose not to K.O. any card.
- `EB05-046`: Trash count may be evaluated after the discard performed in the previous step.
- `EB05-052`: Gloriosa's damage replacement covers the three scenarios addressed: battle damage with zero Life, both instances of Double Attack damage, and damage from an effect.
- `EB05-057`: Allows rested DON!! taken from the Cost Area to be attached in the specific scenario involving a Leader/Character with multiple Attributes, including Special or Wisdom. `GIVE_DON.source_state` records the state before attachment; attached DON!! becomes neither active nor rested (CR 4-4-2). This FAQ does not create general permission.
- `EB05-059`: Immunity to K.O. in battle does not apply in the specific scenario addressed.
- `EB05-061`: Nami's replacement may be used when its own source would be removed, but not in the scenario where there is no Life to perform the consequence.
- `EB05-010`: Negation of printed, continuously granted, or Trigger-granted Blocker follows the specific treatment described by the FAQ.
- `EB05-021`: On Play of a Character played by an effect does not automatically chain into another On Play in the scenario addressed.

General questions not yet covered, including unverified historical FAQs, complete errata reconciliation, collective replacement costs, complex Replacement Effect interactions, unclear evaluation points, effects in secret areas, loops, and regional/language conflicts, remain outside the profile. A specific case depending on an unverified answer must be `BLOCKED_NORMATIVE`. Collective replacement costs deliberately remain without a generic rule.

## Official References

- TOML 1.0.0: [https://toml.io/en/v1.0.0](https://toml.io/en/v1.0.0?utm_source=chatgpt.com)
- ONE PIECE CARD GAME Comprehensive Rules v1.2.1, updated August 28, 2026: [https://en.onepiece-cardgame.com/pdf/rule_comprehensive.pdf?20260828=](https://en.onepiece-cardgame.com/pdf/rule_comprehensive.pdf?20260828\&utm_source=chatgpt.com)
- Official Rules and Errata Index: [https://en.onepiece-cardgame.com/rules/](https://en.onepiece-cardgame.com/rules/?utm_source=chatgpt.com)
- Official FAQ Index: [https://en.onepiece-cardgame.com/rules/faq/](https://en.onepiece-cardgame.com/rules/faq/?utm_source=chatgpt.com)
- Official EB-05 FAQ, updated November 6, 2026: [https://en.onepiece-cardgame.com/pdf/qa_eb05.pdf?20261106=](https://en.onepiece-cardgame.com/pdf/qa_eb05.pdf?20261106\&utm_source=chatgpt.com)

## Certification Boundary

This document establishes a conservative profile of supported constructs. It does not certify that every card, every historical FAQ, every erratum, or every possible interaction has been audited. The criterion is: behavior defined by the profile may be translated; behavior that depends on an unverified rule or a missing construct must be blocked.

Final Rule: The LLM interprets and declares; the validator checks structure and constraints; the compiler transforms only `SUPPORTED` programs; the VM executes valid instructions; the Rules Engine applies the official procedures. None of these stages may invent semantics.
