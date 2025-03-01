# Event data

Data for map events is stored in the map data files. See [Maps](Maps.md) for more details. Moreover NPCs and party members also store events of the same format for conversations.

Here the data format for specific events is shown. Each event can use up to 9 bytes for the event data.

Note that actually 12 bytes are stored for each event. The first byte is the event type you see in parentheses below and the last 2 bytes always store the index of the following event or 0xffff if none follows. Here only the first 10 bytes are listed which differ for each event type. The last 2 bytes which store the next event index are omitted.

Not all bytes are used by every event. In general you can assume that unused bytes are at the end of the data but as words are aligned to even offsets there might be also a gap of 1 byte in-between to keep the word alignment. Often all words are stored at the end of the event so there might be even more unused bytes in the middle of the event data.


## Teleport event (0x01 / 1)

Used for map transitions, teleporters, holes in the ground and so on.

Offset | Type | Description
--- | --- | ---
0x00 | ubyte | Event type (= 0x01)
0x01 | ubyte | New x coordinate (1-based, 0 means keep x)
0x02 | ubyte | New y coordinate (1-based, 0 means keep y)
0x03 | ubyte | New direction (0: up, 1: right, 2: down, 3: left, 4: keep current)
0x04 | ubyte | [Travel type](Enumerations/TravelType.md) to set after teleport (-1 or 0xff means no change, which is the default)
0x05 | ubyte | Transition type
0x06 | uword | New map index (0 means same map)
0x08 | ubyte[2] | Unused

### Transition types

- 0: Normal (black fading)
- 1: Teleporter (no black fading)
- 2: Wind gate (only teleports if the wind chain is active otherwise a message is displayed)
- 3: Climbing up/levitating (plays a levitation animation while transitioning)
- 4: Outro (after the endboss a map change event will 'teleport' you to the outro sequence)
- 5: Climbing down/falling (plays a falling animation while transitioning)


## Door event (0x02 / 2)

Used for locked doors.

Offset | Type | Description
--- | --- | ---
0x00 | ubyte | Event type (= 0x02)
0x01 | ubyte | Lockpicking chance reduction (0-100, see chest event data)
0x02 | ubyte | Door index (used in savegame to determine if a door was unlocked)
0x03 | ubyte | Optional index of a map text to display when showing door window (0xff means no text)
0x04 | ubyte | Optional index of a map text to display when unlocked (0xff means no text)
0x05 | ubyte | Unused
0x06 | uword | Key index
0x08 | uword | Unlock fail event index (0-based, 0xffff means none, this is basically the trap event chain)


## Chest event (0x03 / 3)

Used for chests, piles, lootable map objects etc.

Offset | Type | Description
--- | --- | ---
0x00 | ubyte | Event type (= 0x03)
0x01 | ubyte | Lockpicking chance reduction (0-100)
0x02 | ubyte | Chest flags (see below, only 1 chest has a value other than 0)
0x03 | ubyte | Optional index of a map text to display when showing the opened chest (0xff means no text)
0x04 | ubyte | Chest data index (also used for locked state in savegame)
0x05 | ubyte | Chest loot flags
0x06 | uword | Key index if locked
0x08 | uword | Unlock fail event index (0-based, 0xffff means none, this is basically the trap event chain)

If the lockpicking chance reduction is 0, the chest is always open. A value of 100 means that the chest can't be lockpicked at all. Many chests have a value of 1 which is a chest that is locked and lockpicking it succeeds nearly with the lockpicking ability in percent as chance.
If the key index is not 0, the chest can't be opened with a lockpick.

### Chest flags

- Bit0: Unused
- Bit1: If set a search ability check is performed and only if successful the chest is shown.
- Bit2-7: Unused

Only one chest in Ambermoon uses the flags. It is a skull in the Antique Area which contains the Antique Weapon. You will only find it when your search ability is high enough cause Bit1 is set. The chest uses value 50 (hex 32).

In binary this is 0011 0010. So in addition two other bits are set as well. But the meaning is unknown.

If I get the 68k assembler right, Ambermoon just checks the whole byte value for "not equal zero". But maybe it has some different meaning in Amberstar?

### Chest loot flags

- Bit0: Remove when empty
- Bit1: No chest auto remove
- Bit2-7: Unused

Bit0 will actually close the chest when you have looted all items, gold and food. The chest event (chain) is then deactivated so that you can't access the chest again. This also disallows storing items in that chest.

Bit1 is only used when Bit0 is set as well. If this is set, the event (chain) is not deactivated so that you can access the chest again after fully looting it. This is used for things like the flowers on Lyramion. You can loot them infinitely. Although the chest popup closes after looting and you can't of course store items there.


