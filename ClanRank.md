# ClanRank

`ClanRank` is a property of `Clan` that stores the rank of the player inside the `Clan` object. I made the `ClanRank` class to keep the `Clan` class clean and have the 'responsibility' of clan ranks be a specific class, rather than bloating the `Clan` class.

## Goals
* Keep track of player ranks
* Keep track of ranknames and permissions
* Initiate new rankings if the clan is just being created
* Construct the code so that more ranks can be added as necessary

## Code Structure

The `ClanRank` class is just an extended `Map` class. The functionality of the `ClankRank` is that it would need have X number of unique ranks and that each rank would have similar properties to one another, but different values. The common methods being called would be to check if a player had permission to perform an action based on which rank he/she is in.

Retrieving the rank or ranknumber easily is a benefit of the `Map` class as the `Map.get()` function is native and I can easily check to see if the player is in the rank. The `Map` class was an easy choice as a data structure for this.

```
class ClanRank extends Map {

  constructor(data) {

    let defaultRanks = [
      [1, { rankName: 'Leader', members: new Set(), permissions: new Set() }],
      [2, { rankName: 'Officer', members: new Set(), permissions: new Set() }],
      [3, { rankName: 'Core Member', members: new Set(), permissions: new Set() }],
      [4, { rankName: 'Member', members: new Set(), permissions: new Set() }],
      [5, { rankName: 'Initiate', members: new Set(), permissions: new Set() }],
    ]

    let ranks = data && data.ranks || defaultRanks;
    super(ranks);

    for (const [rankNum, rank] of this) {
      this.set(rankNum, {
        rankName: rank.rankName,
        members: new Set(rank.members),
        permissions: new Set(rank.permissions),
      })
    }
    this.maxRanks = data && data.maxRanks || 5;
  }
```

## Obstacles
One of the biggest obstacles I ran into while implementing this was that I originally wanted to use the [MultiMap](http://www.collectionsjs.com/multi-map) or similar data structure. The problem I ran into when I was testing the functions of this data structure is serializing the data. Because I need to save the `ClanRank` between server boots, I need to be able to JSON serialize the data. As far as I could tell, `MultiMap`s don't support being stringified and reparsed into a new `MultiMap`. I spent a good chunk of time on this, and could not get it to compile correctly.

**Potential Refactor** - This is an area I would like to refactor in the future, but I was able to get 100% functionality I wanted just be using the basic `Map` class. 

## Other Sample Code

### Select Clan helper methods:
```
  getRank(number) {
    return this.get(number);
  }

  getRankNumber(playerName) {
    for (const [rankNum, rank] of this) {
      if (rank.members.has(playerName)) {
        return rankNum;
      }
    }
    return -1;
  }

  hasPermission(player, targetRankNum, permissionString) {
    if (player.level >= C.LEVEL_GOD) {
      return true;
    }
    
    const playerRankNum = this.getRankNumber(player.name)
    const playerRank = this.getRank(playerRankNum);

    if (playerRankNum === 1) {
      return true;
    }

    if (playerRank.permissions.has('all') && playerRankNum < targetRankNum) {
      return true;
    }

    if (playerRankNum >= targetRankNum) {
      return false;
    }

    return playerRank.permissions.has(permissionString);
  }
```

### Sample User Command for a Clan:
Renaming rank #2 to the name 'Officer'.
>clan rankname 2 Officer

If you're in a clan...  
If the player inputting the command has permission to rename the rank...  
If the player typed in args...  
Split and Parse the command args...  
If you submitted both args...  
If the new name of rank doesn't have special characters and between 3 and 15 characters...  
If the new name is different than the previous...  
Then the player updates the name!

```
  subcommands.add({
    name: 'rankname',
    command: state => (args, player) => {

      const clan = player.getClan();
      if (!player.clan || !clan) {
        return say(player, "You must be in a clan to perform those actions.");
      }

      if (!clan.ranks.hasPermission(player, Infinity, 'clanrank')) {
        return say(player, "You dont have permission to rename the clan ranks.");
      }

      if (!args.length) {
        return say(player, "Usage: 'clan rankname <rank number> <new name>'.");
      }

      let [arg1, ...arg2] = args.split(' ');

      arg1 = parseInt(arg1, 10);
      if (isNaN(arg1) || arg1 <= 0 || arg1 > 5) {
        return say(player, `Invalid rank number. Must be from 1 to 5.`);
      }

      arg2 = arg2.join(' ');
      const rank = clan.ranks.get(arg1);

      if (!arg2 || !arg2.length > 0) {
        return say(player, `What do you want to rename rank ${arg1} - '${rank.rankName}' to?`);
      }

      if (hasSpecialCharacter(arg2)) {
        return say(player, `Clan ranks cannot have any special characters. Try again.`);
      }

      if (arg2.length > 16 || arg2.length < 3) {
        return say(player, `Invalid rank name length. Must be between 3 and 15 characters.`);
      }

      arg2 = capitalize(arg2);

      if (rank.rankName === arg2) {
        return say(player, `Rank ${arg1} is already named '${arg2}'.`);
      }

      // TODO: Clan Channel messages
      clan.ranks.updateRankName(arg1, arg2);
      return say(player, `You updated rank name of ${arg1} to '${arg2}'.`);
    }
  });
```  