# FL Card Language (FCL) — Especificação 1.0.0

**Documento normativo único:** `FL_CARD_LANGUAGE_SPEC_v1.0.0.md`  
**Formato textual:** TOML 1.0.0  
**Versão da linguagem e da semântica:** 1.0.0  
**Data de congelamento:** 8 de novembro de 2026  
**Status:** `FROZEN — READY FOR CARD-TRANSLATION TESTING`

---

## 1. Objetivo e autoridade

A **FL Card Language (FCL)** é uma linguagem declarativa para descrever habilidades e efeitos de cartas do One Piece Card Game como programas de dados interpretáveis por uma Rules Engine/VM.

A carta declara o comportamento particular; as regras oficiais determinam os procedimentos e consequências; o motor executa esses procedimentos. FCL não é uma linguagem de propósito geral, não contém código Kotlin, funções de usuário, callbacks, variáveis mutáveis, loops livres, recursão ou acesso arbitrário ao estado.

A versão 1.0.0 define, em um único documento, o formato TOML canônico, o perfil semântico fechado, os vocabulários, a estrutura de Replacement Effects, as regras de validação e a política de bloqueio. **Este documento substitui, para FCL 1.0.0, qualquer especificação, contrato semântico ou adendo anterior.** Não há documento externo que precise ser combinado com ele.

A gramática lexical e sintática externa é TOML 1.0.0. FCL acrescenta um perfil fechado de tabelas, campos, tipos, enumerações e invariantes semânticos. `language_version = "1.0.0"` identifica tanto a estrutura quanto o significado das construções. Não existe um campo independente de perfil semântico.

`FROZEN` significa que o contrato desta versão está fechado para iniciar traduções e testes de cartas. Não significa que toda carta publicada seja suportada, que o compilador/VM já exista ou que toda interação do jogo tenha sido implementada. Qualquer comportamento fora deste perfil deve ser bloqueado, não aproximado. Mudanças de estrutura ou semântica exigem nova versão.

## 2. Escopo e fronteira de responsabilidade

A base normativa é o *ONE PIECE CARD GAME Comprehensive Rules* v1.2.1 (28/08/2026), em conjunto com o texto oficial vigente, erratas e FAQs diretamente aplicáveis. As fontes oficiais e o limite de auditoria estão no fim deste documento.

### 2.1 Incluído

- identidade de jogo impressa e dados editoriais separados;
- Auto, Activate, Permanent e Replacement Effects;
- Keyword Effects reconhecidos e seus procedimentos oficiais;
- condições de ativação/validade/resolução, filtros e predicados;
- seleções, quantidades, escolhas de modo e informação;
- custos de ativação e pagamentos admitidos;
- resolução ordenada, dependências, escolhas opcionais, simultaneidade e repetição finita;
- operações de jogo enumeradas neste perfil;
- efeitos contínuos, escopo, validade e duração;
- referência à origem, rastreabilidade e bloqueios de tradução.

### 2.2 Excluído

- regras completas de turno, fases, batalha, Blocker, Counter, dano, Life, Trigger, K.O., prioridade, `rule processing`, derrota e loops gerais;
- implementação de procedimentos da Rules Engine/VM, bytecode e layout binário;
- legalidade de formatos, banlists e construção de deck;
- operações, condições, timings, durações ou propriedades que não estejam enumerados neste documento;
- comportamento arbitrário, código por carta ou uma instrução exclusiva para uma carta.

Restrições de construção de deck e efeitos que alterem regras de deck construction ficam fora do programa de gameplay FCL 1.0.0. Dados editoriais de legalidade podem ser armazenados fora das instruções executáveis.

### 2.3 Princípios semânticos obrigatórios

1. **Categoria e duração são dimensões diferentes.** As quatro categorias são `AUTO`, `ACTIVATE`, `PERMANENT` e `REPLACEMENT`. `ONE_SHOT` e `CONTINUOUS` descrevem a forma temporal do efeito, não uma quinta categoria. Permanent Effect é uma categoria; Continuous Effect é uma duração/forma de consequência.
2. **Ativação, custo, escolha, seleção e resolução são diferentes.** O momento em que um timing ocorre, a elegibilidade, a ativação, o pagamento, a escolha de modo, a seleção de objetos e a resolução não podem ser fundidos.
3. **Impossibilidade não é falha universal.** Condição não satisfeita, custo impossível, ação impossível, alvo ausente, seleção vazia, escolha `up to 0`, substituição recusada e efeito invalidado são situações diferentes. Não existe rollback, atomicidade ou continuidade universal.
4. **Não inferir equivalência.** K.O. não é Trash; dano não é movimento de Life; compra não é selecionar do Deck; jogar carta não é mover para o Field; Trigger não é qualquer movimento de Life; Power efetivo não é base Power; Cost efetivo não é base Cost; owner não é controller; escolhedor não é executor.
5. **Nenhuma semântica é criada pela sintaxe.** Quando texto/regra/FAQ não determinar sujeito, objeto, jogador, momento, quantidade, ordem, custo, escopo ou consequência, a tradução é bloqueada.
6. **A carta não reimplementa o motor.** Prioridade, fases, batalha, processamento de dano, K.O., Trigger, estabilização de permanentes, regras de informação, substituições concorrentes e loops são procedimentos do motor.

Uma estrutura TOML válida pode continuar semanticamente incorreta. O status `SUPPORTED` exige validação estrutural e fundamento normativo, não apenas parsing.

## 3. Unidade de autoria e organização externa

### 3.1 Uma unidade lógica por carta

A unidade canônica de autoria é um arquivo TOML por identidade lógica de carta (`card number`). Variações de arte e printings não duplicam programas quando compartilham a mesma identidade e o mesmo texto/regra de jogo.

Uma unidade de carta possui cinco áreas conceituais, todas no mesmo documento TOML:

1. identidade de jogo (`[card]`);
2. habilidades e programas (`[[abilities]]` e estruturas descendentes);
3. fonte normativa (`[source]` e estruturas descendentes);
4. resultado de tradução e bloqueios (`[translation]`);
5. metadados editoriais e printings (`[editorial]`).

A separação é semântica e estrutural. Ela não exige arquivos físicos separados.

### 3.2 Dados que não são instruções

Identidade de jogo, proveniência e dados editoriais não são instruções executáveis. O compilador os valida e os classifica separadamente.

Metadados editoriais não devem ser enviados à VM como instruções. A fonte normativa é necessária para auditoria, mas não precisa integrar o catálogo binário de execução.

## 4. Regras TOML obrigatórias

A implementação deve aceitar TOML 1.0.0 e rejeitar extensões de versões posteriores.

| Regra | Exigência |
|---|---|
| Codificação | UTF-8 |
| Chaves | `snake_case`, sensíveis a maiúsculas/minúsculas |
| Enumerações | Strings em `UPPER_SNAKE_CASE` |
| IDs locais | Strings estáveis, únicas no escopo definido |
| Arrays | Ordem significativa quando representam sequência, alternativas ou printings |
| Tabelas | Ordem física das tabelas/chaves não possui significado semântico |
| Valores ausentes | Omitir chave não aplicável; TOML não usa `null` |
| Chaves desconhecidas | Rejeitar |
| Chaves duplicadas | Rejeitar pelo parser TOML |
| Datas | Strings ISO `YYYY-MM-DD`, não valores datetime TOML |
| Números de jogo | Inteiros decimais, salvo quando o tipo textual exigir string |
| Comentários | Permitidos, sem significado de jogo |

Não usar anchors, aliases, tags customizadas ou recursos de YAML. Não usar uma string de texto livre para substituir estrutura semântica exigida.

### 4.1 Ordem

A ordem de resolução deve estar em arrays TOML, nunca na ordem física de tabelas ou propriedades. Arrays de condições representam conjunção somente quando o campo que os contém declara explicitamente essa semântica; nos demais contextos, a interpretação é definida pelo tipo do campo.

### 4.2 Inline tables

Inline tables são usadas para referências, valores dinâmicos, condições e nós compactos de estruturas aninhadas. Cada inline table segue a sintaxe TOML 1.0.0. Estruturas de resolução extensas podem ser organizadas como arrays de tabelas (`[[...]]`).

## 5. Estrutura de topo

As únicas chaves/tabelas de topo permitidas são:

- `format`;
- `language_version`;
- `[card]`;
- `[source]`;
- `[translation]`;
- `[editorial]`;
- `[[abilities]]`.

Exemplo estrutural mínimo:

```toml
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

O exemplo é estrutural. Campos obrigatórios, aplicabilidade e vocabulários são especificados nas seções seguintes.

## 6. Identidade de jogo — `[card]`

Campos permitidos:

| Campo | Tipo | Regra |
|---|---|---|
| `number` | string | Obrigatório; card number oficial |
| `name` | string | Obrigatório |
| `category` | enum | Obrigatório: `LEADER`, `CHARACTER`, `EVENT`, `STAGE`, `DON` |
| `colors` | array de enums | Obrigatório; uma ou mais cores impressas |
| `types` | array de strings | Tipos impressos; vazio somente se oficialmente sem tipo |
| `attributes` | array de enums | Aplicável somente a Leader/Character |
| `cost` | integer | Aplicável a Character/Event/Stage |
| `power` | integer | Aplicável a Leader/Character |
| `life` | integer | Aplicável a Leader |
| `counter` | integer | Aplicável a Character |

### 6.1 Enumerações

`category`: `LEADER`, `CHARACTER`, `EVENT`, `STAGE`, `DON`.

`colors`: `RED`, `GREEN`, `BLUE`, `PURPLE`, `BLACK`, `YELLOW`.

`attributes`: `SLASH`, `STRIKE`, `RANGED`, `SPECIAL`, `WISDOM`, `QUESTION_MARK`.

`types` são dados literais impressos, não enumerações fechadas da linguagem. Preservar grafia e conteúdo oficiais usados pela fonte escolhida.

### 6.2 Aplicabilidade

- `life` só pode existir para `LEADER`.
- `power` só pode existir para `LEADER` e `CHARACTER`.
- `counter` só pode existir para `CHARACTER`.
- `cost` só pode existir para `CHARACTER`, `EVENT` e `STAGE`.
- `attributes` só pode existir para `LEADER` e `CHARACTER`.
- Não preencher campo não aplicável com zero ou string vazia.

A ausência de campo não aplicável é diferente de valor numérico zero.

## 7. Fonte normativa — `[source]`

Campos obrigatórios: `raw_text`, `current_text`, `language` e `region`. `edition` é opcional. `raw_text` preserva a origem recebida; `current_text` contém o texto oficial vigente realmente traduzido. Errata aplicável deve estar refletida em `current_text`; `raw_text` nunca é autoridade quando diverge da fonte vigente.

Todas as fontes usadas na interpretação — Comprehensive Rules, FAQ, errata e texto oficial da carta — ficam em uma única lista `[[source.references]]`. Não há uma segunda tabela de erratas.

| Campo | Tipo | Regra |
|---|---|---|
| `id` | string | Obrigatório, único no arquivo |
| `kind` | enum | `CR`, `FAQ`, `ERRATA` ou `OFFICIAL_CARD_TEXT` |
| `document` | string | Obrigatório; nome identificável da fonte |
| `url` | string | Obrigatório; URL oficial |
| `version` | string | Opcional; versão/edição da fonte |
| `section` | string | Opcional; seção/regra/pergunta relevante |
| `date` | string | Opcional; data ISO `YYYY-MM-DD`, especialmente para errata/FAQ |

```toml
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

