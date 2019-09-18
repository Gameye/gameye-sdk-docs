# Gameye SDK Documentation

## Introduction

Create competitive matches and game sessions for your platform or game without any fixed
monthly costs or any need for your own server infrastructure.

Simply implement the Gameye API to start sessions all over the world. If the game support statistics this will be available in realtime.

# Available SDK options
The Gameye SDK's are the recommended way to use our API.

Our SDK's are available in several popular languages or environments:
- Node.js (Typescript)
- Golang
- PHP
- Raw REST API (curl)

In these pages we well show you how easy it is to use the SDK's and the raw
REST API.

## Getting started

### Get an API KEY

Obtain a free Gameye API key, please send us an
[email](mailto:support@gameye.com).

All API request need to be authorised using your `GAMEYE_API_TOKEN`. You can
either use this token as a [bearer token](https://tools.ietf.org/html/rfc6750)
in the http `Authorization` header if you plan to make raw http request, or
configure it in the api client.

### Install the SDK
Follow the instructions for your language of choice bellow. In case of any
trouble contact our support department for assistance.

#### Install the SDK (Node.js)
Via npm:
```bash
npm install @gameye/sdk -s
```

#### Install the SDK (Go)
Via dep:
```bash
dep ensure -add github.com/Gameye/gameye-sdk-go/clients
```


#### Install the SDK (PHP)
Via composer
```bash
composer require gameye/gameye-sdk-php
```

# Create a Gameye client
The SDK's provide a Gameye client class, you need to pass your
`GAMEYE_API_TOKEN` and the API endpoint `GAMEYE_API_ENDPOINT` (usually
https://api.gameye.com). You can also set the `GAMEYE_API_TOKEN` and
`GAMEYE_API_ENDPOINT` environment variables.

In case of raw API use, use one of the available endpoints and add the
`GAMEYE_API_TOKEN` in the authorization header.

## Initiate a Gameye Client (Node.js)
```typescript
import {
    GameyeClient,
} from "@gameye/sdk"

const client = new GameyeClient({
    endpoint: "https://api.gameye.com",
    token: "GAMEYE_API_TOKEN",
});
```

## Initiate a Gameye Client (Go)
Use `NewGameyeClient` to instantiate a new client and use a
`clients.GameyeClientConfig` to configure it.

```go
import (
	"github.com/Gameye/gameye-sdk-go/clients"
)

gameye := clients.NewGameyeClient(clients.GameyeClientConfig{
    Endpoint: "https://api.gameye.com",
    Token: "GAMEYE_API_TOKEN"
})
```


## Initiate a Gameye Client (PHP)
```PHP
$gameye = new \Gameye\SDK\GameyeClient([
    'AccessToken' => 'GAMEYE_API_TOKEN',
    'ApiEndpoint' => 'https://api.gameye.com',
]);
```

# Concepts
Here we describe the concepts that you need to understand to use the SDK.

## Commands and queries
The Gameye SDK's (and API) are desigend with the
[Command and Query Responsibility Segregation](http://codebetter.com/gregyoung/2010/02/16/cqrs-task-based-uis-event-sourcing-agh/)
design pattern in mind.

The api provide a couple of queries that will get state from the API, queries
will never alter state. State may be retrieved in real-time or as a snapshot.
An example of a query is the `statistic` query that will give back the
statistics from a match. These may be queries in real-time to make a scoreboard,
but can also be queries as a snapshot to process the match after it has ended.

Commands may alter state, but will never retrieve state. They will only report
an error if it occurs. If you issue a command that gives no result or error
back everything is ok! An example of a comaand is the `start` command that
will start a match.

## Selectors
You may use the retrieved state directly, however we urge you to use the
provided selectors. Also, if you want to retrieve something from the state that
is not provided by the selectors, make a new one! And if it is a selectors that
could be useful for other parties, please make a pull request!

## Workflow
So, in order change state, use a command!

If you want to retrieve state from the api, first decide what state you want to
retrieve and if you want to subscribe to this state or simple retrieve a
snapshot.

Then, when you have the state pass it to a selector to get the data you are
looking for.

# Usage
How to use the SDK's.

## Start a match

At the command site of the API we have two commands, one to `start` a match an one to `stop` a match.

`Command`s will not give a result back (just a status code indicating success or error), 
therefore you need to provide your own unique `matchKey` as a reference for the match.

In the sample code we a timestamp with sufficient large resolution as a match key, but any unique (string) ID will do.

To `start` a match you need the following input:
1. a `gameKey` to specify the game (e.g. "csgo")
1. a list of `locationKeys` where you want your match to be hosted
1. a `templateKey` to select a known game template (see above)
1. a unique `matchkey` as a reference for the match
1. optional a set of match configuuration parameters (see above)


### Start and stop a match (raw API)

In case of the raw API you simply POST all argumments 
to the `/action/start-match` like this:

```bash
curl \
--request POST \
--header "Content-Type: application/json" \
--header "Authorization: Bearer GAMEYE_API_TOKEN" \
--data '{
    "locationKeys": ["amsterdam"],
    "gameKey": "csgo",
    "templateKey": "esl1on1",
    "matchKey": "YOUR_UNIQUE_MATCH_KEY",
    "config": {
      "steamToken": "YOUR_STEAM_TOKEN",
      "maxPlayers": 12,
      "tickRate":  128,
      "maxRounds": 6,
      "gameType": 0,
      "gameMode": 1,
      "mapgroup": "mg_active",
      "map": "de_dust",
      "teamNameOne": "TeamA",
      "teamNameTwo": "TeamB"
   }
}' \
https://api.gameye.com/action/start-match
```

and to `stop` use the `/action/stop-match` end point

```bash
curl \
--request POST \
--header "Content-Type: application/json" \
--header "Authorization: Bearer GAMEYE_API_TOKEN" \
--data '{
    "matchKey": "YOUR_UNIQUE_MATCH_KEY"
}' \
https://api.gameye.com/action/stop-match
```

The result wil be a HTTP status code:
- a `204` 'NO CONTENT' in case of a successful started match
- a `200` 'SUCCESS' in case of a successful stopped match
- a `401` 'AUTHORISATION NEEDED' in case of a missing `GAMEYE_API_TOKEN` 
- a `403` 'NOT ALLOWED' in case of an invalid `GAMEYE_API_TOKEN` 
- a `404` 'NOT FOUND' in case of a non existing game (template)
- a `500` in case of any mistakes in the posted payload
  

### Start and stop a match (Node.js)

```typescript
async function start_stop_match(gameye: GameyeClient){
    const gameKey: string = "csgo";
    const locationKeys: string[] = ["amsterdam"];
    const templateKey: string = "esl1on1";
    const matchConfig = {
     "steamToken": "",
     "maxPlayers": 12,
     "tickRate":  128,
     "maxRounds": 6,
     "gameType": 0,
     "gameMode": 1,
     "mapgroup": "mg_active",
     "map": "de_dust",
     "teamNameOne": "TeamA",
     "teamNameTwo": "TeamB"
   };
    const matchKey: string = "" + new Date().getTime(); // we use a timestamp as a unique gameKey

    console.log("\nlet's START a match with key", matchKey , " ...");
    try {
        await gameye.commandStartMatch(matchKey, gameKey, locationKeys, templateKey, matchConfig);
    } catch (error) {
        console.error("Sorry: commandStartMatch call failed: ", error);
        return -1;
    }
    console.log("...done");
    
    // start handling match updates here ...
   
    console.log("\nlet's STOP the game ...");
    try{
        await gameye.commandStopMatch(matchKey);
    } catch (error) {
        console.error("Sorry: commandStartMatch call failed: ", error);
        return -1;        
    }
    console.log("...done");

}
```
## Start and stop a match (Go)

Using the `Go` SDK we can call the `CommandStartMatch` and `CommandStopMatch` commands after
instatiating the client, passing in all arguments needed to start a match, including a unique
`matchKey`. The command calls just return an `error`. If the `error` is `nil` you should use
a `QueryMatch` call and the `SelectMatchItem` selector to get the match handle that contains
information about the started match.
 
```go
package main

import (
    "fmt"
    "time"
    "github.com/Gameye/gameye-sdk-go/clients"
    "github.com/Gameye/gameye-sdk-go/models"
    "github.com/Gameye/gameye-sdk-go/selectors"
)

func main() {

    api_config := clients.GameyeClientConfig{Endpoint: "https://api.gameye.com", Token: "GAMEYE_API_TOKEN"}
    gameye := clients.NewGameyeClient(api_config)

    // all params we need to start a match
    gameKey := "csgo"
    locationKeys := []string{"amsterdam"}
    templateKey := "esl1on1"
    matchConfig := map[string]interface{}{
     "steamToken": "",
     "maxPlayers": 12,
     "tickRate":  128,
     "maxRounds": 6,
     "gameType": 0,
     "gameMode": 1,
     "mapgroup": "mg_active",
     "map": "de_dust",
     "teamNameOne": "TeamA",
     "teamNameTwo": "TeamB",
    };
    // we need a unique matchKey lets use a timestamp ...
    matchKey := fmt.Sprintf("%d", time.Now().UnixNano())

    var err error

    // start a match
    fmt.Printf("\nlet's START a match with key %s ....\n", matchKey)

    err = gameye.CommandStartMatch(matchKey, gameKey, locationKeys, templateKey, matchConfig);
    if err != nil {
        fmt.Printf("\nSorry: CommandStartMatch call failed:  %s\n", err)
        return
    }

    fmt.Printf("...done\n")

    // check for active matches
    var available_matches *models.MatchQueryState

    available_matches, err = gameye.QueryMatch()
    if err != nil {
        fmt.Printf("\nSorry: queryMatch call failed:  %s\n", err)
        return
    }

    // select the match handle
    my_match := selectors.SelectMatchItem(available_matches, matchKey)

	fmt.Printf("\n%#v\n", my_match)

	fmt.Printf("\nThe match is running at host %s at ports %d (game) and %d (gotv)\n", my_match.Host, my_match.Port["game"], my_match.Port["gotv"])

    // stop the match
    fmt.Printf("\nlet's STOP the match with key %s ....\n", matchKey)

    err = gameye.CommandStopMatch(matchKey);
    if err != nil {
        fmt.Printf("\nSorry: CommandStopMatch call failed:  %s\n", err)
        return
    }

    fmt.Printf("...done\n")

}
```

## Listen to real time updates of a running match

After you stared a match you can listen to updates from the live match using a
view simple `query` calls.

If you have any matches in progress you can fetch the state using the  `match`
`query`.

### Match data sample (json)

The match query list all matches indexed by `matchKey` 
```json
{ 
    "match": { 
     "1536150262360": null,
     "1536150484784": null,
     "1536150547618": {
        "matchKey": "1536150547618",
        "gameKey": "csgo",
        "locationKey": "amsterdam",
        "host": "213.163.71.5",
        "created": 1536150547865,
        "port": { 
          "game": 50716, 
          "gotv": 54478
        }
     }
}
```

### Query match (raw API)

```bash
curl \
--header "Authorization: Bearer GAMEYE_API_TOKEN" \
https://api.gameye.com/fetch/match
```

### Query match state (Node.js)

```typescript

async function request_match(gameye: GameyeClient, matchKey: string){

    // get all info about running matches
    let all_matches: MatchQueryState;
    try {
        all_matches = await gameye.queryMatch(); // TODO: suggest to pass matchKey for this query also
        console.log("  - my match = ", all_matches.match[matchKey]);
    } catch (error) {
        console.warn("Problem with queryMatch :", error);
    }
}
```

### Query match state (Go)

```go
    available_matches, err = gameye.QueryMatch()
    if err != nil {
        fmt.Printf("\nSorry: queryMatch call failed:  %s\n", err)
        return
    }
```

### Match selectors

You are encouraged to use: 
- `selectMatchList` to get a list of all active matches (may be empty list `[]`)
- `selectMatchListForGame` to get a list of all active matches for a given `gameKey` (may be empty empty list `[]`)
- `selectMatchItem` to get the active match item for a given `matchKey` (may be empty `null`)

The above code can be rewritten using selectors like this:

```typescript

import { MatchItem, selectMatchItem, selectMatchList, selectMatchListForGame } from "@gameye/sdk";

async function request_match(gameye: GameyeClient, gameKey: string, matchKey: string){

    console.log("\nlet's request a list of all running matches ...");
    let match_query_state: MatchQueryState;
    try {
        match_query_state = await gameye.queryMatch(); // TODO: suggest to pass matchKey for this query also
    } catch (error) {
        console.warn("Problem with queryMatch :", error);
    }
    console.log("...done");

    const all_matches: MatchItem[] = selectMatchList(match_query_state);
    console.log(all_matches);

    const my_matches:  MatchItem[] = selectMatchListForGame(match_query_state, gameKey);
    console.log(my_matches);

    const my_match: MatchItem = selectMatchItem(match_query_state, matchKey);
    console.log(my_match);
}
```

### Query match state  (PHP)

```php

$matches = $client->queryMatch();

echo "<ul>";
foreach($matches->match as $match) {

    echo "<li>";
    echo "Match ID: ".$match->matchKey."<br />";
    echo "Game: ".$match->gameKey."<br />";
    echo "Location: ".$match->locationKey."<br />";
    echo "Game connect: ".$match->host.":".$match->port->game."<br />";
    echo "GOTV connect: ".$match->host.":".$match->port->gotv."<br />";
    echo "</li>";
}

echo "</ul>";

$filteredMatches = GameyeSelector::selectMatchListForGame($matches, 'csgo');

$singleMatch = GameyeSelector::selectMatchItem($matches, $matchKey);
```

### Query match state  (Golang)
Using `QueryMatch` we get a `models.QueryMatchState` structure wih all
active match data. You can use the following  `selectors` to extract information you need: 
- `SelectMatchList` to get a list of all active matches (may be empty list `[]`)
- `SelectMatchListForGame` to get a list of all active matches for a given `gameKey` (may be empty empty list `[]`)
- `SelectMatchItem` to get the active match item for a given `matchKey` (may be empty `null`)

```go
package main

import (
    "fmt"
    "github.com/Gameye/gameye-sdk-go/clients"
    "github.com/Gameye/gameye-sdk-go/models"
    "github.com/Gameye/gameye-sdk-go/selectors"
)

func main() {

    api_config := clients.GameyeClientConfig{Endpoint: "https://api.gameye.com", Token: "GAMEYE_API_TOKEN"}
    gameye := clients.NewGameyeClient(api_config)

    // check for active matches
    var err error
    var available_matches *models.MatchQueryState

    available_matches, err = gameye.QueryMatch()
    if err != nil {
        fmt.Printf("\nSorry: queryMatch call failed:  %s\n", err)
        return
    }

    // use selectors to access the results

    matches := selectors.SelectMatchList(available_matches)
	fmt.Printf("\n%#v\n", matches)

    gameKey := "csgo"

    matches_for_game := selectors.SelectMatchListForGame(available_matches, gameKey)
	fmt.Printf("\n%#v\n", matches_for_game)

    // if we have a matchKey you may want to use that
    matchKey := "MY_MATCH_KEY"

    my_match := selectors.SelectMatchItem(available_matches, matchKey)
	fmt.Printf("\n%#v\n", my_match)
}

```

## Query  / Subscribe Statistics 

After you stared a match you can listen to updates to the match statistics
similar as the match state itself.

If you using one of our SDK's you can listen to updates of the 
statistics in __real time__ using a `subscription`. This is the
preferred and most convenient way to observe a match.

If you want an `one time` status update of a match 
(e.g. after the match has ended) or if you cannot use 
one of de SDK's, you may poll the statistics using (raw API) `Query` 
calls.

If you have any matches in progress you can fetch the state using
the  `statistic` `query` or `subscribe` given a valid `matchKey`.


### Statistic data structure (json)

The `statistic` `query` / `subscribe` returns a json object with one attribute
`statistic` where we find the attributes:
1. `start`(number): a timestamp when the match was started
1. `stop` (number): a timestamp when the match was stopped of `null` id the match is still in progress
1. `startedRounds` (number): number of match rounds started
1. `finishedRounds` (number): number of match rounds finished
1. `player` (object): information about all players indexed by `playerKey`
1. `team` (object): information about all teams indexed by `teamKey`

The `player` information (model) has the following attributes:
1. `connected` (boolean): , the connected state of the player
1. `playerKey` (string): unique key of a player
1. `uid` (string): Steam id of the player
1. `name` (string) nickname of the player "Xander",
1. `statistic` (object): score statistics (numbers) of the player 
    indexed by `scoreKey` (e.g. `assist`, `death`, `kill`) actual statistic
    keys depend on game and gamemode

The `team` information (model) has the following attributes:
1. `teamKey` (string): key of the team
1. `name` (string):  name of the team
1. `statistic` (object): team scores (numbers) indexed by `scoreKey`
1. `player` (object): team members (booleans) indexed by `playerKey`

A sample looks like this:

```json
{
	"statistic": {
		"start": 1536333217000,
		"stop": null,
		"startedRounds": 1,
		"finishedRounds": 0,
		"player": {
			"3": {
				"connected": true,
				"playerKey": "3",
				"uid": "BOT",
				"name": "Xander",
				"statistic": {
					"assist": 0,
					"death": 1,
					"kill": 1
				}
			},
			"4": {
				"connected": true,
				"playerKey": "4",
				"uid": "BOT",
				"name": "Hank",
				"statistic": {
					"assist": 0,
					"death": 1,
					"kill": 0
				}
			}
		},
		"team": {
			"1": {
				"teamKey": "1",
				"name": "TeamA",
				"statistic": {
					"score": 0
				},
				"player": {
					"4": true,
					"5": true,
					"8": true
				}
			},
			"2": {
				"teamKey": "2",
				"name": "TeamB",
				"statistic": {
					"score": 0
				},
				"player": {
					"3": true,
					"6": true,
					"7": true
				}
			}
		}
	}
}
```
    
### Query Statistic (raw API)

Just `GET` the statistic of any running match using the `matchKey` query parameter:

```bash
curl \
--header "Authorization: Bearer GAMEYE_API_TOKEN" \
https://api.gameye.com/fetch/statistic?matchKey=VALID_MATCH_KEY
```

### Query statistic (Node.js)

For a one time request for the match statistics we can use a `queryStatistic` call:

```typescript
async function get_match_stats(gameye: GameyeClient, matchKey: string){

    // one time status request for a match

    let match: StatisticQueryState;

    // use QueryStatistic to get the final scores of the match
    try{
        match = await gameye.queryStatistic(matchKey);
        console.log("  - stats = ", match.statistic);
        for( const team in match.statistic.team ){
            console.log("     - team ", team, " --> ", match.statistic.team[team]);
        }
        for( const player in match.statistic.player ){
            console.log("     - player ", player, " --> ", match.statistic.player[player]);
        }
    } catch (error) {
        console.warn("Problem with queryStatistic :", error);
    }
}
```

### Query statistic (Go)

Using the `QueryMatch` and some or the related selector is illustrated in the
following `Go` code (this can be used for a running or a finished match):

```go

    match_state, err := gameye.QueryStatistic(matchKey)
    if err != nil {
        fmt.Printf("\nSorry: QueryStatistic call failed:  %s\n", err)
        return
    }

    fmt.Printf("match has %d started rounds", match_state.Statistic.StartedRounds)
    fmt.Printf("match was started at %d", match_state.Statistic.Start)
    fmt.Printf("match was stopped at %d", match_state.Statistic.Stop)  // match_state.Statistic.Stop == 0 while it is running
    fmt.Printf("match has %d started rounds", match_state.Statistic.StartedRounds)
    fmt.Printf("match has %d finished rounds", match_state.Statistic.FinishedRounds)

    // use selectors to extract information about the match state
    player_list := selectors.SelectPlayerList(match_state)

    fmt.Printf("\nPayer stats for this match:\n")

    for i, player := range player_list {
        fmt.Printf("  - player %d (%s) --> (Name: %s, UID: %s)\n", i, player.PlayerKey, player.Name, player.UID)
        for stat_key, stat_value := range player.Statistic {
            fmt.Printf("     - player stat: %s --> %d\n", stat_key, stat_value)
        }
    }

    team_list := selectors.SelectTeamList(match_state)
 
    for i, team := range team_list {
        fmt.Printf("  - team %d (%s) --> (Name: %s, UID: %s)\n", i, team.TeamKey, team.Name)
        for stat_key, stat_value := range team.Statistic {
            fmt.Printf("     - team stat: %s --> %d\n", stat_key, stat_value)
        }
        for player_key, is_part_of_team := range team.Player {
            if is_part_of_team {
               fmt.Printf("     - team has player: %s\n", player_key)
            }
        }
    }
```

### Subscribe statistic (Node.js)

To observer the changes in the match statistics we can use `subscribeStatistic`
this will result in events for each new match state. This is much more friendly than
hard polling using bare `queryStatistic` calls.

***see example in SDK repository***


### Team and player selectors

we can rewrite this using the provided selectors:
- use `selectTeamList` to get a list of `TeamItem`s
- use `selectTeam` to get a single `TeamItem` by `teamKey`
- use `selectPlayerList` to get a list of `PlayerItem`s
- use `selectPlayerItem` to get a single `PlayerItem` by `playerKey`
- use `selectPlayerListForTeam` to get a list of `PlayerItem`s for given `teamKey`

***see example in SDK repository***

### Subscribe statistic (Go)

In this final Go program it all comes together, note the use of 
`SubscribeMatch`, `NextState` and the selectors to observe all
relavent changes during the livetime of the match.

```go
package main

import (
    "fmt"
    "time"
    "github.com/Gameye/gameye-sdk-go/clients"
    "github.com/Gameye/gameye-sdk-go/models"
    "github.com/Gameye/gameye-sdk-go/selectors"
)

func main() {

    api_config := clients.GameyeClientConfig{Endpoint: "https://api.gameye.com", Token: "GAMEYE_API_TOKEN"}
    gameye := clients.NewGameyeClient(api_config)

    // all params we need to start a match
    gameKey := "csgo"
    locationKeys := []string{"amsterdam"}
    templateKey := "esl1on1"
    matchConfig := map[string]interface{}{
     "steamToken": "",
     "maxPlayers": 12,
     "tickRate":  128,
     "maxRounds": 6,
     "gameType": 0,
     "gameMode": 1,
     "mapgroup": "mg_active",
     "map": "de_dust",
     "teamNameOne": "TeamA",
     "teamNameTwo": "TeamB",
    };
    // we need a unique matchKey lets use a timestamp ...
    matchKey := fmt.Sprintf("%d", time.Now().UnixNano())

    var err error

    // start a match
    fmt.Printf("\nlet's START a match with key %s ....\n", matchKey)

    err = gameye.CommandStartMatch(matchKey, gameKey, locationKeys, templateKey, matchConfig);
    if err != nil {
        fmt.Printf("\nSorry: CommandStartMatch call failed:  %s\n", err)
        return
    }

    fmt.Printf("...done\n")

    // check for active matches
    var available_matches *models.MatchQueryState

    available_matches, err = gameye.QueryMatch()
    if err != nil {
        fmt.Printf("\nSorry: queryMatch call failed:  %s\n", err)
        return
    }

    // select the match handle
    my_match := selectors.SelectMatchItem(available_matches, matchKey)

	fmt.Printf("\n%#v\n", my_match)

	fmt.Printf("\nThe match is running at host %s at ports %d (game) and %d (gotv)\n", my_match.Host, my_match.Port["game"], my_match.Port["gotv"])

    // listen to live updates during the match

    match_sub, err := gameye.SubscribeStatistic(matchKey)
    if err != nil {
        fmt.Printf("\nSorry: SubscribeStatistic call failed:  %s\n", err)
        return
    }

    match_is_running := true
    for match_is_running {
        match_state, err := match_sub.NextState()
        if err != nil {
            fmt.Printf("\nSorry: NextState call failed:  %s\n", err)
            return
        }
        match_is_running = match_state.Statistic.Stop == 0
        fmt.Printf("\n\nmatch has %d started rounds\n", match_state.Statistic.StartedRounds)
        fmt.Printf("match has %d finished rounds\n", match_state.Statistic.FinishedRounds)

        // show the team statistics during the match
        team_list := selectors.SelectTeamList(match_state)
        for i, team := range team_list {
            fmt.Printf("  - team %2d (%15s) Name: %2s --> score %3d\n", i, team.TeamKey, team.Name, team.Statistic["score"])
        }
        // and the player scores
        player_list := selectors.SelectPlayerList(match_state)
        for i, player := range player_list {
            fmt.Printf("  - player %2d (PlayerKey: %2s, Name: %15s, UID: %5s) --> %3d assists |  %3d kills | %3d deaths\n",
            i, player.PlayerKey, player.Name, player.UID, player.Statistic["assist"], player.Statistic["death"], player.Statistic["kill"])
        }
    }

    // clean up
    match_sub.Cancel()

    // get the final statistics
    //var match_state *models.StatisticQueryState
    match_state, err := gameye.QueryStatistic(matchKey)
    if err != nil {
        fmt.Printf("\nSorry: QueryStatistic call failed:  %s\n", err)
        return
    }

    fmt.Printf("\n\nmatch has %d started rounds\n", match_state.Statistic.StartedRounds)
    fmt.Printf("match was started at %d\n", match_state.Statistic.Start)
    fmt.Printf("match was stopped at %d\n", match_state.Statistic.Stop)  // match_state.Statistic.Stop == 0 while it is running
    fmt.Printf("match has %d started rounds\n", match_state.Statistic.StartedRounds)
    fmt.Printf("match has %d finished rounds\n", match_state.Statistic.FinishedRounds)

    // use selectors to extract information about the match state
    player_list := selectors.SelectPlayerList(match_state)

    fmt.Printf("\nPayer stats for this match:\n")

    for i, player := range player_list {
        fmt.Printf("  - player %d (%s) --> (Name: %s, UID: %s)\n", i, player.PlayerKey, player.Name, player.UID)
        for stat_key, stat_value := range player.Statistic {
            fmt.Printf("     - player stat: %s --> %d\n", stat_key, stat_value)
        }
    }

    team_list := selectors.SelectTeamList(match_state)

    for i, team := range team_list {
        fmt.Printf("  - team %d (%s) --> (Name: %s, UID: %s)\n", i, team.TeamKey, team.Name)
        for stat_key, stat_value := range team.Statistic {
            fmt.Printf("     - team stat: %s --> %d\n", stat_key, stat_value)
        }
        for player_key, is_part_of_team := range team.Player {
            if is_part_of_team {
               fmt.Printf("     - team has player: %s\n", player_key)
            }
        }
    }

    // stop the match
    fmt.Printf("\nlet's STOP the match with key %s ....\n", matchKey)

    err = gameye.CommandStopMatch(matchKey);
    if err != nil {
        fmt.Printf("\nSorry: CommandStopMatch call failed:  %s\n", err)
        return
    }
    fmt.Printf("...done\n")
}
```
