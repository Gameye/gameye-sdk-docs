# Gameye SDK Documentation

## Introduction

Create eSport and competitive matches for for your platform 
without fixed monthly costs or any need for your own server infrastructure for games like:
- Counter-Strike: Global Offensive, 
- Team Fortress 2, Left 4 Dead 2, Killing Floor 2, Insurgency and Day of Infamy 

Simply implement the Gameye API to kick off online matches when you need the
you will even be able to implement the scores/statistics directly on your website. 

How cool is that!

# Available SDK options
The Gameye SDK's are the recommended way to use our API.

Our SDK's are available in several popular languages:
1. Node.js (Typescript)
1. Golang
1. PHP
1. Raw REST API (curl)

In these pages we well show you how easy it is to use the SDK's and the raw REST API.

## Getting started

### Get an API KEY

Obtain a free Gameye API key, please send us an [email](mailto:support@gameye.com).

All API request need to be authorised using your `API_KEY` as an (OAuth) `Bearer` token.

### Install the SDK
Follow the instructions for your language of choice bellow. In case of any trouble contact our support department for assistance.

#### Install the SDK (Node.js using npm)

```bash
$ npm install @gameye/sdk -s
```

#### Install the SDK (Go)

```bash
go get -u github.com/Gameye/gameye-sdk-go/clients
```


#### Install the SDK (PHP using composer)

```php
$ composer require gameye/gameye-sdk-php
{
    "require": {
        "gameye/gameye-sdk-php": "2.*"
    }
}
```

# Create a Gameye client

The SDK's provide a Gameye client class, you will need to initiate it with your `GAMEYE_API_KEY` 
and the API `endoint` you want to use. 

In case of raw API use, use one of the available `endpoints` and add the `GAMEYE_API_KEY` in the
authorization header.

## Initiate a Gameye Client (Node.js)

```typescript
import {
    GameQueryState,
    GameyeClient,
    GameyeClientConfig,
    MatchQueryState,
    QuerySubscription,
    StatisticQueryState,
    TemplateQueryState
} from "@gameye/sdk"

const api_config = <GameyeClientConfig>({endpoint:"https://api.gameye.com", token: "GAMEYE_API_KEY"});

const gameye = new GameyeClient(api_config);
```

## Initiate a Gameye Client (Go)

Make sure to import the needed package(s):

```go

import (
	"github.com/Gameye/gameye-sdk-go/clients"
)
```
Now you can instantiate `GameyeClientConfig` with your `Endpoint`  and api `Token`
and use it to set create a `NewGameyeClient`

```go
api_config := clients.GameyeClientConfig{Endpoint: "https://api.gameye.com", Token: "GAMEYE_API_KEY"}

gameye := clients.NewGameyeClient(api_config)
```


## Initiate a Gameye Client (PHP)

```PHP
$gameye = new \Gameye\SDK\GameyeClient([
            'AccessToken' => 'GAMEYE_API_KEY',
            'ApiEndpoint' => 'https://api.gameye.com',
        ]);
```

# Command and Query Responsibility Segregation (CQRS)