`source_refs`, `replacement.target.rule_ref` e `translation.blockers.references` apontam para IDs existentes nesta lista. Uma FAQ só fundamenta o caso que respondeu; não generalizar seu resultado.

## 8. Resultado da tradução — `[translation]`

`[translation]` contém `status` (obrigatório) e `notes` (opcional, sem semântica executável). `status` aceita `SUPPORTED`, `BLOCKED_NORMATIVE`, `BLOCKED_CAPABILITY`, `INVALID_SOURCE` ou `NON_GAME_TEXT`.

Bloqueios usam `[[translation.blockers]]`:

| Campo | Tipo | Obrigatoriedade |
|---|---|---|
| `code` | enum | Obrigatório: `BLOCKED_NORMATIVE`, `BLOCKED_CAPABILITY` ou `INVALID_SOURCE` |
| `scope` | enum | Obrigatório: `CARD`, `ABILITY` ou `EFFECT` |
| `ability_id` | string | Obrigatório para `ABILITY`/`EFFECT`; proibido para `CARD` |
| `effect_id` | string | Obrigatório somente para `EFFECT`; proibido nos demais |
| `source_excerpt` | string | Obrigatório; trecho literal de `current_text` |
| `reason` | string | Obrigatório; lacuna objetiva |
| `references` | array de strings | Opcional; IDs de `[source.references]` |

```toml
# Fragmento: pressupõe [source.references] com o ID usado em references.
[translation]
status = "BLOCKED_CAPABILITY"

[[translation.blockers]]
code = "BLOCKED_CAPABILITY"
scope = "ABILITY"
ability_id = "A2"
source_excerpt = "..."
reason = "O texto exige uma operação não definida no perfil 1.0.0."
references = ["CR-8-1-3-4"]
```

### 8.1 Consistência

- Somente `SUPPORTED` pode ser compilado para gameplay.
- Qualquer habilidade de jogo bloqueada bloqueia a carta inteira; não compilar parcialmente.
- A prioridade de status é `INVALID_SOURCE` > `BLOCKED_NORMATIVE` > `BLOCKED_CAPABILITY` > `SUPPORTED`.
- Uma habilidade `SUPPORTED` deve estar integralmente representada; `NON_GAME_TEXT` não contém instrução executável.
- `translation.status = "NON_GAME_TEXT"` somente quando não há habilidade de gameplay traduzível (inclusive carta sem habilidade).
- `SUPPORTED` e `NON_GAME_TEXT` não podem ter blockers. Estados `BLOCKED_*` e `INVALID_SOURCE` exigem ao menos um blocker.
- Se existir ao menos uma habilidade de gameplay, `translation.status` é o status de maior severidade entre as habilidades de gameplay e os blockers de escopo `CARD`: `INVALID_SOURCE` > `BLOCKED_NORMATIVE` > `BLOCKED_CAPABILITY` > `SUPPORTED`. Habilidades `NON_GAME_TEXT` não entram nessa comparação. Use `translation.status = "NON_GAME_TEXT"` somente quando a carta não possuir habilidade de gameplay.
- Todo blocker deve corresponder a uma habilidade/efeito existente quando o escopo for `ABILITY`/`EFFECT`; status e blockers devem concordar.
- `notes` não substitui `reason` nem altera a execução.

## 9. Metadados editoriais — `[editorial]`

`[editorial]` é opcional e nunca é enviado à VM. Campos permitidos: `rarity`, `set_id`, `set_name`, `illustrator`, `thumbnail_url`, `rights`, `block_number`; `[[editorial.printings]]`; e `[editorial.extra]`.

Cada printing aceita somente `set_id`, `rarity`, `art_id`, `illustrator`, `thumbnail_role`, `thumbnail_url` e `rights`. `thumbnail_role`: `SAMPLE`, `CUSTOM` ou `FINAL`. `rights` é anotação, não licença.

`[editorial.extra]` é a única área extensível: aceita valores escalares TOML (`string`, `integer`, `float`, `boolean`) e arrays homogêneos desses escalares. Não aceita tabelas, arrays de tabelas ou dados executáveis. Todas as demais chaves desconhecidas são rejeitadas.

