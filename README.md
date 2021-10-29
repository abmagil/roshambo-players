# Player Registrations

## Structures
```
Tournament - A structure of Games with rules defining how the Players compete against each other and what outcomes occur after a Player wins a game. Tournaments must have a winner.
  │
  └ Game - A series of Rounds which can have a Winner/Loser or a tie. A Game has a winner after a set number of Rounds which is whoever has won the most Rounds. If neither player has won the most rounds, a random player will be selected as the Winner of the Game.
    │
    └ Round - A known number of Hands each scored towards the winning score of a Round. A Round can end in a tie if neither player has won more Rounds than the other..
        │
        └ Hand - a pair of Moves associated with the Player who played each. Results in a winner/loser or a tie
            │
            └ Move - a single choice of Rock, Paper, or Scissors
```

## Players

A Player can either be "Local" or "Network". Local Players are effectively built-in bots using precomputed strategies. Network Players are written by others and can have arbitrarily complex strategies. In any given game, there are no guarantees about whether your opponent is a Local or Network Player.

The interface for all Players is simple- their only responsibility is selecting a Move ("R", "P", "S").

### Network Player Patterns

```
┌──────────┐                ┌────────────────┐         ┌──────────────────┐
│Tournament│                │registration svc│         │Network Player    │
└───┬──────┘                └───────┬────────┘         └─────────┬────────┘
    │                               │                            │
    │   GET RegistrationRequest     │                            │
    ├──────────────────────────────►│                            │
    │                               │                            │
    │◄──────────────────────────────┤                            │
    │       RegisterResponse        │                            │
    │                               │                            │
    │                               │                            │
    │                               │                            │
    │                               │   GET MoveRequest          │
    ├───────────────────────────────┼───────────────────────────►│
    │                               │                            │
    │                               │       MoveResponse         │
    │◄──────────────────────────────┼────────────────────────────┤
    │                               │                            │
    │                               │                            │
    │                               │                            │
    │                               │  POST AckRequest           │
    ├───────────────────────────────┼───────────────────────────►│
    │                               │                            │
    │                               │  <any>                     │
    │◄──────────────────────────────┼────────────────────────────┤
    │                               │                            │
    │                               │                            │
```

To register a Network Player, a Registration URL must be filed with the game (MR into this repo). On Tournament boot, a request will be sent to this URL. The Request will be of type `RegistrationRequest` and the response should be of type `RegistrationResponse`. Once all Registration URLs are resolved to Network Players, the tournament will be filled with Local Players to complete the required number of competitors and play will begin.

Play proceeds in two phases- the Tournament will send a `MoveRequest` to the move URL for each Network Player. The Network Player will respond with a `MoveResponse` and the game will score the Hand then send an `AckRequest` to the ack URL for that Network Player. If a Network Player cannot respond within 30s, they automatically forfeit the Hand. This process will be repeated until a Game is won, which will be the final `AckRequest` of a Game. For tournaments of multiple rounds, a new game will begin immediately against a new opponent (for certain tournament types, it is possible to be the same opponent but the tournament considers this a new game).

The only indications that a new game has begun will be a change in the gameId in a MoveRequest.
