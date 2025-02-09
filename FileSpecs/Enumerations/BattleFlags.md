# Battle flags

Each monster has special flags that are stored in a 8 bit value. Each bit represents a specific property of the monster. In theory the bits would have the same effects for party members but none of them has a bit set.

This changes in **Ambermoon Advanced** where the bonus spell damage bits can be used by party members as well. Magicians can increase their spell damage through quests.

Bit | Property
----|----
0 | Undead, can be killed by holy spells
1 | Demon (used by Luminor, Guard demon and gargoyle type monsters)
2 | Boss (immune to Fear, Paralyze, Petrify, DissolveVictim, Madness, Drugs, Irritation, critical strikes and won't flee)
3 | Animal (is set for all spider and lizards except for Gigantula). Moranians don't count as lizards though. :P
4 | Increased earth spell damage (same as ice spell damage, **Ambermoon Advanced** only)
5 | Increased wind spell damage (same as ice spell damage, **Ambermoon Advanced** only)
6 | Increased fire spell damage (same as ice spell damage, **Ambermoon Advanced** only)
7 | Unused

Note: After analyzing the original code it seems that only bits 0 and 2 are used in Ambermoon.

Note: The boss flag also protects against holy spells if the monster is also marked as undead.

Note: It is a known bug that Reg is not marked as boss so you could cast Fear on him and he will flee with all the items. So I guess he should have bit 3 set as well. Possibly bit 1 was set for him by mistake.