```toml
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

Variações de arte/printing não duplicam o programa quando compartilham identidade lógica e texto de jogo vigente.

## 10. Habilidades — `[[abilities]]`

Cada `[[abilities]]` representa uma unidade lógica do texto de jogo. Campos:

- `id`: obrigatório e único dentro da carta;
- `source_text`: obrigatório; trecho exato do texto oficial vigente correspondente à habilidade;
- `status`: obrigatório; `SUPPORTED`, `BLOCKED_NORMATIVE`, `BLOCKED_CAPABILITY`, `INVALID_SOURCE` ou `NON_GAME_TEXT`;
- `source_refs`: opcional; IDs de `[source.references]` que fundamentam a interpretação;
- `keyword_effects`: opcional; Keyword Effects presentes e reconhecidos;
- `event_activation`: opcional; modo de ativação da carta Event, distinto da categoria do efeito;
- `effects`: zero ou mais declarações em `[[abilities.effects]]`.

Uma habilidade `SUPPORTED` precisa estar integralmente representada por `effects`, `keyword_effects` ou `event_activation`. `NON_GAME_TEXT` não pode conter instruções executáveis. Uma habilidade bloqueada pode preservar a parte compreendida, mas bloqueia a carta inteira e exige blocker correspondente.

Não fundir habilidades distintas nem dividir uma habilidade quando isso alterar custo, timing, condição, escolha, ordem ou dependência.

### 10.1 Keyword Effects

`keyword_effects` aceita somente `RUSH`, `DOUBLE_ATTACK`, `BANISH`, `BLOCKER`, `TRIGGER`, `RUSH_CHARACTER` e `UNBLOCKABLE`. Cada termo aciona o procedimento oficial do motor; FCL não reimplementa esse procedimento.

A ocorrência de `TRIGGER` deve corresponder exatamente a uma representação: (a) em carta não-Event, `keyword_effects = ["TRIGGER"]` mais `AUTO/timing = "TRIGGER"`; (b) em Event com texto `[Trigger] Activate this card.`, `keyword_effects = ["TRIGGER"]` mais `event_activation.window = "TRIGGER"`. Não declarar ambas na mesma ocorrência nem registrar apenas a keyword.

Outros Keywords não são armazenados como uma lista indiferenciada. São representados em sua função semântica: timing, condição, custo, limite, operação ou restrição.

### 10.2 Event card activation

`event_activation` descreve a autorização/procedimento de ativar a carta Event, não o Activation Cost nem uma operação de movimento. Formato:

```toml
event_activation = { window = "MAIN" }
```

`window` aceita `MAIN`, `COUNTER` ou `TRIGGER`. `TRIGGER` só é usado quando o texto `[Trigger]` da Event diz para ativar a própria carta. Se o Trigger contém um efeito próprio (e não “Activate this card”), esse efeito é representado como `AUTO` com `timing = "TRIGGER"`, não como `event_activation.window = "TRIGGER"`.

A janela e o procedimento de ativação da Event pertencem ao motor. O efeito normal da Event é declarado separadamente como `ACTIVATE` com `activation_window = "EVENT_CARD_RESOLUTION"`. A ativação da carta Event não pode ser confundida com a ativação de uma habilidade de outra carta.

## 11. Declaração de efeitos — `[[abilities.effects]]`

Cada `[[abilities.effects]]` possui:

- `id`: obrigatório e único dentro da habilidade;
- `category`: obrigatório: `AUTO`, `ACTIVATE`, `PERMANENT` ou `REPLACEMENT`;
- `source_refs`: opcional; IDs de fontes normativas relevantes;
- os campos próprios da categoria.

A categoria é normativa e não é inferida pelo nome da operação. Uma classificação incerta bloqueia a tradução.

### 11.1 Auto Effect

Campos:

- `timing`: obrigatório;
- `activation_conditions`: opcional; uma expressão Condition;
- `activation_costs`: opcional; array ordenado;
- `usage_limit`: opcional;
- `effect_form`: obrigatório: `ONE_SHOT` ou `CONTINUOUS`;
- `resolution`: obrigatório quando `ONE_SHOT`;
- `continuous`: obrigatório quando `CONTINUOUS`.

Timings admitidos: `ON_PLAY`, `ON_KO`, `WHEN_ATTACKING`, `ON_OPPONENT_ATTACK`, `ON_BLOCK`, `TRIGGER`, `START_OF_GAME`, `START_OF_TURN`, `START_OF_MAIN_PHASE`, `END_OF_YOUR_TURN` e `END_OF_OPPONENT_TURN`.

`TRIGGER` é uma janela específica do processamento de dano de Life. Quando uma carta com Trigger seria adicionada à mão a partir da Life nesse procedimento, o motor oferece ao jogador a opção de ativá-lo em vez de adicioná-la à mão. Se aceitar, a instrução declarada resolve pelo procedimento de Trigger; se recusar, o processamento normal continua. Se um efeito ou Replacement Effect impedir que a carta seja adicionada à mão, o Trigger não pode ser ativado (CR 4-6-3-1). Isso não é um passo `OPTIONAL` dentro da resolução.

`START_OF_GAME` só é aceito quando o procedimento oficial de início de jogo dá suporte ao caso. Se o evento de timing ocorreu, mas a carta-fonte saiu da área antes de o Auto Effect poder ser ativado, o efeito não ativa (CR 8-1-3-1-3).

`[On K.O.]` é especial: o motor identifica o K.O. no Field, processa o timing e a ativação conforme o CR, move a carta para a Trash no momento regulamentar e resolve o efeito com a origem na área então aplicável. K.O. não pode ser substituído por Trash genérico.

### 11.2 Activate Effect

Campos:

- `activation_window`: obrigatório;
- `activation_conditions`: opcional; uma expressão Condition;
- `activation_costs`: opcional; array ordenado;
- `usage_limit`: opcional;
- `effect_form`: obrigatório: `ONE_SHOT` ou `CONTINUOUS`;
- `resolution`: obrigatório quando `ONE_SHOT`;
- `continuous`: obrigatório quando `CONTINUOUS`.

`activation_window` aceita `MAIN_PHASE` para Activate Effect autorizado na Main Phase ou `EVENT_CARD_RESOLUTION` para o texto de uma Event depois de sua ativação. A autorização `MAIN`/`COUNTER`/`TRIGGER` para ativar a própria carta Event permanece em `event_activation`.

### 11.3 Permanent Effect

Campos obrigatórios: `scope` e `continuous`. `valid_when` é opcional; quando omitido, o efeito é válido sempre que a carta e as regras permitirem. Quando presente, é uma Condition avaliada continuamente. Não usar `resolution` para convertê-lo em sequência de ações pontuais.

### 11.4 Replacement Effect

Campos obrigatórios: `replacement`; `available_when` é opcional quando a disponibilidade já decorre do processamento-alvo e das regras; `duration` só aparece se o texto/regra delimitar a disponibilidade.

Replacement é categoria própria, não operação e não subtipo de Continuous. Não possui `effect_form`. `available_when` e `duration` delimitam a disponibilidade; a aplicação substitui uma parte de um processamento concreto. Se o procedimento substituto criar efeito contínuo, esse efeito declara sua própria duração.

## 12. Condições e predicados

Uma Condition é uma tabela inline com exatamente uma forma: `all = [CONDITION, ...]`, `any = [CONDITION, ...]`, `not = CONDITION` ou `test = "TEST_NAME"` com os campos do teste. `all` e `any` exigem ao menos um item.

`activation_conditions`, `valid_when`, `available_when`, `filter` e `IF.condition` recebem uma Condition. Condições combinadas por `all` são simultâneas. O ponto de avaliação depende do campo: ativação antes de ativar; validade continuamente; resolução quando o passo é alcançado.

### 12.1 Testes e campos

| `test` | Campos além de `test` |
|---|---|
| `PROPERTY_COMPARE` | `subject`, `property`, `operator` e `value`; com `INCLUSIVE_RANGE`, usar `minimum` e `maximum` no lugar de `value` |
| `PLAYER_TURN` | `value` (`YOUR_TURN` ou `OPPONENT_TURN`) |
| `DON_GIVEN_TO_SOURCE` | `operator`, `value` |
| `DON_IN_FIELD` | `player`, `operator`, `value` |
| `AREA_COUNT` | `area`, `player`, `operator`, `value` |
| `OBJECT_COUNT` | `object`, `areas`, `player`, `operator`, `value` |
| `SUM_COST` | `selection`, `operator`, `value` |

Propriedades de `PROPERTY_COMPARE`: `CATEGORY`, `NAME`, `COLOR`, `TYPE`, `ATTRIBUTE`, `POWER`, `BASE_POWER`, `COST`, `BASE_COST`, `LEADER_LIFE`, `COUNTER`, `STATE`, `AREA`, `PLAYER_CONTEXT`, `OWNER`, `CONTROLLER`, `KEYWORD_EFFECT`, `KEYWORD` e `DON_GIVEN_TO_CARD`. O tipo de `value` deve corresponder à propriedade: numérico para Power/Cost/Life/Counter e `DON_GIVEN_TO_CARD`; string para Name/Type/Keyword; enum para Category/Color/Attribute/State/Area/Keyword Effect; referência PLAYER para Owner/Controller/Player Context. Combinações sem comparação definida ficam bloqueadas.

`DON_GIVEN_TO_SOURCE` consulta DON!! dados à própria instância-fonte; `[DON!! xX]` corresponde a quantidade `GTE X`. `DON_IN_FIELD` conta DON!! nas áreas que formam o Field, não os DON!! dados à carta. `AREA_COUNT.area` é uma área do catálogo da §14. `OBJECT_COUNT.areas` contém referências AREA e `filter` é opcional (ausente = contar todos os objetos elegíveis); quando presente, o filtro usa `CANDIDATE` como sujeito. `SUM_COST` soma Cost do conjunto selecionado, não compara o Cost individual.

Operadores: `EQ`, `GT`, `GTE`, `LT`, `LTE` e `INCLUSIVE_RANGE`. Para `INCLUSIVE_RANGE`, `minimum` e `maximum` são objetos Value numéricos, `minimum` não pode exceder `maximum`, e `value` é proibido. Para os demais operadores, `value` é obrigatório e `minimum`/`maximum` são proibidos. Não há `NE` nem aritmética geral. Campos não listados para o teste são rejeitados. Não congelar todo o estado no início da resolução nem reavaliar continuamente uma condição pontual.

## 13. Valores

Valores literais e dinâmicos usam objetos inline discriminados. Cada objeto contém exatamente uma forma:

- `literal`: inteiro, string, booleano ou enum literal;
- `property = { subject = REF, name = "BASE_POWER" }`;
- `count_area = { area = AREA_REF }`;
- `count_objects = { object = "CARD", areas = [...], player = PLAYER_REF, filter = CONDITION }`; `filter` é opcional;
- `count_selection = "selection_id"`;
- `don_count = { player = PLAYER_REF, scope = "FIELD" }`;
- `don_given = { subject = REF }`;
- `sum_cost = { selection = "selection_id" }`.

`don_count.scope` aceita somente `FIELD`, `COST_AREA` ou `DON_DECK`. `property.name` usa as propriedades consultáveis da seção 12.1. Filtros usam `CANDIDATE` como sujeito para avaliar cada objeto examinado. `count_selection` e `sum_cost.selection` apontam para seleção anterior compatível na mesma resolução. `count_area.area` é uma referência `AREA`. `count_objects` exige `object`, `areas` e `player`; `filter` é opcional. Valores dinâmicos usam somente essas fontes; não há variável livre, função, acesso por ID arbitrário ou aritmética geral.

## 14. Referências

Áreas válidas: `DECK`, `DON_DECK`, `HAND`, `TRASH`, `LEADER_AREA`, `CHARACTER_AREA`, `STAGE_AREA`, `COST_AREA`, `LIFE_AREA`. `NONE` não é área.

Estados de cartas em áreas com estado: `ACTIVE`, `RESTED`. DON!! anexado a uma carta não possui nenhum desses estados (CR 4-4-2). Faces: `FACE_UP`, `FACE_DOWN`.

Posições em pilhas: `TOP`, `BOTTOM`, `NORMAL` quando o procedimento oficial permitir posição normal.

Referências identificam instâncias, papéis ou resultados definidos pelo motor ou por uma seleção anterior. Use somente uma das formas:

| `kind` | Campos adicionais | Significado |
|---|---|---|
| `SOURCE` | nenhum | Instância que contém a habilidade no contexto da ativação |
| `EVENT_OBJECT` | nenhum | Objeto que participou do evento contextual |
| `PLAYER` | `role` | Papel de jogador |
| `AREA` | `player`, `name` | Área pertencente ao jogador |
| `SELECTED_OBJECT` | `selection` | Objeto singular obtido por seleção anterior |
| `SELECTED_SET` | `selection` | Conjunto obtido por seleção anterior |
| `TEXT_DESIGNATED_OBJECT` | `designation` | Objeto singular designado pelo texto/regra |
| `ATTACKER` | nenhum | Atacante do contexto de batalha |
| `ATTACK_TARGET` | nenhum | Alvo do ataque no contexto de batalha |
| `BATTLE` | nenhum | Contexto de batalha fornecido pelo motor |
| `TOP_CARD` | `player`, `area` | Carta que está no topo da área indicada naquele momento |
| `CANDIDATE` | nenhum | Objeto candidato avaliado por um `filter` de seleção, escopo ou contagem |

Uma referência `PLAYER` usa `{ kind = "PLAYER", role = ROLE }`; `AREA` usa `{ kind = "AREA", player = PLAYER_REF, name = AREA_NAME }`. `AREA_NAME` aceita somente as áreas listadas acima. `SELECTED_OBJECT`/`SELECTED_SET` usam `selection` igual ao ID de passo `SELECT` anterior. Os demais `kind` não aceitam campos extras.

Papéis de jogador:

`YOU`, `OPPONENT`, `TURN_PLAYER`, `NON_TURN_PLAYER`, `CONTEXT_PLAYER`, `OWNER`, `CONTROLLER`, `ACTIVATOR`, `CHOOSER`, `EXECUTOR`, `AFFECTED`.

`TEXT_DESIGNATED_OBJECT.designation` deve reproduzir uma expressão exata presente em `abilities.source_text` e corresponder a uma única instância no contexto indicado. Não pode ser um rótulo inventado pela LLM. `CANDIDATE` só pode aparecer em `PROPERTY_COMPARE.subject` dentro de `SELECT.filter`, `continuous.scope.filter`, `OBJECT_COUNT.filter` ou filtro inline de pagamento; fora desses contextos é inválido.

`CONTROLLER` só pode ser usado quando a distinção estiver fundamentada no texto/regra. `YOU` não é substituição universal de `OWNER`, `CONTROLLER`, `TURN_PLAYER` ou `ACTIVATOR`.

`NONE` não é uma área. Uma carta temporariamente sem área, como no ciclo de Trigger, é tratada pelo procedimento do motor e não por uma área artificial `NONE`.

IDs de seleção são locais ao efeito. Uma referência a seleção só pode apontar para uma seleção anterior na mesma resolução. Não há referência a variável global, “última carta” implícita ou ID arbitrário de instância.

## 15. Quantidades, seleções e escolhas

### 15.1 Quantidade

Formas admitidas:

- `{ mode = "FIXED", value = { literal = 2 } }`;
- `{ mode = "UP_TO", maximum = { literal = 2 } }`;
- `{ mode = "DYNAMIC", value = { count_area = { area = { kind = "AREA", player = { kind = "PLAYER", role = "YOU" }, name = "TRASH" } } } }`.

`FIXED` exige quantidade positiva. `UP_TO` exige limite não negativo e permite zero quando o CR/texto permite. `DYNAMIC` usa somente valores dinâmicos da seção 13 e não pode gerar quantidade negativa. Não existe modo `PROCEDURE`: `DRAW` com `UP_TO` executa o procedimento oficial de parar antes de cada compra até o limite. Quantidade fixa, `up to` e contagem dinâmica não são intercambiáveis.

### 15.2 Seleção

Um passo `SELECT` declara:

- `id` único no efeito;
- `object`: `CARD` ou `DON`;
- `areas`: uma ou mais referências de área;
- `filter` quando o texto restringir elegibilidade;
- `quantity`;
- `chooser`;
- `information`;
- `information_step` quando a informação vier de LOOK/REVEAL anterior;
- `order_chooser` quando a regra/texto atribuir a um jogador a escolha da ordem.

`information` aceita `PUBLIC_INFORMATION`, `OWNER_PRIVATE_INFORMATION`, `LOOKED_BY_CHOOSER`, `REVEALED_TO_CHOOSER` ou `FACE_DOWN_UNVERIFIABLE`. `information_step` é obrigatório somente para `LOOKED_BY_CHOOSER` e `REVEALED_TO_CHOOSER`, e proibido nos demais casos. Para `LOOKED_BY_CHOOSER`, deve apontar para um `LOOK` anterior cujo `player` seja o próprio `chooser`; os candidatos ficam limitados às cartas daquele resultado que ainda permaneçam na área examinada. Para `REVEALED_TO_CHOOSER`, deve apontar para um `REVEAL` anterior que tenha incluído o `chooser` em `recipients` (ou omitido `recipients`, tornando a informação pública); os candidatos ficam limitados às cartas reveladas que ainda permaneçam nas áreas indicadas. `OWNER_PRIVATE_INFORMATION` exige que o `chooser` seja o owner da área secreta consultada. `FACE_DOWN_UNVERIFIABLE` não dispensa as restrições do CR para selecionar em área secreta.

A seleção não move, revela, K.O., descarta nem altera o objeto. Operações posteriores referenciam o ID da seleção. Uma seleção de cardinalidade singular produz `SELECTED_OBJECT`; uma seleção plural ou `UP_TO` produz `SELECTED_SET`. IDs são locais ao efeito e não sobrevivem à resolução.

`order_chooser` é uma referência PLAYER e só aparece quando o CR/texto dá a um jogador a escolha. Quando múltiplas cartas são colocadas simultaneamente em uma área, preserve a escolha de ordem exigida pelo CR. Nunca reescreva movimento simultâneo como sequência arbitrária.

### 15.3 Escolha de modo

`CHOOSE_MODE` contém `id`, `chooser` e `options`, array ordenado. Cada opção contém `id` e `steps`. Cada opção representa somente um ramo realmente oferecido pelo texto. Escolher modo não é selecionar carta; escolhedor e executor são papéis separados.

## 16. Replacement Effects — estrutura obrigatória

Esta seção incorpora integralmente o conteúdo normativo relevante do antigo adendo Replacement Effects v1.1.1. **O adendo deixa de existir como documento normativo separado.** Em caso de divergência com versões antigas, esta seção prevalece para FCL 1.0.0.

Toda declaração `category = "REPLACEMENT"` contém `[abilities.effects.replacement]`. `target` e `application` são tabelas descendentes; `costs` e `procedure` são arrays ordenados.

```toml
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

