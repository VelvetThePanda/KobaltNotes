This document aims to layout the basic design of a potential anti-raid system in Kobalt, as well as throwing down ideas for a design. This document is not meant to be taken as a concrete plan, but rather inspiration for what could become a final product.



# Idea One: Sliding Window Checks

This idea is the premise that there is a sliding window in which users are allowed to join, and anything beyond that is considered a raid. 

This is the [current implementation](https://github.com/VelvetThePanda/Silk/blob/df45b291863bf8451694b4a25345b38b17600b1f/src/Silk/Services/Guild/AutoMod/RaidDetectionService.cs#L48-L139) in Silk!, and works fine for the most part, but is inherently limited by this very black and white interpretation of an anti-raid "algorithm".

### This will ***NOT*** be used in Kobalt.

---

# Idea Two: Algorithmic approach

This idea is far more sophisticated in technique, and applies an actual algorithm for determining suspicious activity.

Several factors are taken into account when determining whether a user is suspicious.

- Their account age
- When they joined
- Whether they have an avatar
- The sum of threats from all users in the current time slot
- The invite used to join<sup>1</sup>
- The badges of their account<sup>1, 2</sup>

So how does this work? Firstly, the guild's configuration will look something like this:

```cs
public record GuildRaidConfig
(
	int BaseJoinScore,
	int ThreatScoreThreshold,
	int NoAvatarScore,
	int JoinVelocityScore,
	int MinimumAccountScore,
	int SuspiciousInviteScore,
	int SuspiciousUserScore,
	TimeSpan MinimumAccountAge,
	TimeSpan LastJoinBufferPeriod,
	TimeSpan AntiRaidTimeSlotDuration,
	TimeSpan? MinimumAccountAgeBypass,
	AccountFlags? AccountBadgeBypasses
);
```

<sup>1</sup> If configured
<sup>2</sup> Selectable badges are:
- `Early Nitro Supporter`, 
- `HypeSquad Events`,
- `Discord Staff`, 
- `Discord Moderator Alumini`
- `Bug Hunter Level 1`/`Bug Hunter Level 2`
- `Early Verified Bot Developer`

Most things should be pretty self explanatory, however the "suspicious invite" more arbitrary.

Effectively, when an invite is made, Kobalt keeps track of *when* the invite was made, and *who* made it. From this, it's used to track suspicious behavior (raiders occasionally make a new invite, and just use that, but not always.) 

User joins ➜ Creates invite ➜ Another user joins ➜ Both user's threat scores increase.

This is the "invite check" of this graph:
![[4Kjtp.png]]

The Acc[ount] age check is the aforementioned check about the user's minimum age. Simply, if the user's account age is too new (say, made within the last two weeks)