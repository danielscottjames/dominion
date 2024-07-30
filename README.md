# Benchmarking LLMs as Casual Card Game AIs

I wanted to test if LLMs could be used as casual AIs for card/board games. The goal isn't to build the best or even a competitive AI. Given how easy it is to hook up an LLM AI to a game engine, this could be a low effort method of creating an AI that is interesting to play against.

Card games are an interesting application here IMO because they often have flexible rules based on the description of cards in play. This makes writing a procedural AI extremely difficult. However, LLMs obviously excel at understanding these descriptions. The question remains though if LLMs can form a coherent strategy based on the cards in play.

## Method

I found an [existing Dominion game engine](https://github.com/andrewrk/dominion) that outputs a log and all possible moves per turn. I feed the logs and turn prompt into the LLM as context in addition to some basic rules, strategies, and card descriptions.

I ask the LLM to output a JSON object that contains move # and an explanation. Some APIs were able to force JSON output, however, I found that all the LLMs I tested were very good at returning the correct JSON format anyways.

The context might look like:
```
... more logs preceded by kingdom card descriptions, rules, etc ...
Player 1 chooses: Buy Militia for 4 coins
Player 1 gains a Militia
Player 1 draws 5 cards
Player 2's Turn

Supply
10 Curse          8 Estate         8 Duchy          8 Province    
60 Copper        34 Silver        26 Gold        
10 Cellar        10 Chapel         8 Gardens        8 Militia       10 Throne Room 
 9 Council Room   8 Festival      10 Library        9 Mine          10 Witch       

Trash: Copper

Player 1 (3 victory points)
In Play: (empty)

Player 2 (3 victory points)
   In Deck: Copper Estate Festival Gold Silver
In Discard: 2xCopper 2xEstate 2xGold Silver
   In Play: (empty)
   In Hand: Militia, Copper, Copper, Copper, Copper

Waiting for you (Player 2) to play an action
Actions: 1  Buys: 1  Coins: 0
Possible moves:
 1 Play Militia
 2 Done playing actions
```
and the LLM will respond with
```
{ "explanation": "Playing Militia will not only give me 2 coins but will also force Player 1 to discard down to 3 cards, potentially disrupting their next turn.", "move": 1 }
```

Logs for every game played are kept in the [logs](./logs/) directory.

## Results

Small models are disappointing. Unfortunately, any model small enough to work on consumer hardware would not feel like a competent opponent.

Flagship models play at a level that I would describe as potentially interesting to beginner-level players. However, most players would quickly learn to out-play the LLMs.

## Benchmark Rankings

I made the AIs play many games against each other. I used an ELO system to rank the players.

If you aren't familiar with ELO the gist is that every 400 points on the scale represents 10x better odds of winning. For example, if two models have ranks 200 and 600, the former model is estimated to win 1/10 times. If the two models have ranks 200 and 1000, the former model is estimated to win 1/100 times.

(ELO rankings are not comparable across benchmarks. My rankings are normalized on the `random` AI having an ELO of 0.)

| Rank | Name                          | Elo  |
|------|-------------------------------|------|
| 1    | gpt-4o                        | 1377 |
| 1    | claude-3-5-sonnet             | 1377 |
| 2    | llama3.1-405B-instruct-turbo  | 1223 |
| 3    | naive                         | 1143 |
| 4    | llama3.1-70B-instruct-turbo   | 1132 |
| 5    | gpt-4o-mini                   | 420  |
| 6    | bigmoney                      | 340  |
| 7    | llama3.1-8B-instruct-turbo    | 332  |
| 8    | random                        | 0    |

The rankings show that small models and large models are on a different playing field. `gpt-4o` and `claude-sonnet` are neck and neck. (Many more rounds might reveal that one has an edge over the other but I don't care enough to spend the $$.) The flagship models have a slight edge over a `naive` procedural algorithm. Private models perform slightly better at this task than their open source counterparts.

## Remarks and Future Explorations
* Each inference in my experiment is limited to <4k tokens, but many models can accommodate many more tokens. It would be interesting to explore testing if providing more context (in the form of more detailed logs, strategy articles from the internet, etc...) results in measurably improved performance.
* Comparing these models with current state of the art Dominion AIs would not be interesting. The LLMs would be crushed.
* The 8B and 70B Llama models aren't even in the same league. However, the 405B model only beats the 70B model in 2/3rds of games. The private flagship models perform slightly better than the 405B model. These results might imply substantial diminishing returns on reasoning capabilities.
* If much more capable models can exist, this benchmark has plenty of room for growth. I also believe it would be hard for models to game this benchmark. As new models are released, I may come back and update this page with the results.
* It could be interesting to compare how quantization affects these scores. (ex compare q2/3 405B model vs full weight 70B model) 

## Building
This is essentially two packages, plus some helper scripts.
* Run `npm install` in the base directory and `llmai`. 
* Create a file `llmai/src/config.json` and fill in your `*_API_KEY`s
* Run `npm build` in the `llmai` directory.


[compete.js](compete.js) will continually run matches between AIs<br>
[stats.js](stats.js) will normalize the elo rankings based the data stored in the logs directory.

## Command Line Usage
```
Usage: dominion-cli [--player <AI_Name>] [--seed <seed>]
AIs available:
  cli
  lmstudioai -- (Play against a model running locally)
  naive
  bigmoney
  random
  claude-3-5-sonnet
  gpt-4 -- (Pretty expensive to run)
  gpt-4o
  gpt-4o-mini
  llama3.1-405B-instruct-turbo
  llama3.1-70B-instruct-turbo
  llama3.1-8B-instruct-turbo
  
Sets available:
  Base Set (second edition)

(Note, requests to LLM apis are not deterministic and therefore the RNG seed will not reproduce between games with LLM opponents.)
```