O exemplo demonstra forma estrutural; só é `SUPPORTED` se texto vigente, regra e contexto da carta sustentarem a substituição.

### 16.1 Campos obrigatórios

- `target`: procedimento que ocorreria, referência normativa e sujeito/contexto;
- `point`: `BEFORE_PROCESSING` ou `DURING_PROCESSING`, conforme o ponto real de aplicação;
- `replaced_part`: parte exata substituída;
- `available_when`: condição adicional, se necessária;
- `duration`: duração da disponibilidade, somente se definida por texto/regra;
- `application`: decisão de aplicação;
- `costs`: array de pagamentos, vazio se não houver;
- `procedure`: sequência ordenada que substitui a parte identificada.

`target` contém `procedure`, `rule_ref` e `subject`. `rule_ref` deve apontar para um ID de `[source.references]` que identifique a regra/FAQ aplicável. `subject` é a referência do processamento que seria realizado; não pode ser omitida. O próprio local da declaração identifica carta, habilidade, efeito e owner de origem; o motor preserva essa proveniência para a atribuição normativa do CR 8-1-3-4-7.

Catálogo fechado de `target.procedure`:

- `ACTIVATION_COST_PAYMENT` — pagamento de Activation Cost (CR 8-3-1-7);
- `LEADER_DAMAGE_PROCESSING` — processamento de dano ao Leader (CR 4-6 e/ou 7-1-4, conforme a causa);
- `CHARACTER_KO_PROCESSING` — procedimento de K.O. de Character (CR 10-2-1);
- `EFFECT_REMOVAL_FROM_FIELD` — remoção de carta do Field por efeito, somente quando essa classe ampla estiver expressa no texto, por exemplo “would be removed from the field by your opponent's effect”;
- `ORDINARY_CARD_MOVEMENT` — movimento comum entre áreas que não substitui K.O., dano, compra, Trigger, jogar carta ou DON!! −X.

`replaced_part` aceita `ACTIVATION_COST_PAYMENT`, `LEADER_DAMAGE`, `LIFE_TO_HAND_DURING_DAMAGE`, `CHARACTER_KO`, `FIELD_REMOVAL_BY_EFFECT` ou `ORDINARY_CARD_MOVEMENT`, conforme a matriz:

| `target.procedure` | `replaced_part` permitido |
|---|---|
| `ACTIVATION_COST_PAYMENT` | `ACTIVATION_COST_PAYMENT` |
| `LEADER_DAMAGE_PROCESSING` | `LEADER_DAMAGE`, `LIFE_TO_HAND_DURING_DAMAGE` |
| `CHARACTER_KO_PROCESSING` | `CHARACTER_KO` |
| `EFFECT_REMOVAL_FROM_FIELD` | `FIELD_REMOVAL_BY_EFFECT` |
| `ORDINARY_CARD_MOVEMENT` | `ORDINARY_CARD_MOVEMENT` |

Esses valores não são intercambiáveis. `EFFECT_REMOVAL_FROM_FIELD` não significa qualquer mudança de área: só corresponde ao evento definido pelo texto de remoção do Field causada por efeito. Se o texto não permitir identificar a classe, bloquear.

`application` contém exatamente `chooser` (PLAYER_REF), o jogador que decide aplicar ou recusar a substituição segundo a prioridade do CR 8-1-3-4. Toda substituição disponível pode ser recusada; se recusada, a substituição não resolve e o processamento original continua sem ela. Essa escolha é inerente ao procedimento de Replacement Effect e não é representada por um campo `optional`. Não confundir com `OPTIONAL` dentro de resolução, custo opcional ou seleção `UP_TO`.

`costs` usa a estrutura de pagamentos da seção 17. `procedure` usa passos das seções 18 e 19. Os IDs dos passos são únicos dentro do Replacement Effect. O procedimento substituto precisa ser integralmente realizável e suportado; se não puder ser realizado, a substituição não pode ser aplicada.

### 16.2 Regras que pertencem ao motor

A carta declara a substituição; o motor aplica CR 8-1-3-4:

1. se disponível e recusada, a substituição não resolve;
2. substituições concorrentes seguem a prioridade normativa: a relacionada à carta que gerou o efeito, depois as do Turn Player na ordem escolhida, depois as do Non-Turn Player na ordem escolhida;
3. a mesma substituição não pode ser aplicada repetidamente ao mesmo processamento individual já substituído;
4. a substituição troca toda a parte indicada, mesmo durante outra resolução; depois dela, o restante do texto continua na ordem do CR;
5. procedimento substituto impossível não pode ser aplicado;
6. uma substituição pode habilitar outra, sujeita às regras de prioridade e não repetição;
7. a parte substituída é tratada como efeito ativado pelo owner do Replacement Effect.

A carta não declara `priority`, algoritmo de encadeamento ou permissão para reaplicar a mesma substituição. O motor controla esses procedimentos. A identidade do evento/processamento substituído é mantida pelo motor; um novo evento independente pode voltar a oferecer a mesma substituição.

### 16.3 Substituição de Activation Cost — CR 8-3-1-7

Quando `target.procedure = "ACTIVATION_COST_PAYMENT"`:

1. a declaração identifica expressamente o pagamento de Activation Cost substituído;
2. o procedimento substituto é processado no lugar do pagamento original;
3. se o Activation Cost não for pago como descrito no texto, o efeito depois de `:` não é processado;
4. custo original, procedimento substituto e resolução depois de `:` permanecem elementos distintos.

A substituição não autoriza mover o pagamento para a resolução nem presumir que o texto depois de `:` sempre resolve.

### 16.4 Custos coletivos

Não existe regra genérica de pagamento único para várias substituições. Evento simultâneo que afeta vários objetos, várias remoções individuais e um custo textual são situações distintas. A tradução preserva a quantidade/unidade de pagamentos determinada pela carta e pelas fontes oficiais. Quando isso não estiver determinado, usar `BLOCKED_NORMATIVE`; não codificar “pagar uma vez independentemente da quantidade”.

### 16.5 Limite do catálogo

