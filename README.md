# MKS Fly System
A client and server **KrunkScript fly system** developed as part of the **[Miekie KrunkerScript Architecture Framework (MKS AF)](https://miekie-mks.vercel.app/)**.

The system is inspired by the flight mechanics of Minecraft's Elytra, including the use of double-tapping SPACE to toggle flight and double-tapping W, A, S, and D to increase flight speed.

The system provides player flight with server-side state management, client-side movement, network synchronization, double-tap flight activation, and movement boosting.

## Features
* Server-side flight state management using per-player flight data
* Client-side flight movement and input handling
* Double-tap Space to toggle flight
* Double-tap W A S D to boost flight speed
* Server-side flight and boost state synchronization
* Client/server communication through KrunkScript network messages
* Flight state initialization when a player spawns
* Flight state reset when flight is toggled
* Smooth acceleration and deceleration for flight movement
* Pitch-based vertical flight movement
* Space to move upward while flying
* Crouch to move downward while flying
* In-game flight instructions displayed through the client overlay
* Normal flight speed of `0.10`
* Boosted flight speed of `0.30`
* `400ms` double-tap detection window for flight and movement boost

## Controls
| Input                 | Action             |
| :-------------------- | :----------------- |
| `Double-tap SPACE`    | Toggle flight      |
| `W A S D`             | Move while flying  |
| `Double-tap W A S D`  | Boost flight speed |
| `SPACE` while flying  | Move upward        |
| `CROUCH` while flying | Move downward      |

## Known Issues
You might experience bugs when toggling flight on and off repeatedly or spamming the flight toggle. This is a known issue, and I am still finding a solution for it.