The Gameye SDK's (and API) are desigend using the
[Command and Query Responsibility Segregation](http://codebetter.com/gregyoung/2010/02/16/cqrs-task-based-uis-event-sourcing-agh/)
design pattern.

We have disticts set or `queries` you can use the GET information from the API, 
and we have an other set of `commands` you can use to change things. You can use the 
`query` interface to get information about possible games and options and about statistics of
a match in progress. However you will need to use the `command` interface for things such as to
`start` and `stop` a match etc.


## Get information about available games and locations

A wide range of games and game opions are available, 
you can `GET` an live list using the `game` `Query`.

The result will be a structure with a list of `game`s, a list of `location`s and 
for each `game` a list of locations where the game is available.

###  Query Game Sample Data (json)

The game and location data are represented as a simple json structure like this:

```json
{
    "game": {
        "csgo": {
            "gameKey": "csgo",
            "location": {
                "rotterdam": true,
                "los_angeles": true,
                "washington_dc": true,
                "frankfurt": true
            }
        },
        "tf2": {
            "gameKey": "tf2",
            "location": {
                "rotterdam": true
            }
        },
        "css": {
            "gameKey": "css",
            "location": {}
        }
    },
    "location": {
        "rotterdam": {
            "locationKey": "rotterdam"
        },
        "dubai": {
            "locationKey": "dubai"
        },
        "tokyo": {
            "locationKey": "tokyo"
        },
        "los_angeles": {
            "locationKey": "los_angeles"
        }
    }
}
```

###  Query Game (raw API)

Note that the basic information about games and locations is public and you can fetch this data
without the need for an valid `GAMEYE_API_KEY`

```bash
curl https://api.gameye.com/fetch/game
```


###  Query Game (Node.js)

In `node.js` (typescipt) the `query` interface returns a `promise`. 

You can use the clasic `.then().catch()` syntax like this:

```typescript
gameye.queryGame().then(games_and_locations => {
    // extract game and location data
    console.debug("Games and locations available:");
    for(const game in games_and_locations.game){
        console.log("game : ", game, " available at locations:", games_and_locations.game[game].location);
    }
    for(const location in games_and_locations.location){
        console.log("location : ", location,  games_and_locations.location[location]);
    }
}).catch(error=>{
    console.error("Sorry: queryGame call failed: ", error);
});

```

However we prever to use the `async`, `await` [syntax](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-1-7.html).
like this:

```typescript
async function get_games_and_locations(gameye: GameyeClient) {

    console.log("\nGet available games and locations ...");
    let games_and_locations: GameQueryState;
    try {
        games_and_locations = await gameye.queryGame();
    } catch (error) {
        console.error("Sorry: queryGame call failed: ", error);
        return -1;
    }
    console.log("... done");

    for (const game in games_and_locations.game) {
        console.log("game : ", game, " available at locations:", games_and_locations.game[game].location);
    }

    for (const location in games_and_locations.location) {
        console.log("location : ", location, "  -->", games_and_locations.location[location]);
    }
}
```

### Location selector

The Gameye SDK provides you with predefined selector functions that can make it easier
to traverse the data structure used in de API. Your code will be more robust against future
changes in the API's datastructures, so we encourage you to use them.

The snippet above can be rewritten using the `selectLocationListForGame` that returns a list of
`LocationItem`s: 

```typescript
// import location related selector

import { LocationItem, selectLocationListForGame } from "@gameye/sdk";

async function get_games_and_locations(gameye: GameyeClient) {

    console.log("\nGet available games and locations ...");
    let games_and_locations: GameQueryState;
    try {
        games_and_locations = await gameye.queryGame();
    } catch (error) {
        console.error("Sorry: queryGame call failed: ", error);
        return -1;
    }
    console.log("... done");
    
    for (const game in games_and_locations.game) {
        const available_locations: LocationItem[] = selectLocationListForGame(games_and_locations, game);
        for (const locItem of available_locations) {
            console.log("game : ", game, " available at location:", locItem.locationKey );
        }
    }
}
```

###  Query Game (Go)

In the following (complete) sample we show how to use the `QueryGame` method 
in Go to retrieve all `Location`s and all `Game`s

A `Query` in the `Go` SDK returns a (`QueryState`, `Error`) tuple.

You can run this part of the SDK without a valid `API TOKEN`.

```go
package main

import (
	"fmt"
	"github.com/Gameye/gameye-sdk-go/clients"
	"github.com/Gameye/gameye-sdk-go/models"
	"github.com/Gameye/gameye-sdk-go/selectors"
)

func main() {
    // init the client
    api_config := clients.GameyeClientConfig{Endpoint: "https://api.gameye.com"} // no token needed
    gameye := clients.NewGameyeClient(api_config)

    fmt.Printf("fetch game info from Gameye client\n")

    var err error
    var games_and_locations *models.GameQueryState

    games_and_locations, err = gameye.QueryGame()
    if err != nil {
        fmt.Printf("\nSorry: queryGame call failed:  %s\n", err)
        return
    }

    fmt.Printf("\nWe have %d locations\n", len(games_and_locations.Location))

    for i, loc_item := range games_and_locations.Location {
        fmt.Printf("  - location %s --> %#v\n", i, loc_item)
    }

    fmt.Printf("\nWe have %d games\n", len(games_and_locations.Game))

    for game_key, game_item := range games_and_locations.Game {
        fmt.Printf("  - game %s available at locations: %#v\n", game_key, game_item.Location)
    }

    // using a selector
    fmt.Printf("\nThese game are available to start\n")

    for game_key, _ := range games_and_locations.Game {
        locationList := selectors.SelectLocationListForGame(games_and_locations, game_key)
        for _, loc_item := range locationList {
           fmt.Printf("  - game %s available at location: %s\n", game_key, loc_item.LocationKey)
        }
    }

}
```


###  Query Game (PHP)

TODO

## Query Game templates and parameters

For each game a variety of templates exists to start a match. The API allows you to get 
an up to date list of known templates and the allowed values.

 
### Template Sample Data (json)

Fetching the list of templates for a valid `gameKey` results in a json structure where
all valid templates are listed as keys in the `template` attribute.

Each `template` has a `templateKey` and an `arg` attribute.

The `arg` attribute list all the valid arguments than can be passed when staring a match.

Arguments have:
 - a `name`, 
 - a `type` (`string` or `number`) 
 - a `defaultValue`
 - a `minimumValue` and `maximumValue` in case of numeric values
 - an `option` array of allowed values of the correct type
 - a `validatePattern` (string) and a `validateIgnoreCase` (boolean) for `string` 
   arguments that need to be validated against a regex.
 
A sample looks like this:
 
```json
{
	"template": {
		"2v2wingman": {
			"templateKey": "2v2wingman",
			"arg": []
		},
		"bots": {
			"templateKey": "bots",
			"arg": [{
				"name": "tickRate",
				"type": "number",
				"defaultValue": 128,
				"option": [64, 128]
			}, {
				"name": "steamToken",
				"type": "string",
				"defaultValue": ""
			}, {
				"name": "hostname",
				"type": "string",
				"defaultValue": "gameye.com Match Server"
			}, {
				"name": "mapgroup",
				"type": "string",
				"defaultValue": "mg_active",
				"validatePattern": "^\\w+$",
				"validateIgnoreCase": true
			}, {
				"name": "map",
				"type": "string",
				"defaultValue": "de_dust2",
				"validatePattern": "^\\w+$",
				"validateIgnoreCase": true
			}, {
				"name": "gameMode",
				"type": "number",
				"defaultValue": 1,
				"option": [0, 1, 2]
			}, {
				"name": "gameType",
				"type": "number",
				"defaultValue": 0,
				"option": [0, 1]
			}, {
				"name": "maxRounds",
				"type": "number",
				"defaultValue": 30,
				"minimumValue": 1,
				"maximumValue": 30
			}]
		},
		"esl1on1": {
			"templateKey": "esl1on1",
			"arg": []
		}
	}
}
```


### Query Template (raw API)

```bash
curl \
--header "Content-Type: application/json" \
--header "Authorization: Bearer GAMEYE_API_KEY" \
https://api.gameye.com/fetch/template?gameKey=VALID_GAME_KEY
```

### Query Template (Node.js)

```typescript
async function get_templates_for_game(gameye: GameyeClient, gameKey: string) {

    let available_templates: TemplateQueryState;

    console.log("\nGet avaiable templates for game", gameKey, " ...");
    try {
        available_templates = await gameye.queryTemplate(gameKey);
    } catch (error) {
        console.error("Sorry: queryTemplate call failed: ", error);
        return -1;   
    }
    console.log("...done");
    
    console.log("\nknown templates for game: ", gameKey);

    for (const template in available_templates.template) {
        console.log("known template for game: ", gameKey,  " --> ", template, " --> ",  available_templates.template[template]);
    }
}
```

###  Template selectors

The gameye SDK provides two convient selectors for templates too, we encourage you to use them.

The snippet above can be rewritten using the `selectTemplateList` and `selectTemplateItem`  
selectors like this:


```typescript
// import the template selectors and related types
import { TemplateItem, selectTemplateItem, selectTemplateList } from "@gameye/sdk"

async function get_templates_for_game(gameye: GameyeClient, gameKey: string) {

    let available_templates: TemplateQueryState;

    console.log("\nGet avaiable templates for game", gameKey, " ...");
    try {
        available_templates = await gameye.queryTemplate(gameKey);
    } catch (error) {
        console.error("Sorry: queryTemplate call failed: ", error);
        return -1;
    }
    console.log("...done");

    console.log("\nknown templates for game: ", gameKey);

    const template_list: TemplateItem[] = selectTemplateList(available_templates);
    for (const template of template_list) {
        console.log("  --> ", template.templateKey);
    }
    
    // if the gameKey is known use the selectTemplateItem
    
    const templateKey: string = 'bots';
    const selected_template: TemplateItem = selectTemplateItem(available_templates, templateKey);
    if(!selected_template){
        // selected_template will ne null in case it is not found
        console.error("Sorry, template", templateKey, "is not available for game:", gameKey," aborting...");
        return -1;
    }

    // now it is easy to list all arguments of the template
    
    console.log("\nallowed arguments for ", gameKey, " --> ", templateKey);
    for(const arg of selected_template.arg){
        console.log(" --> ", arg );
    }
}
```

In the following sample we demonstrate how to extract a list of game `Template`s
and use selectors to access the `Arg`s for a given `template`.

### Query Template (Go)

```Go
package main

import (
    "fmt"
    "github.com/Gameye/gameye-sdk-go/clients"
    "github.com/Gameye/gameye-sdk-go/models"
    "github.com/Gameye/gameye-sdk-go/selectors"
)

func main() {

    api_config := clients.GameyeClientConfig{Endpoint: "https://api.gameye.com", Token: "GAMEYE_API_KEY"}
    gameye := clients.NewGameyeClient(api_config)

    gameKey := "csgo"
    fmt.Printf("fetch %s game templates from Gameye client\n", gameKey)

    var err error
    var available_templates *models.TemplateQueryState

    available_templates, err = gameye.QueryTemplate(gameKey)
    if err != nil {
        fmt.Printf("\nSorry: queryTemplate call failed:  %s\n", err)
        return
    }

    fmt.Printf("\nWe have %d templates available for game '%s'\n", len(available_templates.Template), gameKey)

    for templ_key, templ_item := range available_templates.Template {
        fmt.Printf("  - template '%s' with %d args\n", templ_key, len(templ_item.Arg))
    }

    // use a selector if you like a list better

    templateList := selectors.SelectTemplateList(available_templates)

    fmt.Printf("\nWe have %d templates available for game '%s'\n", len(templateList), gameKey)

    for i, templ_item := range templateList {
        fmt.Printf("  - template %d --> '%s' with %d args\n", i, templ_item.TemplateKey, len(templ_item.Arg))
    }

    // pick a specific template and show the args

    templateKey := "bots"
    template := selectors.SelectTemplateItem(available_templates, templateKey)

    fmt.Printf("\nThese %d args are available for template '%s' and game '%s'\n", len(template.Arg), templateKey, gameKey)

    for i, arg_item := range template.Arg {
        fmt.Printf("  - arg %d  -->  %#v\n", i, arg_item)
    }
}
```

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
--header "Authorization: Bearer GAMEYE_API_KEY" \
--data '{
    "locationKeys": ["rotterdam"],
    "gameKey": "csgo",
    "templateKey": "bots",
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
--header "Authorization: Bearer GAMEYE_API_KEY" \
--data '{
    "matchKey": "YOUR_UNIQUE_MATCH_KEY"
}' \
https://api.gameye.com/action/stop-match
```

The result wil be a HTTP status code:
- a `204` 'NO CONTENT' in case of a successful started match
- a `200` 'SUCCESS' in case of a successful stopped match
- a `401` 'AUTHORISATION NEEDED' in case of a missing `GAMEYE_API_KEY` 
- a `403` 'NOT ALLOWED' in case of an invalid `GAMEYE_API_KEY` 
- a `404` 'NOT FOUND' in case of a non existing game (template)
- a `500` in case of any mistakes in the posted payload
  

### Start and stop a match (Node.js)

```typescript
async function start_stop_match(gameye: GameyeClient){
    const gameKey: string = "csgo";
    const locationKeys: string[] = ["rotterdam"];
    const templateKey: string = "bots";
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

    api_config := clients.GameyeClientConfig{Endpoint: "https://api.gameye.com", Token: "GAMEYE_API_KEY"}
    gameye := clients.NewGameyeClient(api_config)

    // all params we need to start a match
    gameKey := "csgo"
    locationKeys := []string{"rotterdam"}
    templateKey := "bots"
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

After you stared a match you can listen to updates from the live match using a view simple `query` calls.

If you have any matches in progress you can fetch the state using the  `match` `query`.

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
        "locationKey": "rotterdam",
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
--header "Authorization: Bearer GAMEYE_API_KEY" \
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



### Query match state  (Go)
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

    api_config := clients.GameyeClientConfig{Endpoint: "https://api.gameye.com", Token: "GAMEYE_API_KEY"}
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


### statistic data structure (json)

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
--header "Authorization: Bearer GAMEYE_API_KEY" \
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

```typescript

async function observe_match(gameye: GameyeClient, matchKey: string){

    // observe the match in real time

    let match_observer: any;
    let match: false | StatisticQueryState;
    let match_is_running = true;
    
    try {
        match_observer = await gameye.subscribeStatistic(matchKey);
    } catch (error) {
        console.warn("Problem with subscribeStatistic (aborting) :", error);
        match_is_running = false; // match my be stopped ?
    }
    
    while(match_is_running){
        try{
            // wait for new (changed) state
            match = await match_observer.nextState(); // will return a false of the observer is destroyed
            if(!match) break;
            
            match_is_running = !match.statistic.stop;
                        
            console.log("  - start = ", match.statistic.start);
            console.log("  - stop = ", match.statistic.stop);
            
            console.log("  - startedRounds = ", match.statistic.startedRounds);
            console.log("  - finishedRounds = ", match.statistic.finishedRounds);
            
            for( const team in match.statistic.team ){
                console.log("     - team ", team, " --> ", match.statistic.team[team]);
            }
            for( const player in match.statistic.player ){
                console.log("     - player ", player, " --> ", match.statistic.player[player]);
            }
        } catch (error) {
            console.warn("Problem with queryStatistic (aborting) :", error);
            // match my be stopped ?
            match_is_running = false;
        }
    }

    // clean up after observing is done
    match_observer.destroy();

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

### Team and player selectors

we can rewrite this using the provided selectors:
- use `selectTeamList` to get a list of `TeamItem`s
- use `selectTeam` to get a single `TeamItem` by `teamKey`
- use `selectPlayerList` to get a list of `PlayerItem`s
- use `selectPlayerItem` to get a single `PlayerItem` by `playerKey`
- use `selectPlayerListForTeam` to get a list of `PlayerItem`s for given `teamKey`

```typescript

// import selectors and types for players and teams
import { selectPlayerItem, selectPlayerItem, selectPlayerList, selectPlayerListForTeam } from "@gameye/sdk";
import { TeamItem, selectTeamItem, selectTeamList } from "@gameye/sdk";

async function observe_match(gameye: GameyeClient, matchKey: string){

    // observe the match in real time

    let match_observer: any;
    let match: false | StatisticQueryState;
    let match_is_running = true;
    let match_state: StatisticQueryState;

    try {
        match_observer = await gameye.subscribeStatistic(matchKey);
    } catch (error) {
        console.warn("Problem with subscribeStatistic (aborting) :", error);
        match_is_running = false; // match my be stopped ?
    }
    
    while(match_is_running){
        try{
            // wait for new (changed) state
            match = await match_observer.nextState(); // will return a false of the observer is destroyed
            if(!match) break;
            
            match_state = <StatisticQueryState>match_state_or_false;

            match_is_running = !match.statistic.stop;
            
            // extract player stats
            let player: PlayerItem;
            const player_list: PlayerItem[] = selectPlayerList(match_state);

            for( player of player_list){
                console.log("     - player ", player.playerKey , " --> ", player);
            }

            // extract team stats
            let team: TeamItem;
            const team_list: TeamItem[] = selectTeamList(match_state);

            for( team of team_list ){
                console.log("     - team ", team.teamKey , " --> ", team);
            }

        } catch (error) {
            console.warn("Problem with queryStatistic (aborting) :", error);
            // match my be stopped ?
            match_is_running = false;
        }
    }

    // clean up after observing is done
    match_observer.destroy();
}
```

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

    api_config := clients.GameyeClientConfig{Endpoint: "https://api.gameye.com", Token: "GAMEYE_API_KEY"}
    gameye := clients.NewGameyeClient(api_config)

    // all params we need to start a match
    gameKey := "csgo"
    locationKeys := []string{"rotterdam"}
    templateKey := "bots"
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