Replacement não pode ser omitido, convertido em `PERMANENT`, representado como operação `REPLACE` em resolução ou convertido em movimento genérico sem preservar o processamento substituído. Casos fora do catálogo desta seção ficam `BLOCKED_CAPABILITY` ou `BLOCKED_NORMATIVE`, conforme a causa.

## 17. Custos e pagamentos

`activation_costs` e `replacement.costs` são arrays ordenados. Cada pagamento é uma tabela inline discriminada por `kind`; campos não definidos para o tipo são rejeitados.

| `kind` | Campos obrigatórios | Regra |
|---|---|---|
| `REST_DON` | `kind`, `payer`, `selection` | Selecionar e restar a quantidade indicada de DON!! active na Cost Area do pagador. |
| `DON_MINUS` | `kind`, `payer`, `selection` | Selecionar a quantidade indicada de DON!! nas Leader, Character e Cost Areas do pagador e retorná-los ao DON!! Deck, conforme CR 8-3-1-6; qualquer estado é elegível. |
| `REST_CARD` | `kind`, `payer`, `selection` | Selecionar e restar uma carta active elegível no contexto do pagador. |
| `TRASH_HAND_CARD` | `kind`, `payer`, `selection` | Selecionar uma carta da mão do pagador e enviá-la à Trash. |

Em pagamentos, `selection` é uma tabela inline própria, não um ID de passo `SELECT`. Campos obrigatórios: `object` (`CARD` ou `DON`), `areas` (array não vazio de AREA_REF), `quantity` (§15.1), `chooser` (PLAYER_REF) e `information` (vocabulário de §15.2). `filter` (Condition) é opcional e, quando presente, usa `CANDIDATE` como sujeito. Para `REST_DON`, `areas` deve apontar apenas à Cost Area de `payer`, `object = "DON"` e `filter` deve restringir `STATE = ACTIVE`. Para `DON_MINUS`, `areas` deve apontar às três áreas permitidas do pagador. Para `REST_CARD` e `TRASH_HAND_CARD`, `quantity` deve ser FIXED 1. `REST_CARD.filter` deve restringir `STATE = ACTIVE`. Para `TRASH_HAND_CARD`, `areas` deve apontar à Hand de `payer`. O `chooser` é o jogador que escolhe; se texto/regra não permitir determinar esse jogador, bloquear.

DON!! só pode ser anexado pela operação `GIVE_DON`; pagamentos não transferem DON!! dado a outra carta. `REST_DON` usa DON!! active. `GIVE_DON.source_state = "RESTED"` só é permitido quando texto/FAQ oficial autorizar selecionar DON!! rested da Cost Area; a EB05-057 confirma somente seu caso específico. Depois de anexado, DON!! não é active nem rested (CR 4-4-2).

Regras obrigatórias:

- custo para jogar uma carta não é `activation_costs`;
- pagamentos ocorrem antes da resolução e preservam ordem textual;
- custos adicionados por outros efeitos são processados na ordem normativa do efeito que os adicionou;
- se, antes da ativação, não for possível pagar todo o custo, o efeito não pode ser ativado;
- se, depois da ativação, o pagamento se tornar impossível durante o processamento, paga-se o máximo possível, mas o texto depois de `:` não resolve (CR 8-3-1-3-1);
- se um custo opcional é recusado, o efeito associado não ativa;
- caminho alternativo de custo não é escolha de modo; não presumir que custos alternativos/adicionais sejam combináveis;
- substituição de Activation Cost é declarada como Replacement Effect e segue §16.3.

Outros tipos de pagamento ficam `BLOCKED_CAPABILITY` até extensão versionada.

## 18. Resolução — `resolution` e `steps`

`resolution` é um array ordenado de passos. Todo passo tem `id` único no efeito (inclusive ramos aninhados) e `kind`. Campos por `kind` são fechados:

| `kind` | Campos obrigatórios além de `id`/`kind` | Campos opcionais |
|---|---|---|
| `SELECT` | `object`, `areas`, `quantity`, `chooser`, `information` | `filter`, `information_step`, `order_chooser` |
| `OPERATION` | `operation` e os campos obrigatórios de §19.1 | somente os opcionais de §19.1 |
| `IF` | `condition`, `steps` | — |
| `THEN` | `steps` | — |
| `IF_YOU_DO` | `depends_on`, `steps` | — |
| `OPTIONAL` | `chooser`, `steps` | — |
| `CHOOSE_MODE` | `chooser`, `options` | — |
| `SIMULTANEOUS` | `steps` | — |
| `REPEAT` | `quantity`, `steps` | — |

`SELECT` segue §15.2. `OPERATION` só usa referências válidas no momento de execução. Em `IF`, `steps` contém exatamente o trecho que depende da condição; passos fora desse bloco não herdam a condição implicitamente. `THEN` contém os passos da cláusula textual correspondente, sem criar por si só dependência de sucesso do passo anterior. `IF_YOU_DO.depends_on` aponta para passo anterior que executa a ação citada; a dependência é o resultado definido pelo texto/CR, não uma regra universal de sucesso total. Se esse resultado não for determinável, bloquear.

### 18.1 `SELECT`

Cria uma seleção conforme §15.2. O ID do passo nomeia o resultado e só pode ser referenciado depois dele, dentro da mesma resolução.

### 18.2 `OPERATION`

Contém `operation` e os parâmetros obrigatórios da matriz §19.1. Só usa referências válidas no momento de execução.

### 18.3 `IF`

A condição é avaliada quando o passo é alcançado. Ações impossíveis e consequências sobre passos posteriores seguem texto e CR; não existe rollback geral.

### 18.4 `THEN`

Contém `steps`. Marca a cláusula textual subsequente; não cria dependência universal de sucesso do passo imediatamente anterior.

### 18.5 `IF_YOU_DO`

Contém `depends_on` (ID de passo anterior) e `steps`. Não definir sucesso como “todos”, “pelo menos um” ou “ação atômica” sem fundamento específico.

### 18.6 `OPTIONAL`

Contém `chooser` e `steps`. Representa somente ação durante a resolução que o texto permite recusar. Não é custo opcional, seleção `UP_TO`, escolha de modo nem recusa de Replacement Effect.

### 18.7 `CHOOSE_MODE`

Contém `chooser` e `options`, array não vazio. Cada opção contém `id` único dentro do `CHOOSE_MODE` e `steps`, em ordem textual. Cada ramo contém somente instruções daquela opção.

### 18.8 `SIMULTANEOUS`

Contém `steps` agrupados como simultâneos. O agrupamento não pode ser reescrito como sequência. Escolhas de ordem e ordem de decisões entre jogadores continuam sendo determinadas pelo CR.

### 18.9 `REPEAT`

Contém `quantity` e `steps`. Somente repetição finita: quantidade fixa, dinâmica explicitamente suportada ou `UP_TO` quando autorizado. Não há loop livre, condição de parada arbitrária ou recursão. O motor trata eventos gerados, Trigger, `rule processing` e loops gerais.

## 19. Operações admitidas

`operation` aceita somente as famílias abaixo. O identificador é uma instrução semântica; a execução concreta é procedimento do motor.

| Identificador | Semântica |
|---|---|
| `DRAW` | Comprar do topo do Deck pelo procedimento oficial. `UP_TO` usa a escolha de parar antes de cada compra. |
| `TRASH_HAND` | Enviar carta selecionada da Hand à Trash; não é K.O. |
| `TRASH_DECK` | Enviar carta do Deck à Trash pelo procedimento autorizado de seleção/informação; não é compra. |
| `KO_CHARACTER` | Aplicar procedimento oficial de K.O. a Character elegível; gera timing/evento de K.O. |
| `MOVE_CARD` | Movimento comum entre áreas; não equivale a K.O., dano, compra, Trigger, jogar carta ou DON!! −X. |
| `REST` | Alterar active para rested; objeto já rested permanece assim. |
| `SET_ACTIVE` | Alterar rested para active; objeto já active permanece assim. |
| `GIVE_DON` | Anexar DON!! da Cost Area a Leader/Character permitido; `source_state` declara o estado antes de anexar. O DON!! anexado não fica em estado active nem rested (CR 4-4-2). |
| `ADD_DON` | Mover DON!! do DON!! Deck para Cost Area com estado de chegada definido. |
| `DON_MINUS` | Executar procedimento oficial DON!! −X, selecionando DON!! nas Leader, Character e Cost Areas permitidas. |
| `PLAY_CARD` | Invocar procedimento oficial de jogar carta por efeito; não é movimento simples para o Field. O procedimento de jogar por efeito não cobra o Cost impresso, salvo custo adicional expressamente aplicável. |
| `REVEAL` | Revelar sem mover o `subject`, a `selection` ou todas as cartas vistas por `look_step`; `recipients` identifica quem recebe a informação. O motor mantém o histórico conforme o procedimento oficial. |
| `LOOK` | Olhar as `quantity` cartas do topo da área ordenada indicada, sem mover, revelar aos demais ou alterar a ordem. Produz um conjunto temporário, identificado pelo próprio ID do passo e disponível somente na mesma resolução. |
| `SHUFFLE_DECK` | Embaralhar Deck autorizado pelo procedimento. |
| `MODIFY_POWER` | Modificar Power efetivo, não base Power; aceita valores negativos conforme CR. |
| `SET_BASE_POWER` | Definir base Power com precedência normativa; não ajustar Power efetivo. |
| `MODIFY_COST` | Modificar Cost efetivo; Cost final usado pelo jogo não fica abaixo de zero. |
| `SET_BASE_COST` | Definir base Cost com precedência normativa; não modificar Cost efetivo. |
| `MODIFY_CHARACTERISTIC` | Alterar/conceder somente `NAME`, `TYPE` ou `ATTRIBUTE` neste perfil. |
| `GRANT_KEYWORD_EFFECT` | Conceder Keyword Effect/Keyword reconhecido, com validade e duração. |
| `DEAL_LEADER_DAMAGE` | Invocar procedimento oficial de dano ao Leader por efeito, incluindo Life, Trigger, substituições e derrota. |
| `MOVE_LIFE_CARD` | Movimento comum envolvendo Life; não é dano e não oferece Trigger por si só. |
| `POSITION_CARD` | Posicionar carta no topo/fundo de pilha autorizada. |
| `RETURN_CARD` | Retornar carta à Hand, Deck ou Trash, preservando destino e owner/player. |
| `END_GAME` | Conceder vitória/derrota somente quando explicitamente autorizado pelo texto/regra. |

