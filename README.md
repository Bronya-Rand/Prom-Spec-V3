# The Prom V3 Character Card Specification (A rewrite of V2 and RisuAI's V3 Character Card format)

This is a character concept of what I (as a AI bot creator and site developer for one AI site) think should be included into a character card regarding the current state of character card specifications for AI bot cards and my knowledge of working in AI for the last year.

For a TL;DR on all of this, see [here](#overall-takeaways). For the specification, see [here](./Concept.md).

> The following below isn't meant to bash on RisuAI or the person who made the V3 format. This is just a critical review about the format overall.

## Purpose/Concerns

I originally looked at the "current" V3 specification that RisuAI has created a while back and even though there have been changes to said spec that many in the past didn't like, I feel there are many things in it that really doesn't make sense for a character card specification myself. 

### CharX (.charx)

Taking a look at the current CharX specification here, it states that CharX is purely a fancy ".zip" file that contains character data in the form of a JSON file as well as including asset data such as PNGs, MP4s, Miku Miku Dance models (MMDs) and such. In my opinion, I feel like this format isn't really the most ideal from both a frontend and creation standpoint as well as just overall use for sites and user local ends.

**The `assets` folder**

I can get that in more modern times that things like character sprites, backgrounds and such have become more at play with the AI community. I am one myself as I have made RVC models and character sprites for several of HoYoverse's Honkai: Star Rail girls as well as Genshin Impact and Honkai Impact 3rd. However there are major concerns regarding handling these things.

- Safetensors/CKPTs/ONYXs

    Most users that use character cards either use a Chat Completion service (OpenAI/Claude/OpenRouter) or use Text Completions (Aphrodite/KoboldCPP/TabbyAPI) that already come with existing models that either comes from the Chat Completion provider or via a user's AI models folder. While it's understandable that a creator might want users to use a specific model for their characters, the truth of the matter is that AI is constantly evolving day by day. Take for example the following example:

    > Ellen Joe releases today (September 27th, 2024) a character card regarding Citlali from Genshin Impact and includes the model "Mistral-Instruct-7B" as part of the CharX file.

    1. While Mistral-Instruct-7B might work for Ellen, maybe the user using the card is not a fan of Mistral-Instruct and would prefer to use Mistral-Nemo or any other model that exists. Users have specific tastes regarding roleplay so including a model into the card not only makes it unnecessary but a waste of space. While I can understand making it easy for newcomers to get into AI with this format, there are more efficient ways to handle this than include a model in the card itself.
    2. Frontends that use their own models (like AI sites) will probably not use the model associated in the card in favor of those hosted on the frontend respectively, making this requirement useless to a heavy degree unless the site is a BYOM (Bring your own model) site that just provides a backend and frontend to you.
    3. Not everyone's system is the same. The specification does not discount whether the model in the CharX archive is FP16, GPTQ, AWQ, or GGUF. 
    
        Say that Ellen included Mistral-Instruct-7B at `FP16` in the card and runs a NVIDIA RTX **A6000** and Jane Doe downloads said card but has a NVIDIA RTX **4060**. 
        
        For one, the A6000 has either 40 or 80GB VRAM depending on the GPU itself while the 4060 only has 8GB VRAM. While this is a drastic example, you can clearly see that while Ellen can run Mistral-Instruct, Jane would be unable to even run the provided model Ellen added to the Citlali card. The same can be said if this was a RTX 3090/4090 or whatever other GPU and any other model if it's Exl2, FP16 or GPTQ 8-bit. Unless Ellen specifically stated that the card comes with FP16 Mistral-Instruct or there is a detection system in the frontend to state "Hey! Ellen added Mistral-Instruct FP16 to Citlali!" any newcomer might hit a quick roadblock with many cards that either:

        1. They might be discouraged to get into AI, thinking it's too much effort.
        2. That creators are inconsiderate of others that don't have powerful hardware compared to them.

        the list goes on and on.

        Frankly, the `Application's MAY reject the CHARX file if the file is too large` seems like a band-aid fix if someone was to slap a FP16 model into a card, but does that mean the responsibility should be put on the creators if they wish to include such a model (even despite me saying that it's probably useless given AI is ever-changing)? Do frontends now have to be responsible for CharX sizes and impose limits making some cards (maybe there only exists one Citlali card at the moment or ever due to her popularity?) What if the creator forgets or doesn't include that information for the public to know about? What if the one downloading it just misses said info or not realize that only X frontend works for the card but not the one they use? What about sites storing the card information? Are we really now going to have things such as "this" on characters?

        | System Requirements | |
        | ------------------- | - |
        | GPU | NVIDIA RTX A6000 (40GB) |
        | Model | Mistral-Instruct-7B (FP16) |
        | Frontend | SillyTavern-CharXEdition (or any CharX frontend that supports XGB) | 
        | HDD/SDD | XGB |

- Code (LUA/JS)

    Programming 101 and common sense would state that you should review any code/app that you are unsure of before executing it as it might lead to malicious behavior happening on your device. Having CharX support JS and LUA code opens the door to potential security risks that may exploit a specific thing on a site platform or a user's device through backdoors in pieces of code on the frontend or through other libraries. How can the average user or frontend (whether self-hosted or site) be sure that Ellen doesn't include a JavaScript file that steals someone's Discord token or downloads a malicious payload that encrypts their files? Yes this is a overexaggeration, but in this digital age, can we really trust what we download? Supporting external code is just a gray area in general to be adding to a card format given that the average person (say Jane Doe) might not be as tech savvy as Hare who might know about CharX being just a "fancy ZIP file", opens it up and notices the strange JS file? 

- Copyrighted Content (PNGs/MP4s/MMDs/Fonts)

    I can understand that the format might be specific to content that is specific to original creations, however, most of the time, bots end up being made out of existing characters (like Sparkle, Rita, Serika or Furina) from existing franchises. There is a slight concern that the assets folder may include copyrighted content such as assets taken from Blue Archive, Genshin Impact, etc. How can a frontend site be sure that what Ellen imports is her responsibility outside of just a `Whatever you upload must be something you are legally able to upload or have gotten permission to upload`? Several companies have already gone after others that hosted works that were fan-made for example Nintendo with Garry's Mod or Take-Two Interactive with GTA mods. What is there to assure that content is purposely obtained with permission or something original? Are sites required to have a moderation queue and do the mod team have to do diligent searching to see if the assets in the card come from a copyrighted work or not? While this isn't something that can be shut off, it does raise concern on how a frontend that DOES handle such content needs to handle itself in terms of hosting and providing said content over to users.

- Network and Storage

    Say that Ellen uploads Citlali with a 3D model of her from a Genshin Impact update, has images from Natlan, and includes Natlan writing fonts with Mistral-Nemo. The size of this CharX, with average ZIP compression from Windows or 7-Zip makes the size of the card itself well over 10-11 GB that a frontend and a user's HDD/SDD has to handle to both download and import to the frontend in question. This is also concerning if someone was to upload said card to a frontend site and has low upload speeds (say 40 Mbps Upload). The question now becomes:

    1. Do character sites now have to provide options to what to include in a CharX file to a user such as if they want to download Citlali with Mistral, 3D Models, fonts and such? 
    2. Do Frontends need to list out what to import from the CharX file itself?

    Not everyone in the world is fortunate to have high network speeds or even unlimited data. Some people might be stuck to a cap of internet by their ISP or suffer from lower speeds due to budget constraints. Is the user just having to sit there as they download.upload the CharX file to include everything at a lower tier plan? 

    What about loading times? Sure, with SSDs, loading times might not be as noticeable but what about older systems but with HDDs? It takes me 3 mins to load Magnum-32B from a HDD and that's 22GB. Would it be similar with a similar sized card?

### CharacterCardV3 as a Object
Before explaining my points with V3, I should show what already exists in both V1 and V2.

#### TavernCardV1
```ts
type TavernCardV1 = {
  name: string
  description: string
  personality: string
  scenario: string
  first_mes: string
  mes_example: string
}
```

#### TavernCardV2 (excluding CharacterBook)
```ts
type TavernCardV2 = {
  spec: 'chara_card_v2'
  spec_version: '2.0' // May 8th addition
  data: {
    name: string
    description: string
    personality: string
    scenario: string
    first_mes: string
    mes_example: string

    // New fields start here
    creator_notes: string
    system_prompt: string
    post_history_instructions: string
    alternate_greetings: Array<string>
    character_book?: CharacterBook

    // May 8th additions
    tags: Array<string>
    creator: string
    character_version: string
    extensions: Record<string, any>
  }
}
```

#### CharacterCardV3
```ts
type CharacterCardV3{
  spec: 'chara_card_v3'
  spec_version: '3.0'
  data: {
    // fields from CCV2
    name: string
    description: string
    tags: Array<string>
    creator: string
    character_version: string
    mes_example: string
    extensions: Record<string, any>
    system_prompt: string
    post_history_instructions: string
    first_mes: string
    alternate_greetings: Array<string>
    personality: string
    scenario: string

    //Changes from CCV2
    creator_notes: string
    character_book?: Lorebook

    //New fields in CCV3
    assets?: Array<{
      type: string
      uri: string
      name: string
      ext: string
    }>
    nickname?: string
    creator_notes_multilingual?: Record<string, string>
    source?: string[]
    group_only_greetings: Array<string>
    creation_date?: number
    modification_date?: number
  }
}
```

In V3 we keep the V1 information for Name, Description, Personality, Scenario, First Message and Message Examples. Additionally, we also have a few V2 things present like Extensions, System Prompt, Creator Notes, Character Books (Lorebooks), Post-History Instructions, Alternate Greetings, and Tags. However there are a few things in here that is either not present in V3 or changed from V1/V2.

### `character_version`
- In V3, Character Version is missing from the JSON itself which several creators and users use to differ between a card version from another such as a Furina card that either focuses on her before Genshin Impact 4.2 (labelled as 1.0-A) and another after said Genshin Impact version (1.0-B). With the removal of such a label, we are basically removing a option to label cards to specific things, mostly relying on creators to let us know differences in bots (if there exists multiple versions of the same bot) or frontends to handle tagging or versioning info. In my opinion, this should have been kept as-is in V3.

There are some good things that do come out of V3 such as `source` to label where the character comes from and `group_only_greetings` to differ normal solo greetings from ones with groups but there are a few things that just doesn't make sense to have as part of the format iself.

### `creator_notes_multilingual`
I feel this option will barely be used as a AI creator. Sure, there might be people who speak other languages that may want to include information about the bot in other languages so people from other places can understand the bot information a creator has provided, but this just doesn't feel neccessary, especially on bot pages and, considering the fact we don't even handle multi-lingual greetings unless it's in alternate greetings, feels rather unneccessary to include. Just adding the additional language stuff as part of `creator_notes` already does the job so this could just not exist overall.

### `nickname`
While I get the intentions of this field to differ from the actual character name, this feels rather redundant as you can just name the character by said name itself. If someone were to use write the name `Elucia de Lute Ima` a.k.a. Elsie from The World God Only Knows or `Luciana Auxesis Theodoro de Montefio` a.k.a. Lucy fron Zenless Zone Zero, most would either write Lucy or Elsie in the card name itself OR add the actual name into the description as either
```
..., real name(Luciana Auxesis Theodoro de Montefio), ...
```
OR
```
Lucy, full name "Luciana Auxesis Theodoro de Montefio" ...
```

If someone were to really use "Luciana Auxesis Theodoro de Montefio" as the character card name, but then add Lucy as part of the description, the same applies, but this also opens up a can of worms regarding token counting. Overall, this is best left to be added as reinforcement to the bot itself, in which the user states the nickname or the nickname is already the name with the full name being referenced in the description. This just feels like a long-winded way to fix something that can be easily fixable by someone in the frontend or by purely telling the AI what name to use.

### `source`
The usage of source being a array of strings is rather niche and probably result in confusion depending on the user's understanding of this field. Say that a user is specifically wanting Labrys from the Persona series. With this source field, there could just be `Persona 4 Arena` OR there could be that and/or `Persona 4 Arena: Ultimax`, `Blazblue: Cross Tag Battle`, etc. The addition of such additional sources can confuse people, especially for characters that have crossed over other games like YoRHa No. 2 Type-B that appears in not just Nier: Automata but also Final Fantasy XIV: Shadowbringers, Fall Guys, Punishing Gray Raven, Naraka, etc. What's to say that the card is specifically written to be 2B in XIV? What about PGR? From what I seen in the site I worked on is that all users (or mostly all of them) use our source field to specify one source than multiples or use a `/` to be indicative of a joint source. IMO, this should just be a literal string that anyone can put whatever in like `Final Fantasy XIV/Nier Automata`, `Final Fantasy XIV, Nier Automata`, etc.

### `assets`
Now here is the one I have a bit to say on...

There are several instances of "*If the `type` is X the asset SHOULD ...*" in this field that feels so vague that I had to translate what was said to better understand what V3 is on about here. Some sections specify that if there exists but 1 thing to use that but multiple can either use X, Y or do Z instead.

While I do see that some of the types are *technically* optional, it is vague as to if the user has certain freedoms outside of if a card contains multiple types in the assets section. IMO, the better way to handle this is this

1. If the type is a background, ask the user IF they want to import and use the backgrounds. Do not force anything upon the user.
2. If the type is a icon of the char, do use it but still allow users the ability to change it on the frontend (unless the site doesn't support custom avatars)
3. If the type is a user icon, you can ask the user to use it but TBH this should just not exist as the user should be left with the choice on how they appear in chats to the character and let their imagination picture the scene. Sure it can work for if the card is X to Y character, but again, user choice is what matters here in my opinion.

Security-wise, I feel that the format should ONLY target stuff embedded in CharX (and that's being gracious ignoring the off-chance of malicious PNGs). While giving HTTPS links is fine, you are basically being held on the mercy of the creator of the stuff to not do bad actions or that the URL exists after X time. Additionally, you are also at the helm that a certain frontend doesn't handle these assets differently (say `admire` is used that `admiration` in one frontend than the other). Mix-matching frontend design sort of prevents a unified way to download/import stuff from a card unless all frontends just stick to one thing and one thing only and the same can be said on CharX or other providers.

#### `name`

> if type is user_icon, name SHOULD be used as a name of the user

I have a big gripe on this. Enforcing a name on the user is NOT what should be done. IDK if this is badly worded or what, but this feels really restrictive just for a user icon existing. Just let the user be who they are with the Persona section. Same applies to anything else with this of enforcement.

### Lorebooks

```ts
    //On V2 it was optional, but on V3 it is required to implement
    constant?: boolean
```

There isn't really a need for this to be required. `constant?` is already optional that it is either `true`, `false` or `undefined`. If it's undefined, then it should be false and any declaration of a bool should be treated as such. (Also the format still lists this as `constant?` so the whole `V3 it is required to implement` part is untrue).

## Overall Takeaways
Like other people, I am personally not a fan of the current V3 card layout proposed by RisuAI. I feel most of things they are trying to add to this V3 is just completely unneccesary or just only exists for the purposes of RisuAI itself. The list here is already long as it is with asset inclusions that make no sense for use in the future, security issues related to code, costly bandwith for users and frontends to download and/or process, fields that make no sense to exist, the list goes on and on. The character card format should be simple enough to be a character card, not a character pack. Sure character packs might be nice or a AIO character card with everything, but with everything in AI nowadays from chatting on Claude to running Magnum-32B locally on your PC on Exl2, etc. there is just so much that exists here that just makes little sense in why it should be included. Why exactly do we need AI models in a character card? Why do we need to enforce some restrictions not present in V1 or V2? IMO the format feels rather jank and should be changed which is why I propose a more simpler V3 be done that makes genuine sense for just normal use than adding on additional things that, at the end of the day, are purely optional for people to use or may be obsolete as time goes by.