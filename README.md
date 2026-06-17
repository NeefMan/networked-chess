Networked Chess (networked-chess)

A two-player online chess game built in Python with Pygame and raw TCP sockets.

Networked Chess lets two players connect through a relay server, pair by username, and play on a graphical chess board in real time. The project implements its own TCP client-server protocol with JSON messages rather than relying on a game-networking framework.

The server coordinates player connections, assigns colors, and relays moves. Each client renders the board, calculates available moves, and updates its local game state.

Table of Contents

* Security
* Background
* Install
* Usage
* Architecture
* Network Protocol
* Project Structure
* Current Limitations
* Maintainer
* Contributing
* License

Security

This project is an educational prototype and should not be treated as a production-ready game server.

* Traffic is not encrypted.
* Usernames and control messages are not authenticated.
* The server trusts moves submitted by clients.
* kill.py sends an unauthenticated shutdown request.
* The server stores connection and game state only in memory.

Do not expose the server to an untrusted network without first adding authentication, authorization, input validation, rate limiting, and encrypted transport. Never commit private keys, passwords, tokens, or other credentials to this repository.

Background

This project was created to explore how a real-time multiplayer game can be built from the ground up. It combines chess movement logic, a Pygame interface, TCP sockets, threads, JSON serialization, and a relay-server architecture.

The main abstract dependencies are:

* A graphical client that renders the board and handles player input.
* A TCP client that exchanges framed JSON messages with the server.
* A relay server that pairs users, assigns turns, and forwards moves.
* A shared understanding of board coordinates and message formats.

All chess movement and networking logic in this repository is implemented directly rather than through a chess engine or game-networking library.

Install

Dependencies

* Python 3
* Pygame
* Git, if cloning the repository from GitHub

Clone the repository and enter its directory:

git clone https://github.com/NeefMan/networked-chess.git
cd networked-chess

Create and activate a virtual environment on macOS or Linux:

python3 -m venv .venv
source .venv/bin/activate
python3 -m pip install pygame

On Windows PowerShell:

py -m venv .venv
.venv\Scripts\Activate.ps1
py -m pip install pygame

Usage

Configure the server address

The server listens on every network interface at TCP port 5000. For local testing, set the host in client.py to the loopback address:

self.HOST = "127.0.0.1"
self.PORT = 5000

For internet play, replace 127.0.0.1 with the server’s public IP address or hostname. Allow inbound TCP traffic on port 5000 only from networks or addresses you trust.

Start the server

Run this command from the repository root:

python3 server.py

Windows users may use:

py server.py

Start two clients

Each player runs the graphical client from a separate computer or terminal:

python3 main.py

Windows users may use:

py main.py

Both players should enter unique usernames and enter each other’s usernames when prompted. The server waits for the requested opponent, pairs the players, and randomly assigns the first turn.

CLI

Player pairing occurs through terminal prompts before the Pygame window starts:

What is your username? alice
Who would you like to connect to? bob
Connected to server
Successfully connected to user: bob

Controls

Input	Action
Left-click a piece	Select the piece
Left-click a highlighted square	Move the selected piece
F	Flip the board
R	Reset the board for both players
Q	Disconnect and quit

Architecture

Each player runs main.py, which creates the Pygame interface and initializes a Client instance from client.py.

The client:

1. Opens a TCP connection to the relay server.
2. Registers a username and requested opponent.
3. Receives its assigned turn and color.
4. Sends local moves to the server.
5. Polls the server for the opponent’s moves.
6. Applies received moves to the local board.

The server in server.py accepts multiple socket connections and handles each connection in a daemon thread. It stores usernames, sockets, pending moves, turn assignments, and connection timestamps in memory.

The server currently acts as a relay rather than an authoritative chess engine. Legal move calculations happen on each client.

Network Protocol

Messages are serialized as JSON and terminated with the custom delimiter:

*&^%

A move request has the following form:

{
  "username": "alice",
  "task": "move",
  "to_user": "bob",
  "move": [[4, 6], [4, 4]]
}

Other task values used by the current protocol include:

* connect
* check_move
* reset
* dc
* kill

The protocol is internal and may change while the project is in development.

Project Structure

networked-chess/
├── README.md
├── main.py
├── client.py
├── server.py
├── kill.py
├── classic_piece_map.py
└── images/
    └── pieces/
        ├── white_pawn.png
        ├── black_pawn.png
        ├── white_bishop.png
        ├── black_bishop.png
        ├── white_knight.png
        ├── black_knight.png
        ├── white_rook.png
        ├── black_rook.png
        ├── white_queen.png
        ├── black_queen.png
        ├── white_king.png
        └── black_king.png

* main.py contains the game loop, board rendering, piece classes, movement logic, controls, and client integration.
* client.py handles server configuration, pairing, sending messages, receiving moves, and disconnecting.
* server.py accepts players, assigns turns, relays moves, and forwards reset requests.
* kill.py sends the server’s current shutdown task and is intended only for controlled development environments.
* classic_piece_map.py defines the initial board layout.
* images/pieces/ contains the chess-piece sprites.

Current Limitations

* The server address is hardcoded in client.py.
* The protocol has no authentication or encryption.
* The server does not independently validate moves.
* Check and checkmate are not implemented.
* Castling is not implemented.
* En passant is not implemented.
* Pawn promotion is not implemented.
* There is no automatic matchmaking.
* There is no reconnect or game-recovery system.
* There is no move history, timer, or captured-piece display.
* Server state is lost when the process stops.
* The game is not packaged as a standalone executable.

Maintainer

* Ethan Lahr (@NeefMan)

Contributing

Questions may be asked by opening an issue in the NeefMan/networked-chess GitHub repository.

Pull requests are welcome. Before contributing:

1. Open an issue before making a large behavioral or protocol change.
2. Keep changes focused and explain the reason for them.
3. Test changes with one server and two connected clients.
4. Keep Python examples and source code consistent with the project’s existing style.
5. Do not commit virtual environments, __pycache__ directories, credentials, or private server information.
6. Update this README when installation, controls, protocol messages, or behavior changes.

No contributor sign-off is currently required.

License

UNLICENSED — Copyright (c) Ethan Lahr.

No license has been granted to copy, modify, or distribute this project.