### 19.1 Parâmetros por operação

`subject` é uma referência singular da §14; `selection` é o ID de um `SELECT` anterior no mesmo efeito. Quando a matriz diz `subject` ou `selection`, exatamente um deve aparecer. Referências de área são AREA_REF, `player` é PLAYER_REF, quantidades seguem §15.1 e durações seguem §21. Nenhum campo adicional é permitido.

| Operação | Campos obrigatórios | Campos opcionais permitidos |
|---|---|---|
| `DRAW` | `player`, `quantity` | — |
| `TRASH_HAND` | `player`, `selection` | — |
| `TRASH_DECK` | `player`, `selection` | — |
| `KO_CHARACTER` | `subject` ou `selection` | — |
| `MOVE_CARD` | `subject` ou `selection`, `source_area`, `destination_area` | `face`, `state`, `position`, `order_chooser` |
| `REST` / `SET_ACTIVE` | `subject` ou `selection` | — |
| `GIVE_DON` | destinatário `subject` ou `selection`, `quantity`, `source_state` | — |
| `ADD_DON` | `player`, `quantity`, `arrival_state` | — |
| `DON_MINUS` | `player`, `quantity` | — |
| `PLAY_CARD` | carta `subject` ou `selection`, `source_area`, `player` | `state` |
| `REVEAL` | exatamente um: `subject`, `selection` ou `look_step` | `recipients` |
| `LOOK` | `area`, `player`, `quantity` | — |
| `SHUFFLE_DECK` | `player` | — |
| `MODIFY_POWER` | `subject` ou `selection`, `operator`, `value`, `duration` | — |
| `SET_BASE_POWER` | `subject` ou `selection`, `value`, `duration` | — |
| `MODIFY_COST` | `subject` ou `selection`, `operator`, `value`, `duration` | — |
| `SET_BASE_COST` | `subject` ou `selection`, `value`, `duration` | — |
| `MODIFY_CHARACTERISTIC` | `subject` ou `selection`, `property`, `operator`, `value`, `duration` | — |
| `GRANT_KEYWORD_EFFECT` | `subject` ou `selection`, `keyword_effect`, `duration` | — |
| `DEAL_LEADER_DAMAGE` | `player`, `quantity` | — |
| `MOVE_LIFE_CARD` | `subject` ou `selection`, `source_area`, `destination_area` | `position`, `face`, `order_chooser` |
| `POSITION_CARD` | `subject` ou `selection`, `destination_area`, `position` | `order_chooser` |
| `RETURN_CARD` | `subject` ou `selection`, `destination_area` | `position` |
| `END_GAME` | `player`, `outcome` (`WIN` ou `LOSE`) | — |

`source_area`, `destination_area` e `area` são AREA_REF, exceto `AREA_COUNT.area` na §12.1, que é AREA_NAME. `recipients` é array não vazio de PLAYER_REF; omitido significa informação pública. `LOOK` não muda a publicidade da carta: a informação permanece restrita ao jogador indicado pelo procedimento; não há `recipients` em `LOOK`. `LOOK` examina as primeiras `quantity` cartas de uma área ordenada, cria um conjunto temporário e não altera a área nem a ordem. `REVEAL.look_step` é o ID de um passo `LOOK` anterior e revela todas as cartas daquele conjunto; se `REVEAL` usar `subject` ou `selection`, revela somente o objeto/conjunto indicado. `look_step` só pode apontar para um `LOOK` anterior na mesma resolução. `face` aceita `FACE_UP` ou `FACE_DOWN`; `state`/`source_state`/`arrival_state` aceitam `ACTIVE` ou `RESTED`; `position` aceita `TOP`, `BOTTOM` ou `NORMAL` quando permitido pelo procedimento. `order_chooser` é PLAYER_REF.

`GIVE_DON.source_state = "ACTIVE"` é o caso normal; `source_state = "RESTED"` só é suportado quando texto/FAQ oficial autorizar essa seleção. EB05-057 confirma apenas o cenário perguntado. Ao anexar, o DON!! deixa de ter estado active/rested, conforme CR 4-4-2. `GIVE_DON` sempre usa DON!! da Cost Area, nunca DON!! dado a outra carta. `ADD_DON` move DON!! do DON!! Deck para a Cost Area.

`MODIFY_POWER.operator` e `MODIFY_COST.operator` aceitam `ADD` ou `SUBTRACT`. `MODIFY_CHARACTERISTIC.property` aceita somente `NAME`, `TYPE` ou `ATTRIBUTE`. `operator` aceita `SET` para substituir o valor, `ADD` para acrescentar TYPE/ATTRIBUTE ou `REMOVE` para remover TYPE/ATTRIBUTE. `NAME` aceita somente `SET`; valores de `NAME`/`TYPE` são strings e `ATTRIBUTE` é um enum da §6.1. Mudança de cor/categoria fica bloqueada.

`END_GAME` não representa concession: concession não pode ser forçada por carta nem substituída (CR 1-2-4). Representa somente efeito explícito de vitória/derrota (CR 1-2-5).

`GRANT_ABILITY`, `NEGATE_EFFECT`, `PROHIBIT_ACTION` e `MODIFY_PROCEDURE` não são operações desta versão. Se o texto exigir uma delas, usar `BLOCKED_CAPABILITY`.

### 19.2 Semânticas que não podem ser colapsadas

- K.O. gera o procedimento/evento de K.O.; Trash comum não gera `[On K.O.]`.
- `MOVE_LIFE_CARD` não causa dano e não oferece Trigger. Trigger só ocorre no procedimento oficial de dano.
- `PLAY_CARD` processa jogabilidade, custo/isenção, destino e timing de entrada; não é `MOVE_CARD`.
- DON!! dado a uma carta é diferente de DON!! na Cost Area e de DON!! no DON!! Deck.
- `MODIFY_POWER` não altera base Power. A redação “set Power to 0” tem semântica própria no CR 4-12 e não é `SET_BASE_POWER` nem `MODIFY_POWER`; como FCL 1.0.0 não possui instrução específica para esse procedimento, fica `BLOCKED_CAPABILITY`.
- Power pode tornar-se negativo sem mover a carta. Cost efetivo fora de cálculo é tratado como zero se negativo; o valor negativo pode participar de cálculo conforme CR.
- Negação genérica de efeito, concessão de habilidade, proibição de ação e alteração de procedimento ficam bloqueadas nesta versão; não são aproximadas por remoção de keyword ou por `MOVE_CARD`.
- `BLOCKER` não é redirecionamento genérico; o motor executa o procedimento da keyword.

## 20. Efeitos contínuos — `continuous`

`continuous` representa consequência contínua, não sequência de ações. Exatamente um de `subject` ou `scope` deve identificar onde o efeito se aplica. Campos restantes dependem de `kind`:

| `kind` | Campos adicionais obrigatórios | Campos opcionais |
|---|---|---|
| `MODIFY_POWER` / `MODIFY_COST` | `operator`, `value`, `duration` em Auto/Activate | — |
| `SET_BASE_POWER` / `SET_BASE_COST` | `value`, `duration` em Auto/Activate | — |
| `MODIFY_CHARACTERISTIC` | `property`, `operator`, `value`, `duration` em Auto/Activate | — |
| `GRANT_KEYWORD_EFFECT` / `REMOVE_KEYWORD_EFFECT` | `keyword_effect`, `duration` em Auto/Activate | — |

`scope` aceita `{ object = "CARD"|"DON", areas = [AREA_REF, ...], player = PLAYER_REF, filter = CONDITION }`; `player` e `filter` são opcionais. Sem `player`, o escopo inclui todos os jogadores. Em filtros, `subject = { kind = "CANDIDATE" }` representa cada objeto examinado. `subject` é uma referência singular da seção 14. `property` aceita `NAME`, `TYPE` ou `ATTRIBUTE`. `keyword_effect` aceita somente um Keyword Effect listado em §10.1.

Em `AUTO`/`ACTIVATE`, `duration` é obrigatório para todo `continuous`; o efeito `PERMANENT` usa `valid_when` opcional (ausente = sempre válido, sujeito às regras de área) e não declara `duration` dentro de `continuous`. `operator` aceita `ADD` ou `SUBTRACT`. `MODIFY_POWER` pode produzir Power negativo; `MODIFY_COST` segue o tratamento do CR para Cost negativo. `SET_BASE_POWER` e `SET_BASE_COST` alteram valores-base e seguem precedência normativa. `valid_when`, quando presente em `PERMANENT`, não se repete dentro de `continuous`.

`GRANT_ABILITY`, `NEGATE_EFFECT`, `PROHIBIT_ACTION` e `MODIFY_PROCEDURE` estão fora do perfil executável 1.0.0. Se o texto exigir uma dessas capacidades, usar `BLOCKED_CAPABILITY`. Replacement Effects continuam sendo exclusivamente `category = "REPLACEMENT"` (§16).

## 21. Durações admitidas

`duration` usa objeto inline com `kind` e campos condicionais:

- `THIS_TURN`;
- `UNTIL_END_OF_TURN`;
- `UNTIL_END_OF_END_PHASE`;
- `THIS_BATTLE`;
- `UNTIL_START_OF_NEXT_TURN` com `player`;
- `WHILE_VALID` com `condition`;
- `PERMANENT`, somente quando texto/regra determinar duração sem expiração temporal ordinária.

Não normalizar `THIS_TURN` e `UNTIL_END_OF_TURN`. O motor processa expirações nos marcos oficiais. Não inventar contadores ou eventos de expiração (`next attack`, `next draw`, etc.) não presentes no catálogo.

Uma consequência contínua criada por Auto/Activate não desaparece automaticamente porque a carta-fonte saiu do Field; sua validade e duração são avaliadas conforme texto e CR. Permanent Effects dependem da validade da habilidade, com exceções de área somente quando o CR/texto as permitir. O motor aplica a ordem de estabilização de permanentes: Turn Player, depois Non-Turn Player, repetindo quando o estado mudar conforme CR 8-1-3-3-5.

