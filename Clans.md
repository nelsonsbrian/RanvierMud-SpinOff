# Clans
Clans are a social grouping structure that allows players to be a member of a 'clan', 'guild', or 'tribe'. Being a member of the same clan has the players able to share a common chat channel between members, have access to a clan-based storage system, and have a unique clan area that is only accessible by members.
## Goals
* Implment the clan system
* Add clan storage and persist the data of items stored
* Add 'rankings' so players can be 'officers' or 'members'
* Add 'permissions' so the clan leader can assign available actions in the clan to members of a certain rank (ie able to withdraw from clan storage if you're an officer)
* Add stats tracking/metadata to Clans. Keep track of kills, losses, and other stats.
* Add clan-based quests
* Add clan-based effects (buffs/debuffs)

## Major Properties
* [ClanRanks](ClanRank.md)
* GlobalQuestTracker
* Locker (persistant storage)
* GlobalEffectList (buff/debuff)

## Code Structure

Ranvier does not have any native support or modules to base this structure off of. What I came up with when planning it out is something like:

```
class Clan extends Metadatable(EventEmitter) {
  constructor(id, data, GameState) {
    super();

    this.id = id;
    this.isActive = data.isActive || true;
    this.name = data.name;
    this.members = new Set(data.members);
    this.onlineMembers = new Map();
    this.level = data.level || 1;
    this.createdDate = data.createdDate || new Date().toJSON();
    this.createdBy = data.createdBy || '';
    this.leader = data.leader || '';

    this.home = new Map(data.home);
    this.ranks = new ClanRank(data.ranks);
    this.areas = new Map();

    this.invited = new Set(data.invited);

    const active = data.quests && data.quests.active || [];
    const completed = data.quests && data.quests.completed || [];
    this.GlobalQuestTracker = new GlobalQuestTracker(null, active, completed, GameState, id);
    this.GlobalEffectList = new GlobalEffectList(null, data.effects, GameState, id);

    this.locker = new InventoryEx(data.locker || {});
    this.metadata = data.metadata || {};
  }
```

## Obstacles
The big obstacle for this feature is just the shear size of the class. I had written out a list of properties and features I thought this class was going to have, and the actual implementation ended up being more than my plan.

## TO-DO Remaining
What I haven't done yet and may wait until near or after deployment is hooking in more `MetaData` sort of stats. I am tracking basic kills and have some of the properties for other stats like: quests completed, time in combat, treasures found. I have not put in the code that would increment most of those stats at momement, but would be a create opportunity to come back and add more data tracking as more features get implemented.

**Thoughts on data:** One of the great opportunities that I see when we track quest completions either with clanquests or quests themselves is being able to track which quests are being completed, started but not completed, or not even tried at all. That sort of data is invaluable to be able to investigate bugs or issues into why a certain feature is not 'fun' or acessable. It would be hard to tell which quests were not being ran without that sort of data.

## Other Sample Code

### Select Clan helper methods:
```
  login(state, player) {
    this.onlineMembers.set(player.name, player);
    this.GlobalQuestTracker.updateRestrictions(state, player);
  }

  logout(state, player) {
    this.onlineMembers.delete(player.name);
    this.GlobalQuestTracker.removeFromQuests(state, player);
  }
```

### Sample Screenshots

The command `clan rankpermissions` allows the leader to see what each ranking has permission to do. Also, which player is in which rank.
![ClanPermissions](/screenshots/RankPermissions.PNG)

The command `clan info <clanNumber>` displays basic information about the clan as well as basic stats like CPK (Chaotic Player Kills) and NPK (Nuetral Player Kills) which are different types of player-vs-player interations.
![ClanInfo](/screenshots/ClanInfo.PNG)