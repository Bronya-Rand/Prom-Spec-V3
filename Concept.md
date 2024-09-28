# The Prom V3 Spec

## Keywords

- *MUST*: The folowing field is an absolute requirement of the specification.
- *MUST NOT*: The folowing field cannot do the following according to the specification.
- *SHOULD*: The folowing field can do the following, but isn't necessarily required to function.
- *SHOULD NOT*: The folowing field shouldn't do the following functions.
- Character Editor: A editor used to create Prom V3 objects.
- Application: The application that uses the Prom V3 specification.
- (Same as X): The logic of this field is the same as the following previous specifications.

## JSON Objects in Prom V3 (Character Card V3.1)
Prom V3 takes what already exists in V2 and RisuAI's V3 and adapts it to be easier to read for application developers to implement in their own codebases without the unnecessary bloat of Risu's `assets` folder (that I have already explained [here](./README.md#assets)) and the incorporation of beneficial features such as:

1. Adding the ability for multiple scenarios and/or greetings in one object.
2. Combining character info outside the character data card if it's not needed.
3. Add Point 1 to Group Chats.
4. Make example messages easier to parse.

Due to these optimizations, primarily the new `CharacterScenario` object, V1 and V2 character card readers will not be able to read Prom V3 cards. V3 readers however can read V1/V2 by adapting the fields in V1/V2 to V3 as described in *Deprecated Fields*. V1/V2 readers may be able to adapt V3 entries to V1/V2 types via *Deprecated Fields* but I cannot guarantee everything will be in it (especially with multiple scenarios and/or group scenarios).

In addition, due to the removal of the `assets` field, any features from RisuAI or other frontends that use that field will not be imported to Prom V3. This *can* be added back by a field addition, but IMO, this is better reserved in `extensions` if it's actually needed (see [CharX](./README.md#charx-charx) for my thoughts).

For an example of a bot written in Prom V3, see the example bot provided [here](./Example%20Bot.json).

## Embedding Methods
Prom V3 may be embedded using traditional V2 embedding methods via a PNG/APNG or JSON file. I cannot guarantee if Prom V3 can work under a CharX (.charx) file type.

## CharacterCard Object
```ts
interface CharacterCardV3_1 {
    type: "chara_card"
    spec_version: '3.1'
    data: {
        name: string
        description: string
        personality: string

        // new in Prom V3
        char_info?: CharacterInfo
        created_at: number
        updated_at: number

        // (major change) combines first message, alt greetings and scenario as a object array
        scenarios: Array<CharacterScenario> 
        group_scenarios: Array<CharacterScenario>
        example_messages: Array<CharacterExampleMessage> // (major change)

        // additional content for backwards compatibility
        system_prompt: string
        post_history_instructions: string
        extensions: Record<string, any>
        character_book?: CharacterBookV2
    }
}
```

### Field Descriptions

#### `type`

This **MUST** be set as `chara_card`

#### `spec_version`

(Same as V1/V2/Risu V3) For Prom V3.1, this **MUST** be set as `"3.1"`.

#### `name`

(Same as V1/V2/Risu V3) Stores the name of the character. This **MUST** be a string.

#### `description`

(Same as V1/V2/Risu V3) Stores the description of the character (**NOT** to be confused with `creator_notes`). For some characters, this may also contain all the information of the bot itself (see [Silver Wolf](https://bronya-rand.github.io/reimagined-couscous/chars/[HSR]%20Silver%20Wolf/Silver%20Wolf.json)). This **MUST** be a string.

#### `personality`

(Same as V1/V2/Risu V3) Stores the personality of the character. This **MUST** be a string.

#### `char_info` 

Stores information about the character such as the version of the card, creator notes and information and tags. This **MUST** be a `CharacterInfo` object. 

1. This field **MUST** return an empty object if no creator data is present.
2. **ALL APPLICATIONS, CHARACTER EDITORS, ETC. MUST** follow the [CharacterInfo](#characterinfo-object) specification.

#### `scenarios`

Stores several different scenarios and greeting messages available for the character. This **MUST** be a array of `CharacterScenario` objects.
> This combines `alternate_greetings`, `scenario` and `first_mes` into one object.

1. This field **MUST** return an empty object if no scenarios is present.
2. **ALL APPLICATIONS, CHARACTER EDITORS, ETC. MUST** follow the [CharacterScenario](#characterscenario-object) specification.

#### `group_scenarios`

Stores several different scenarios and greeting messages available for the character that are reserved for group chats. This **MUST** be a array of `CharacterScenario` objects.
> This combines `alternate_greetings`, `scenario` and `first_mes` into one object for group chats.

1. This field **MUST** return an empty object if no group scenarios is present.
2. **ALL APPLICATIONS, CHARACTER EDITORS, ETC. MUST** follow the [CharacterScenario](#characterscenario-object) specification.

#### `created_at`

Stores the timestamp (in Unix time) of when the character was first created. This **MUST** be a number.
1. This field **MUST** not be modified after character creation.
2. This field **MUST** return a Unix timestamp (in seconds) with the timezone set as UTC.

#### `updated_at`

Stores the timestamp (in Unix time) of when the character was last updated. This **MUST** be a number.
1. This field **MUST** be modified if a character change has occured.
2. This field **MUST** return a Unix timestamp (in seconds) with the timezone set as UTC.

#### `example_messages`

Stores example messages to teach the AI how to interact with the user as the character. This **MUST** be a array of `CharacterExampleMessage` objects.
1. This field **MUST** return an empty object if no example messages are present.
2. **ALL APPLICATIONS, CHARACTER EDITORS, ETC. MUST** follow the [CharacterExampleMessage](#characterexamplemessage-object) specification.

#### `system_prompt`

(Same as V2) Stores the system prompt used by the character when sending messages to the AI model. Replaces the frontend system prompt (if in use). This **MUST** be a string.

#### `post_history_instructions`

(Same as V2) Stores the "jailbreak" used by the character when sending messages to the AI model. Replaces the frontend jailbreak (if in use). This **MUST** be a string.

#### `extensions`

(Same as V2) Stores additional information from content creators, sites, character editors, apps, etc.

1. This field **MUST** return an empty object if no additional content is needed to be written.
2. This field **MAY** contain any arbitrary JSON key-value pair.
3. Character Editors **MUST NOT** destroy key-value pairs that already exist in the field.
4. Applications **MAY** store any object data in this field.
5. Applications **SHOULD** namespace keys that they use to avoid name conflicts i.e. `"agnai/voice": /* ... */` or `"agnai_voice": /* ... */` or `"agnai": { "voice": /* ... */ }"`.

#### `character_book`

Stores a character lorebook that can be used/imported onto Applications. This **MUST** be a `CharacterBookV2` object. 

1. This field **MUST** return an empty object if no lorebook is linked to the character.
2. Applications **SHOULD** ask users to import lorebooks if present.
3. **ALL APPLICATIONS, CHARACTER EDITORS, ETC. MUST** follow the [CharacterBookV2](#characterbookv2-object) specification.

### Deprecated Fields

#### `spec`

Deprecated for `type`.

#### `first_mes`

Deprecated for `greetings` in `CharacterScenario`.

#### `alternate_greetings`

Deprecated for `greetings` in `CharacterScenario`.

#### `scenario`

Deprecated for `scenario` in `CharacterScenario`.

#### `mes_example`

Deprecated for `example_messages`.

#### `creator`

Moved to `CharacterInfo`.

#### `character_version`

Moved to `version` in `CharacterInfo`.

#### `tags`

Moved to `CharacterInfo`.

#### `creator_notes`

Moved to `CharacterInfo`.

## CharacterScenario Object
This object handles different scenarios and greeting messages that a user can pick from for chatting with the character.
```ts
// New in Prom V3
interface CharacterScenario {
    scenario: string
    greetings: Array<string>
}
```

### Field Descriptions

#### `scenario`

Stores the scenario the user and character are involved in. This **MUST** be a string.

#### `greetings`
Stores the greetings that can be used with the character. THIS **MUST** be an array of strings.

1. This field **MUST** return an empty array if no greetings are present.
2. The first greeting in the greeting list **SHOULD** be treated as the first message of the character.

## CharacterExampleMessage Object
This object handles example messages that a creator can provide to better teach the AI how to speak like the character using example outputs.
```ts
// New in Prom V3
interface CharacterExampleMessage {
    role: "user" | "assistant" | "system"
    content: string
}
```

### Field Descriptions

#### `role`

Stores the sender of the message. This **MUST** be either of the following:
1. "user" - Applications **MUST** add the `{{user}}` macro or the instruct equivalent if the role is a user.
2. "assistant" - Applications **MUST** add the `{{char}}` macro or the instruct equivalent if the role is a assistant.
3. "system" - Applications **SHOULD** handle this sender type as needed.

#### `content`
Stores the contents of the message. This **MUST** be a string.

## CharacterInfo Object
This object handles character metadata more efficiently, making the data field in CharacterCardV3_1 more compact and easier to read.
```ts
// New in Prom V3
interface CharacterInfo {
    creator: string
    version: string
    source: string
    tags: Array<string>
    creator_notes: string
}
```

### Field Descriptions
#### `creator`

Stores the name of the person who created the character. This **MUST** be a string.

#### `version`

Stores the version of the character card (**NOT** the specification). This **MUST** be a string.

#### `source`

Stores the franchise name the character comes from. This **MUST** be a string.

#### `tags`

Stores the tags associated with the character. This **MUST** be a array of string.
> This field **MUST** return an empty array if no tags are added to the character.

#### `creator_notes`

Stores any notes about the character from the character creator. This **MUST** be a string. 
> Applications **MUST** show at least show one sentence of the creator notes.

## CharacterBookV2 Object
```ts
// New in Prom V3
interface CharacterBookV2 {
    type: 'chara_book' // (New)
    spec_version: '2.0' // (New)
    name?: string
    description?: string
    scan_depth?: number 
    token_budget?: number
    recursive_scanning?: boolean
    extensions: Record<string, any>
    entries: Array<CharacterEntry> // (New)
}
```

### Field Descriptions

#### `type`

This **MUST** be set as `chara_book`

#### `spec_version`

For Prom V3.1, this **MUST** be set as `"2.0"`.

#### `name`

(Same as V2/Risu V3) Stores the name of the lorebook. This **MUST** be a string.

#### `description`

(Same as V2/Risu V3) Stores the description of the lorebook. This **MUST** be a string.

#### `scan_depth`

(Same as V2/Risu V3) Stores the depth value the application uses to scan entries recursively. This **MAY** be a number or undefined.

#### `token_budget`

(Same as V2/Risu V3) Stores the max amount of tokens the application can use when scanning for entries in chat context. This **MAY** be a number or undefined.

#### `recursive_scanning`

Determines if a application can recursively scan the lorebook. This **MUST** be a boolean.

#### `extensions`

Stores additional information from content creators, sites, character editors, apps, etc. Same rule applies here as `extensions` in [CharacterCardV3_1](#extensions).

#### `entries` 

Stores the entries contained in the lorebook. This **MUST** be an array of `CharacterEntry` objects.

1. This field **MUST** return an empty object if no entry data is present.
2. **ALL APPLICATIONS, CHARACTER EDITORS, ETC. MUST** follow the [CharacterEntry](#characterentry-object) specification.


### CharacterEntry Object
```ts
// New in Prom V3
interface CharacterEntry {
    name: string
    keys: Array<string>
    secondary_keys?: Array<string>
    content: string
    constant?: boolean 
    selective?: boolean
    enabled: boolean
    insertion_order: number
    case_sensitive?: boolean

    // From RisuAI V3
    use_regex?: boolean
    
    // Optional Fields
    priority?: number // For Agnai
    id?: number | string // For ST/RisuAI
    comment?: string // For ST/RisuAI

    position?: 'before_char' | 'after_char' | 'before_an' | 'after_an' 
    extensions: Record<string, any>
}
```

### Field Descriptions

#### `name`

The name of the entry. This **MUST** be a string. This value **SHOULD NOT** be used in prompt engineering.

#### `keys`

(Same as V2/Risu V3) The keywords to trigger the entry. This **MUST** be an array of strings.

#### `secondary_keys`

(Same as V2/Risu V3) The keywords that can trigger/not trigger the entry. This **MUST** be a string.
> This field **MUST** return an empty array if no secondary keys are added to the character.

#### `content`

(Same as V2/Risu V3) Stores the content of the entry itself. This **MUST** be a string.

#### `constant`

(Same as V2) Stores whether the entry gets added for every message sent. This **MUST** be a boolean or undefined. If result is `undefined`, it is determined to be *False*.

#### `selective`

(Same as V2/Risu V3) Stores the selective mode of the entry (AND ANY, AND ALL, NOT ANY, NOT ALL). This **MUST** be a boolean or undefined. If result is `undefined`, it is determined to be *False*.

#### `enabled`

(Same as V2/Risu V3) Stores whether the entry is active. This **MUST** be a boolean.

#### `insertion_order`

(Same as V2/Risu V3) Stores where to insert the entry in prompt engineering. Lower the number, the earlier the entry gets added and vice-versa. This **MUST** be a number. This value **SHOULD NOT** be used in prompt engineering.

#### `case_sensitive` 

(Same as V2/Risu V3) Stores whether the entry gets triggered by the exact spelling of the word. This **MUST** be a boolean or undefined. If result is `undefined`, it is determined to be *False*.

#### `use_regex`

(Same as Risu V3) Stores whether to use regex to process keywords in `keys`. This **MUST** be a boolean or undefined. If result is `undefined`, it is determined to be *False*.

#### `priority`
> For Agnai use.

(Same as V2/Risu V3) Similar function as [`insertion_order`](#insertion_order). This **MUST** be a number. If `undefined`, use `insertion_order` instead.

#### `id`
> For ST/RisuAI use.

(Same as V2/Risu V3) Stores the ID of the entry in the lorebook. This **MUST** be a number. This value **SHOULD NOT** be used in prompt engineering.

#### `comment`
> For ST/RisuAI use.

(Same as V2/Risu V3) Similar function as [`name`](#name-2). This **MAY** be a string or be left blank. This value **SHOULD NOT** be used in prompt engineering. Applicaitons **SHOULD** preferably use the `name` field instead.

#### `position`

Stores where to insert the entry at. This **MUST** be either: `'before_char'`, `'after_char'`, `'before_an'`, `'after_an'`.

#### `extensions` 

Stores additional information from content creators, sites, character editors, apps, etc. Same rule applies here as `extensions` in [CharacterCardV3_1](#extensions) and [CharacterBookV2](#extensions-1).