## 22. Limites de uso

`usage_limit` aceita somente:

```toml
usage_limit = { kind = "ONCE_PER_TURN" }
```

A aplicação é por carta/instância, conforme CR 10-2-13. Sair da área e retornar pode produzir uma nova instância para a regra. Se a ativação chegou ao pagamento e o custo se tornou impossível durante seu processamento, `[Once Per Turn]` ainda é consumido e não pode ser ativado novamente naquele turno (CR 10-2-13-5). O motor controla reset, identidade da instância e momento de consumo.

Não admitir `ONCE_PER_BATTLE`, `ONCE_PER_GAME`, `ONCE_PER_PHASE` ou `ONCE_PER_ATTACK` como variantes genéricas.

## 23. Regras de validação

Todas as camadas são obrigatórias para compilar.

### 23.1 Sintática

O arquivo deve ser TOML 1.0.0 válido em UTF-8. Erro de parser encerra a validação.

### 23.2 Estrutural

- somente tabelas/chaves autorizadas neste documento;
- tipos de valor corretos; chaves obrigatórias presentes e proibidas ausentes;
- enumerações reconhecidas;
- IDs únicos no escopo definido;
- campos específicos de categoria não misturados;
- arrays usados onde a ordem importa;
- parâmetros compatíveis com operação, teste, pagamento e passo;
- `editorial.extra` é a única área de metadados livres e não executáveis;
- nenhum `null`, chave duplicada ou tipo TOML fora do perfil.

### 23.3 Referencial

- toda referência usa forma autorizada da seção 14;
- `source_refs`, `replacement.target.rule_ref` e `translation.blockers.references` apontam para IDs existentes em `[source.references]`;
- referência a seleção aponta para `SELECT` anterior e compatível no mesmo efeito;
- `depends_on` aponta para passo anterior compatível;
- área identifica jogador e nome válidos;
- propriedade/estado é aplicável ao objeto;
- não existe acesso arbitrário por ID de instância.

### 23.4 Semântico

- categoria, timing e janela corretos;
- condições no contexto/momento corretos;
- custo separado da resolução;
- seleção preserva objeto, área, filtro, cardinalidade, escolhedor e informação;
- escolha de modo separada de seleção;
- executor separado de escolhedor;
- `if`, `then` e `if you do` distintos;
- operações correspondem a procedimentos oficiais;
- ordem e simultaneidade preservadas;
- efeitos contínuos têm sujeito/escopo, validade e duração;
- Replacement Effect contém todos os elementos exigidos em §16;
- K.O., Trash, dano, Life, Trigger, jogar carta e DON!! −X continuam distintos;
- qualquer semântica não demonstrada resulta em bloqueio.

### 23.5 Normativa

Interpretação que dependa de CR, FAQ ou errata deve ser rastreável por `source_refs` ou `translation.blockers.references`. FAQ específica não pode ser generalizada para além do caso respondido. Texto `raw_text` nunca prevalece sobre o texto oficial vigente `current_text`.

## 24. Política de bloqueio

Estados possíveis são definidos na seção 8. A LLM tradutora deve bloquear quando não puder provar, a partir do texto vigente e das fontes aplicáveis:

- jogador, owner, controller, escolhedor, executor ou afetado;
- timing, janela, condição ou momento de avaliação;
- objeto designado, seleção, cardinalidade ou informação disponível;
- seleção aleatória, quando o perfil não fornecer construção específica;
- ordem, simultaneidade ou dependência;
- custo, pagamento ou opcionalidade;
- processamento substituído, ponto, parte substituída ou procedimento substituto;
- validade, escopo ou duração;
- consequência de `if you do`;
- identidade da carta/instância ou versão de texto aplicável.

`BLOCKED_NORMATIVE` significa que falta fundamento ou esclarecimento oficial. `BLOCKED_CAPABILITY` significa que a semântica é conhecida, mas o perfil 1.0.0 não possui construção para representá-la. `INVALID_SOURCE` significa que texto, errata, idioma/região ou dados de origem são insuficientes/inconsistentes.

Nenhum bloqueio pode ser convertido em operação vazia, valor padrão, ramo aproximado ou omissão de texto. A LLM nunca inventa enumeração, opcode, propriedade, jogador ou exceção.

## 25. Versionamento e compatibilidade

- `language_version` identifica estrutura textual, catálogo e semântica de FCL. A versão desta especificação e da linguagem é `1.0.0`; não há versão semântica independente.
- Campos desconhecidos são rejeitados, exceto chaves escalares em `[editorial.extra]`.
- Mudança incompatível de estrutura, enumeração, tipo ou significado exige nova versão principal.
- Extensão compatível sem alterar significado existente exige nova versão menor.
- Correção textual sem mudança de semântica pode receber versão patch do documento, mas não altera `language_version`.
- Mudança que altere significado de construção exige nova versão da linguagem.
- Compilador rejeita versão que não declara suportar; nunca reinterpreta silenciosamente.

## 26. Exemplos de referência

Os exemplos seguintes demonstram a estrutura TOML. São exemplos sintáticos, não substituem auditoria da carta e das regras.

### 26.1 Permanent Effect simples

```toml
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

### 26.2 Auto Effect com resolução ordenada (fragmento)

```toml
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

### 26.3 Condição de resolução após um passo anterior

```toml
resolution = [
  { id = "S1", kind = "SELECT", object = "CARD", areas = [{ kind = "AREA", player = { kind = "PLAYER", role = "YOU" }, name = "HAND" }], quantity = { mode = "FIXED", value = { literal = 1 } }, chooser = { kind = "PLAYER", role = "YOU" }, information = "OWNER_PRIVATE_INFORMATION" },
  { id = "S2", kind = "OPERATION", operation = "TRASH_HAND", player = { kind = "PLAYER", role = "YOU" }, selection = "S1" },
  { id = "S3", kind = "IF", condition = { test = "AREA_COUNT", area = "TRASH", player = { kind = "PLAYER", role = "YOU" }, operator = "GTE", value = { literal = 5 } }, steps = [
    { id = "S3A", kind = "OPERATION", operation = "SET_BASE_POWER", subject = { kind = "SOURCE" }, value = { literal = 5000 }, duration = { kind = "UNTIL_END_OF_TURN" } }
  ] }
]
```

A condição é consultada no ponto em que o nó `IF` é processado. Esse exemplo não concede licença para congelar o estado no início da resolução.

### 26.4 Replacement Effect de Activation Cost

```toml
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

O exemplo só pode ser usado quando o texto vigente, a regra e o contexto da carta efetivamente suportarem essa substituição. A regra CR 8-3-1-7 determina a consequência para o efeito cujo Activation Cost foi substituído. A seleção e a operação constituem o procedimento substituto; não são declaradas como um Activation Cost adicional.

### 26.5 Auto Effect de Trigger

Exemplo estrutural para uma carta não-Event cujo texto de Trigger executa um efeito próprio:

```toml
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

O motor oferece a ativação durante o processamento de dano de Life. Este exemplo é um fragmento; os metadados de carta/fonte e o estado de tradução devem constar no arquivo completo.

### 26.6 Event com Trigger que ativa a própria carta

Para texto `[Trigger] Activate this card.`, usar `keyword_effects = ["TRIGGER"]` e `event_activation = { window = "TRIGGER" }`. A habilidade normal da Event é declarada separadamente como `ACTIVATE` com `activation_window = "EVENT_CARD_RESOLUTION"`. Não criar uma operação `MOVE_CARD` para simular a ativação.

## 27. Conformidade

Um programa é conforme com FCL 1.0.0 somente quando:

1. é TOML 1.0.0 válido;
2. usa `format = "FL_CARD_LANGUAGE"` e `language_version = "1.0.0"`;
3. respeita integralmente estrutura, vocabulários e invariantes deste documento;
4. passa validações sintática, estrutural, referencial, semântica e normativa;
5. não contém habilidade de gameplay bloqueada;
6. não contém campos desconhecidos fora de `[editorial.extra]`.

`SUPPORTED` é conclusão de tradução, não prova de que a VM implementa todas as regras do jogo. Conformidade do programa, cobertura do compilador e cobertura do motor são três verificações distintas.

## 28. Congelamento da versão 1.0.0

FCL 1.0.0 congela:

- TOML 1.0.0 como formato textual;
- um arquivo por identidade lógica de carta;
- separação entre identidade, fonte normativa, tradução, editorial e habilidades;
- as quatro categorias normativas, incluindo Replacement Effect de primeira classe;
- condições, referências, valores, seleções, escolhas, custos, resolução, operações, contínuos, durações e limites de uso aqui enumerados;
- o tratamento do Trigger e de Event card activation;
- a estrutura completa de Replacement Effects incorporada na seção 16;
- validação estrita e bloqueio de construções não suportadas.

Esta versão está **congelada para iniciar testes de tradução de cartas reais**. Ela não declara que todas as cartas do jogo sejam traduzíveis nem que a Rules Engine esteja pronta. Casos fora do perfil permanecem bloqueados; não alteram silenciosamente a versão congelada.

Para mudar semântica ou estrutura, abrir proposta de extensão, registrar carta/texto/regra que exige a capacidade, demonstrar por que as construções existentes são insuficientes, definir a interação com o perfil atual e publicar nova versão. Não editar o significado de FCL 1.0.0 durante a tradução de cartas.

## 29. Fronteira do motor e semântica normativa complementar

Esta seção consolida as responsabilidades que pertencem ao motor e não são instruções FCL.

### 29.1 Identidade e papéis

A carta impressa (card number/texto vigente) não é a instância em jogo. Uma instância tem área, owner, controller/contexto, estado, relações com DON!! e histórico regulamentar. Mudança de área pode criar nova instância para efeitos específicos; o card number não substitui identidade de instância.