## Text popup event (0x04 / 4)

Show a text popup with an optional image. Note that the following map events in the list are only executed after the popup is closed. This is important e.g. for music change events before and after this event.

Offset | Type | Description
--- | --- | ---
0x00 | ubyte | Event type (= 0x04)
0x01 | ubyte | Event picture index (0-based) or 0xff if no image
0x02 | ubyte | Popup trigger (0: none, 1: move, 2: eye cursor, 3: both)
0x03 | ubyte | Also trigger when blind (only 0 or 1)
0x04 | ubyte | Unused
0x05 | ubyte | Map text index
0x06 | ubyte[4] | Unused

Note: There is no explicit trigger for hand cursor. If this is needed, a condition event can be inserted before. The trigger is ignored if the text event is not the first one in an event chain.

Note: In this context, the status "blind" means either the ailment or the fact that you have no light source in a dungeon.


## Spinner event (0x05 / 5)

Only used in 3D maps. Rotates the player to a specific or random direction if he steps onto it.

Offset | Type | Description
--- | --- | ---
0x00 | ubyte | Event type (= 0x05)
0x01 | ubyte | Post-spin [direction](Enumerations/Directions.md)
0x02 | ubyte[8] | Unused

Note: If the direction is random, it will never be the same as before.


## Trap event (0x06 / 6)

Damages the player and/or add ailments (fireplaces, traps, etc).

This is used for normal map traps and also for traps on doors and chests.

Offset | Type | Description
--- | --- | ---
0x00 | ubyte | Event type (= 0x06)
0x01 | ubyte | Ailment
0x02 | ubyte | Trap target (0 = active player, 1 = all party members)
0x03 | ubyte | Affected genders (0 or 3: both, 1: male only, 2: female only)
0x04 | ubyte | Base damage
0x05 | ubyte[5] | Unused

The real damage is `BaseDmg + rand(0, (BaseDmg / 2) - 1)`.
Base damage can be 0 in which case only the ailment is inflicted.

### Trap ailments

- 0: none
- 1: crazy
- 2: blind
- 3: stoned
- 4: paralyzed
- 5: poisoned
- 6: petrified
- 7: diseased
- 8: artificial aging
- 9: dead (corpse)


## Remove buffs event (0x07 / 7)

Removes/stops active buffs.

Offset | Type | Description
--- | --- | ---
0x00 | ubyte | Event type (= 0x07)
0x01 | ubyte | Buff to stop
0x02 | ubyte[8] | Unused

### Buffs

- 0: All
- 1: Light
- 2: Magic attack
- 3: Magic protection
- 4: Anti-magic barrier
- 5: Clairvoyance
- 6: Mystic map


## Riddlemouth event (0x08 / 8)

Offset | Type | Description
--- | --- | ---
0x00 | ubyte | Event type (= 0x08)
0x01 | ubyte | Riddle text index
0x02 | ubyte | Solution text index (used when riddle was solved)
0x03 | ubyte[3] | **Unknown**
0x06 | uword | Correct answer 1 index in word dictionary
0x08 | uword | Correct answer 2 index in word dictionary (can be equal to the first if only 1 word is possible)


## Reward (0x09 / 9)

This gives some reward to the party.

Offset | Type | Description
--- | --- | ---
0x00 | ubyte | Event type (= 0x09)
0x01 | ubyte | Reward type
0x02 | ubyte | Reward operation
0x03 | ubyte | Random (0 or 1)
0x04 | ubyte | Reward target
0x05 | ubyte | Unused
0x06 | uword | Reward type value (e.g. which attribute)
0x08 | uword | Value

If "Random" is set, the real value is a random value between 0 and "Value".

The operation "Fill" will ignore the Value and fully fill. This is used for LP/SP filling.

### Reward type

Value | Meaning
--- | ---
0 | Attribute
1 | Ability
2 | LP
3 | SP
4 | SLP
5 | Ailments
6 | Usable spell types (spell school)
7 | Language
8 | EP
9 | Max attribute (**Ambermoon Advanced** only)
10 | Attacks per round (**Ambermoon Advanced** only, this is overriden by next level up though!)
11 | TP (**Ambermoon Advanced** only)
12 | Level (**Ambermoon Advanced** only)
13 | Damage (**Ambermoon Advanced** only)
14 | Defense (**Ambermoon Advanced** only)
15 | Max LP (**Ambermoon Advanced** only)
16 | Max SP (**Ambermoon Advanced** only)
17 | Empower spell types (value is 0: earth, 1: wind, 2: fire) (**Ambermoon Advanced** only)

### Reward operation

Value | Meaning
--- | ---
0 | Increase
1 | Decrease
2 | Increase by percentage
3 | Decrease by percentage
4 | Fill
5 | Clear bit/remove (e.g. language)
6 | Set bit/add (e.g. language)
7 | Toggle bit (e.g. language)

### Reward target

Value | Meaning
--- | ---
0 | Active player
1 | Whole party
2 | Random player (**Ambermoon Advanced** only)
3 | First animal (**Ambermoon Advanced** only)

**Note:** Operation 'Fill' just sets the current value to the max value. The 3 bit operations should only be used for languages, ailments and spell schools. The percentage is in relation to the max value and should only be used for LP and SP I guess. Using percentage or fill operations on SLP, languages, ailments or spell schools might have strange effects.

**Note:** There is no reward to add training points in Ambermoon. :(


## Change tile event (0x0A / 10)

Offset | Type | Description
--- | --- | ---
0x00 | ubyte | Event type (= 0x0A)
0x01 | ubyte | Tile's x coordinate (1-based)
0x02 | ubyte | Tile's y coordinate (1-based)
0x03 | ubyte[3] | Unused
0x06 | uword | Tile data
0x08 | uword | Map index (0 means same map)

The tile data gives the new tile index for 2D maps and the object or wall index for 3D maps.

In 2D the tile flags of the new tile specifies if the background tile or foreground tile is replaced. If bit 2 of the flags (render order) is set, the foreground tile is replaced, otherwise the background tile. If the new tile index is 0, always the front tile is removed.


## Start battle event (0x0B / 11)

Offset | Type | Description
--- | --- | ---
0x00 | ubyte | Event type (= 0x0B)
0x01 | ubyte[5] | **Unknown**
0x06 | uword | Monster group index
0x08 | uword | **Unknown**


## Enter place event (0x0C / 12)

Offset | Type | Description
--- | --- | ---
0x00 | ubyte | Event type (= 0x0C)
0x01 | ubyte | Text index if place is closed (text taken from map texts, 0xff = no message (used for places that never close like inns))
0x02 | ubyte | Place type (see below)
0x03 | ubyte | Opening hour (0-23)
0x04 | ubyte | Closing hour (0-23)
0x05 | ubyte | Text index when you use the place (text taken from map texts, 0xff = no message). Using a place means buy a horse/ship, buy goods, heal, train, etc.
0x06 | uword | Place data index (1-based index inside the Place_data file)
0x08 | uword | Index (merchant data index)

### Place types

Value | Type
--- | ---
0 | Trainer (all kinds)
1 | Healer
2 | Sage (identify items)
3 | Enchanter
4 | Inn / Sleeping room
5 | Merchant (goods)
6 | Food dealer
7 | Library (spell scroll dealer)
8 | Raft dealer (not used in Ambermoon)
9 | Ship dealer
10 | Horse dealer
11 | Blacksmith (item repair)


## Condition event (0x0D / 13)

Condition events represent conditions that control if following events (in the list) are executed or not. Multiple conditions can be chained which equals a logical AND conjunction.

Offset | Type | Description
--- | --- | ---
0x00 | ubyte | Event type (= 0x0D)
0x01 | ubyte | Condition type
0x02 | ubyte | Condition value (e.g. variable value, always a boolean 0 or 1, can invert the meaning like 1="has"/0="has not")
0x03 | ubyte | Count (item count, etc)
0x04 | uword | Not allowed party member ailments (**Ambermoon Advanced** only)
0x06 | uword | Object index (depends on condition's type, e.g. variable or item index)
0x08 | uword | Map event index to continue with if condition was not fulfilled or 0xffff to stop the event list in this case.

### Condition types

Value | Type
--- | ---
0 | Global variable (game variable, 0-8191)
1 | Event bit (64 bits per map for up to 1024 maps, 0-65535)
2 | Door open (0-255)
3 | Chest open (0-255)
4 | Character bit (32 bits per map for up to 1024 maps, 0-32767)
5 | Party member present
6 | Item owned (item in inventory)
7 | Use item (from inventory)
8 | Keyword known
9 | Success (is chained after other events like battles, riddlemouths or treasures and is something like "battle won", "riddle solved" or "treasure fully looted")
10 | Game option set
11 | Can see (not blind etc)
12 | Player direction
13 | Has ailment
14 | Hand cursor interaction
15 | Say word (mouth + enter keyword)
16 | Enter number
17 | Levitating
18 | Has gold amount
19 | Has food amount
20 | Eye cursor interaction
21 | Mouth cursor interaction (**Ambermoon Advanced only**)
22 | Transport at current location (**Ambermoon Advanced only**)
23 | Multiple cursor interaction (hand, eye, mouth) (**Ambermoon Advanced only**)
24 | Current travel type (**Ambermoon Advanced only**)
25 | Active party member class (**Ambermoon Advanced only**)

**Note:** In conversations the global variable 0 is checked to be value 0 before executing a PrintText event that
should be executed in any case. I guess PrintText events always need a preceding Condition event and the global
variable 0 is always 0. So this is like a "always true condition".

**Note:** For condition 22, the object index gives the [transport type](Enumerations/TravelType.md). But 0 is special and means "any".

**Note:** For condition 23, the object index gives the cursors as bit flags. Bit 0 is hand, bit 1 is eye and bit 2 is mouth.

**Note:** In **Ambermoon Advanced** the word at offset 0x4 specifies which ailments are not allowed for the "Party member present" condition. It is not used for any other condition or in the original. The word specifies a bit for each ailment, see [here](Enumerations/Ailments.md). As this condition is mostly used to let a specific character perform some action or let him say something, a good choice for those flags is 0xe412. This won't allow all dead states, petrify, blind or crazy. You can also add in 0x0100 (paralyzed) if the character has to do something, etc. The blind state 0x0010 might be omitted if the party member just has to head something.


## Action event (0x0E / 14)

Actions are related to conditions. For example they can change variable values which are used in conditions. The structure is very similar. If a condition type checks something, the action type with the same value changes this "something". You'll also find the associated properties of the structure in the same spot. For example the "object index" is at offset 0x06 in condition and action events.

Offset | Type | Description
--- | --- | ---
0x00 | ubyte | Event type (= 0x0E)
0x01 | ubyte | Action type
0x02 | ubyte | Action value (e.g. variable value to set, 0="reset/remove", 1="set/add", 2="toggle/change/invert")
0x03 | ubyte | Count (item count, etc)
0x04 | ubyte[2] | Unused
0x06 | uword | Object index (depends on action's type, e.g. variable index)
0x08 | ubyte[2] | Unused

### Action types

Value | Type
--- | ---
0 | Set global variable (game variable)
1 | Change event bit
2 | Change open door flag
3 | Change open chest flag
4 | Change character bit
5 | Unused
6 | Add/remove items
7 | Unused
8 | Add keyword
9 | Unused
10 | Set game option
11 | Unused
12 | Set direction
13 | Add/remove ailment
14-17 | Unused
18 | Add/remove gold
19 | Add/remove food
20 | Unused

Note: The set game option can theoretically do things like turn on or off music or even activate the fast battle mode. In Ambermoon though it is only used once.
There is a special option which is not visible nor changable in the game but which is set by an NPC event. It is a bit which indicates if you got the yellow sphere
from the Moranian. It is then used to determine which version of the outro should be shown at the end.

Setting an event bit to 1 actually disables the event chain.

Setting a character bit to 1 removes that character from the map. Setting it to 0, adds/spawns it. So this event is used to spawn monsters and NPCs.

## Dice 100 event (0x0F / 15)

Rolls a dice and triggers the following event by a chance.
Optionally an event can be triggered in case the fortune wasn't on your side.

Offset | Type | Description
--- | --- | ---
0x00 | ubyte | Event type (= 0x0F)
0x01 | ubyte | Chance (0..100)
0x02 | ubyte[6] | Unused
0x08 | uword | Map event index if failed


## Conversation event (0x10 / 16)

This is the start of every NPC or party member event chain. It is used as a trigger for interactions.

Offset | Type | Description
--- | --- | ---
0x00 | ubyte | Event type (= 0x10)
0x01 | ubyte | Interaction type
0x02 | ubyte[4] | **Unknown**
0x06 | uword | Value
0x08 | ubyte[2] | **Unknown**

The value's meaning depends on the interaction type.

For show or give item it is the item index.
For keyword it is the dictionary index.

### Interaction types

Value | Type
--- | ---
0 | Keyword
1 | Show item
2 | Give item
3 | Give gold
4 | Give food
5 | Join party
6 | Leave party
7 | Talk (mouth cursor), initial text
8 | Leave (exit conversation)

**Note:** When examining someone, always the first text of the NPC
or party member is used (text index 0). This is not done through
events. Characters which won't open a conversation window can't be examined.

**Note:** This event starts an event chain but will not perform any action. The real action (remove given item/gold/food etc) is done by an Interaction event later.

Talking to an NPC triggers the conversation event chain of interaction type Talk (7).


## Print text event (0x11 / 17)

This is used to define the text to display in conversations.
Therefore it must be part of an event chain which was started by a
conversation event.

Offset | Type | Description
--- | --- | ---
0x00 | ubyte | Event type (= 0x11)
0x01 | ubyte | NPC message index
0x02 | ubyte[8] | Unused

Note: In conversations the global variable 0 is checked to be value 0 before executing a PrintText event that
should be executed in any case. I guess PrintText events always need a preceding Condition event and the global
variable 0 is always 0.


## Create event (0x12 / 18)

Create items, gold or food. Items are stored in the item view of the conversation window, gold and food is directly added to the active party member.

The event chain proceeds immediately so the item has not to be taken to do so.

Offset | Type | Description
--- | --- | ---
0x00 | ubyte | Event type (= 0x12)
0x01 | ubyte | 0: Item, 1: Gold, 2 or above: Food
0x02 | ubyte[4] | Unused
0x06 | uword | Amount (for items only the byte at 0x07 is used)
0x08 | uword | Item index


## Question popup event (0x13 / 19)

Text popup as question with buttons 'Yes' and 'No'.
The next map event is only executed if answered with 'Yes'.
But it is possible to specify a map event for the 'No' case.

Offset | Type | Description
--- | --- | ---
0x00 | ubyte | Event type (= 0x13)
0x01 | ubyte | Map text index
0x02 | ubyte[6] | Unused
0x08 | uword | Index (0-based) of map event to continue with when 'No' is chosen


## Change music event (0x14 / 20)

Changes the currently played music. If the music index 0xff is given,
the default music of the current map is used (= reset music to normal).

There is a byte that is always 0xff. I guessed that this might be the volume. But as it is always 0xff in Ambermoon this could be anything. This could only be tested if the original data is manipulated and then in Ambermoon the volume is checked.

Offset | Type | Description
--- | --- | ---
0x00 | ubyte | Event type (= 0x14)
0x01 | ubyte | Sync flag
0x02 | ubyte | Music index (0x00 = stop, 0xff = default map music or last song, 1-32 are valid song indices)
0x03 | ubyte | Volume (always 0xff -> 100%, not used in original)
0x04 | ubyte[6] | Unused

**Note**: If the sync flag is set (value is not 0), all pending bus cycles are awaited which effectivly syncs the pipelines. In original code this results just in a NOP instruction on the Amiga. The rest of the data is just ignored then. Maybe this was reserved for some future functionality.

**Note**: The volume value is not used in original Ambermoon code. It isn't even read. But I guess this was planned as a volume value.

## Exit event (0x15 / 21)

Used to exit a conversation immediately.
It is used if a conversation should not be continued
and the window should close after the first text is
displayed (e.g. first talk to Father Anthony).

Offset | Type | Description
--- | --- | ---
0x00 | ubyte | Event type (= 0x15)
0x01 | ubyte[9] | Unused


## Spawn event (0x16 / 22)

Spawns a horse, raft, ship, sand lizard or sand ship.

Offset | Type | Description
--- | --- | ---
0x00 | ubyte | Event type (= 0x16)
0x01 | ubyte | X coordinate (1-based)
0x02 | ubyte | Y coordinate (1-based)
0x03 | ubyte | [Travel type](Enumerations/TravelType.md)
0x04 | ubyte[2] | **Unknown**
0x06 | uword | Map index
0x08 | ubyte[2] | **Unknown**

**Note:** Only travel types 1 (horse), 2 (raft), 3 (ship), 9 (sand lizard) and 10 (sand ship) should be used as other travel types can't be spawned.


## Interact event (0x17 / 23)

This event is only used in conversations. It performs the associated action to a chosen interaction type.

This means it will do the following things:
- If player had chosen "Give Item" it will remove that item.
- If player had chosen "Give Gold" it will remove that gold.
- If player had chosen "Give Food" it will remove that food.
- If player had chosen "Ask To Join" it will add the person to the party.
- If player had chosen "Ask To Leave" it will remove the person from the party.
- For all other interaction types (Keyword, Show Item, Talk, Leave) this has no effect at all.

The event has no data. It uses the chosen interaction type from the previous conversation event.

Offset | Type | Description
--- | --- | ---
0x00 | ubyte | Event type (= 0x17)
0x01 | ubyte[9] | Unused

### Example usage

This event is necessary for quests where you need to bring more than 1 item. If the conversation event that is triggered by the "Give Item" button would immediately remove the item you have given, this would be fatal if you don't have the other required item. Therefore the associated action like removing the item is only performed with this extra event. This event can then be chained after some condition events so that it only removes the item if some conditions are met.