Referências aceitas: `SOURCE`, `EVENT_OBJECT`, `PLAYER`, `AREA`, `SELECTED_OBJECT`, `SELECTED_SET`, `TEXT_DESIGNATED_OBJECT`, `ATTACKER`, `ATTACK_TARGET`, `BATTLE` e `TOP_CARD`. Papéis: `YOU`, `OPPONENT`, `TURN_PLAYER`, `NON_TURN_PLAYER`, `CONTEXT_PLAYER`, `OWNER`, `CONTROLLER`, `ACTIVATOR`, `CHOOSER`, `EXECUTOR` e `AFFECTED`. `YOU` nunca é substituto universal de owner, controller, Turn Player ou ativador. `OWNER`/`CONTROLLER` são relativos ao objeto contextual. `TEXT_DESIGNATED_OBJECT.designation` reproduz expressão literal de `source_text` e precisa identificar uma única instância. `NONE` não é área.

Áreas: `DECK`, `DON_DECK`, `HAND`, `TRASH`, `LEADER_AREA`, `CHARACTER_AREA`, `STAGE_AREA`, `COST_AREA`, `LIFE_AREA`. Leader, Character, Stage e Cost Areas formam o Field. Estados: `ACTIVE`, `RESTED`; faces: `FACE_UP`, `FACE_DOWN`; posições: `TOP`, `BOTTOM`, `NORMAL` quando autorizada.

DON!! no DON!! Deck, na Cost Area e anexado a uma carta são posições distintas. DON!! anexado não é active nem rested (CR 4-4-2), não está disponível na Cost Area e não pode ser transferido diretamente entre cartas. Quando a carta-hospedeira sai do Field, o procedimento de retorno de DON!! pertence ao motor.

### 29.2 Eventos, ativação e ordem

Evento é um acontecimento contextual do jogo; timing é o ponto regulamentar em que uma habilidade pode ativar. O motor gera eventos e timings. FCL declara apenas timings do catálogo da seção 11. Condições de turno como `YOUR_TURN`/`OPPONENT_TURN` são condições, não eventos.

Auto Effect distingue timing satisfeito, efeito pendente, ativação efetiva e resolução. Se a origem que satisfez o timing mudar de área antes da ativação, o Auto Effect não ativa (CR 8-1-3-1-3). Efeito já ativado/resolvido que criou consequência contínua segue sua validade/duração; sair da área não o apaga automaticamente.

`[On K.O.]` só é gerado pelo procedimento de K.O. definido pelo CR. Enviar uma carta à Trash por outro meio não gera esse timing automaticamente.

`[Trigger]` só é oferecido durante o processamento oficial de dano de Life, no lugar de adicionar a carta à mão. Se o jogador não ativar, o processamento normal continua; se ativar, a resolução do Trigger ocorre no ponto regulamentar e a carta segue o destino previsto pelo procedimento. Se efeito ou Replacement Effect impedir a adição à mão, o Trigger não pode ser ativado (CR 4-6-3-1). Mover uma carta para/da Life por efeito comum não oferece Trigger. Em Double Attack, o dano/Trigger é processado ponto a ponto conforme o CR.

Ativar Event card e ativar efeito de uma carta são atos diferentes. `[Main]`/`[Counter]` de Event são janelas de ativação da carta; texto que resolve depois da ativação é representado separadamente. Blocker, Counter, batalha, fases e prioridades são procedimentos do motor.

### 29.3 Condições, seleção e informação

Condição de ativação, condição de validade contínua, condição de resolução e filtro de seleção são contextos distintos. Condições de ativação são verificadas antes da ativação; condições de resolução são avaliadas no momento exigido pelo texto/CR, podendo observar alterações feitas por passos anteriores. Não presumir snapshot do estado no início da resolução.

Seleção identifica objetos elegíveis; não os move, revela ou afeta. Quantidade fixa não é `UP_TO`; `UP_TO` permite zero quando autorizado. O procedimento `DRAW UP TO X` permite parar antes de cada compra. Um objeto designado pelo texto ausente não é o mesmo que uma seleção vazia. Áreas secretas obedecem ao procedimento de informação do CR; olhar, revelar e escolher são decisões diferentes.

Quando múltiplos jogadores precisam escolher simultaneamente, o Turn Player escolhe primeiro. Quando múltiplas cartas são colocadas simultaneamente numa área, o owner decide a ordem nos termos do CR, salvo regra/texto diferente.

### 29.4 Resolução e impossibilidade

A resolução segue a ordem textual. Ação impossível não acontece; não existe rollback geral das ações anteriores. A consequência sobre passos posteriores depende da estrutura e do CR. `IF`, `THEN` e `IF_YOU_DO` permanecem distintos. Instruções independentes continuam quando o texto/CR as torna independentes.

Custo de jogar carta é distinto de Activation Cost. Se custo é impossível antes da ativação, o efeito não ativa. Se pagamento se torna impossível durante processamento após a ativação, paga-se o máximo possível, a resolução não ocorre e o limite `[Once Per Turn]` é consumido quando aplicável. Custo voluntário recusado impede ativação. Escolha de modo, seleção `UP_TO` e ação opcional durante resolução não são custos.

### 29.5 Efeitos contínuos, negação e proibição

Permanent Effect permanece válido enquanto a condição e as regras permitirem. Alguns efeitos “according to/under the rules” podem permanecer válidos em área secreta; só se aplica quando texto/CR identificar a exceção. Efeitos contínuos são aplicados/estabilizados pelo motor na ordem Turn Player, Non-Turn Player e repetição se o estado mudar (CR 8-1-3-3-5).

Proibição de ação, negação/invalidação de efeito e impossibilidade de uma operação são fenômenos diferentes. FCL 1.0.0 não possui instruções genéricas para os dois primeiros; quando o texto exigir esses comportamentos, bloquear. A VM não deve simulá-los com remoção de keyword ou operação de movimento.

### 29.6 Keywords e procedimentos próprios

Keyword Effects do perfil: Rush, Double Attack, Banish, Blocker, Trigger, Rush: Character e Unblockable. Cada termo chama procedimento oficial. Outros Keywords são codificados pela função: `ON_PLAY`, `ON_KO`, `WHEN_ATTACKING`, `ON_BLOCK`, `ON_OPPONENT_ATTACK`, `MAIN`, `COUNTER`, `DON!! xX`, `DON!! −X`, `YOUR_TURN`, `OPPONENT_TURN`, `ONCE_PER_TURN` e `TRASH`.

A linguagem não fornece operação genérica para transferir controle, copiar qualquer efeito, redirecionar ataque, cancelar ataque/batalha, modificar arbitrariamente regras do motor ou repetir resolução sem limite. Blocker não é redirecionamento genérico; encerramento de batalha é consequência do procedimento de batalha.

### 29.7 Casos FAQ incorporados e limite

As FAQs abaixo foram incorporadas somente nos casos perguntados; não devem ser generalizadas além deles:

- `EB05-006`: olhar o topo da Life antes de escolher um dos modos é permitido no caso perguntado.
- `EB05-008`: efeito Counter com `up to 1` pode ser ativado sem Character no Field.
- `EB05-011`: a condição específica de On Play não é satisfeita quando a carta do topo da Life já está face-down no caso perguntado.
- `EB05-028`: o oponente do ativador escolhe as quatro cartas da própria mão para Trash.
- `EB05-033`: o requisito de tipo do Leader e diferença de DON!! do Field é condição obrigatória.
- `EB05-034`: a condição posterior é avaliada depois do DON!! −2 e pode deixar de ser satisfeita.
- `EB05-045`: On Play pode K.O. a própria fonte e, no caso perguntado, pode escolher não K.O. nenhuma carta.
- `EB05-046`: a contagem de Trash pode ser avaliada depois do descarte realizado no passo anterior.
- `EB05-052`: a substituição de dano de Gloriosa abrange os três cenários respondidos: dano de batalha com zero Life, ambos os danos de Double Attack e dano por efeito.
- `EB05-057`: permite anexar DON!! rested, retirado da Cost Area, no cenário específico de Leader/Character com múltiplos Attributes incluindo Special ou Wisdom. `GIVE_DON.source_state` registra o estado antes do anexo; o DON!! anexado passa a não ser active nem rested (CR 4-4-2). A FAQ não cria permissão geral.
- `EB05-059`: a imunidade à K.O. em batalha não se aplica no cenário específico respondido.
- `EB05-061`: a substituição de Nami pode ser usada quando a própria fonte seria removida, mas não no cenário sem Life para realizar a consequência.
- `EB05-010`: negação de Blocker impresso, concedido continuamente ou por Trigger tem o tratamento específico descrito pela FAQ.
- `EB05-021`: On Play de Character jogado pelo efeito não encadeia automaticamente outro On Play no cenário respondido.

Questões gerais ainda não cobertas, incluindo FAQs históricos não verificados, reconciliação integral de erratas, custo de substituições coletivas, interações complexas de Replacement Effects, momentos de avaliação não esclarecidos, efeitos em áreas secretas, loops e conflitos regionais/idioma, continuam sendo limites do perfil. Caso concreto dependente de uma resposta não verificada fica `BLOCKED_NORMATIVE`. Custo coletivo de substituições permanece deliberadamente sem regra genérica.

---

## Referências oficiais

- TOML 1.0.0: https://toml.io/en/v1.0.0
- *ONE PIECE CARD GAME Comprehensive Rules* v1.2.1, atualizado em 28/08/2026: https://en.onepiece-cardgame.com/pdf/rule_comprehensive.pdf?20260828=
- Índice oficial de Rules e erratas: https://en.onepiece-cardgame.com/rules/
- Índice oficial de FAQs: https://en.onepiece-cardgame.com/rules/faq/
- FAQ oficial EB-05, atualizada em 06/11/2026: https://en.onepiece-cardgame.com/pdf/qa_eb05.pdf?20261106=

## Limite de certificação

Este documento fecha um perfil conservador de construções suportadas. Não certifica que todas as cartas, todas as FAQs históricas, todas as erratas ou todas as interações possíveis foram auditadas. O critério é: comportamento definido pelo perfil pode ser traduzido; comportamento que dependa de regra não verificada ou construção ausente deve ser bloqueado.

**Regra final:** a LLM interpreta e declara; o validador verifica a estrutura e as restrições; o compilador transforma somente programas `SUPPORTED`; a VM executa instruções válidas; a Rules Engine aplica os procedimentos oficiais. Nenhuma dessas etapas pode inventar semântica.
