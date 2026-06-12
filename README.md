# Online Chess

A two-player online chess game built in Python with Pygame and raw TCP sockets.

This project lets two players connect to a server, enter usernames, pair with each other, and play chess in real time. The game uses a custom client-server networking system instead of a game networking library.

## Features

- Two-player online chess
- Pygame graphical board
- Raw TCP socket networking
- JSON-based client/server messages
- Username-based player connection system
- Randomized first turn
- Board flipping for black/white perspective
- Legal move highlighting
- Piece movement logic for pawns, bishops, knights, rooks, queens, and kings
- Board reset support
- Quit/disconnect support

## Tech Stack

- Python
- Pygame
- Python `socket` module
- Python `threading` module
- JSON message formatting

## Project Structure

```text
chess/
├── main.py
├── client.py
├── server.py
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
```

## File Overview

### `main.py`

This is the main game file. It starts the client, creates the Pygame window, builds the chess board, handles player input, and runs the main game loop.

The `Game` class initializes the network client, starts the client listener thread, creates the board, and runs the Pygame event loop.

The `Board` class handles:

- Drawing the board
- Drawing pieces
- Detecting clicked squares
- Flipping the board
- Selecting pieces
- Computing valid moves
- Moving pieces
- Sending player moves to the server

The piece classes are also stored in this file:

- `Piece`
- `Pawn`
- `Bishop`
- `Knight`
- `Rook`
- `Queen`
- `King`

Each piece has a `move_set` that defines how it is allowed to move.

### `client.py`

This file handles the client-side networking.

The client connects to the server, asks the user for their username, asks who they want to connect to, and sends a connection request to the server.

The client sends and receives JSON data over a TCP socket. Each message ends with a custom delimiter so the receiver can tell where one complete message ends.

The client also runs a background loop that checks for moves from the other player. When a move is received, it updates the local board.

### `server.py`

This file runs the relay server.

The server listens on port `5000`, accepts client connections, stores user data, and forwards moves between players.

The server keeps track of:

- Each username
- The user's socket connection
- The user's opponent
- Pending moves
- Turn assignment
- Connection timestamps

When two users connect to each other, the server randomly chooses who goes first. When one player makes a move, the server stores that move for the other player to receive.

### `classic_piece_map.py`

This file stores the starting board layout.

It maps board coordinates to piece abbreviations. For example:

- `wp` = white pawn
- `bp` = black pawn
- `wr` = white rook
- `br` = black rook
- `wq` = white queen
- `bq` = black queen
- `wki` = white king
- `bki` = black king

## How the Game Works

The game uses a client-server model.

Each player runs `main.py`. When the game starts, the client asks for:

1. Your username
2. The username of the player you want to connect to

The client then connects to the server and sends a connection request.

The server waits until both users are connected. Once both players are available, it randomly assigns the first turn. One player starts as white and the other starts as black.

When a player makes a move:

1. The player clicks a piece.
2. The game calculates valid moves for that piece.
3. The player clicks a destination square.
4. The client sends the move to the server.
5. The server stores the move for the opponent.
6. The opponent's client checks the server for a move.
7. The opponent's board updates.

## Controls

| Key / Action | What it does |
|---|---|
| Left click a piece | Selects a piece |
| Left click a valid square | Moves the selected piece |
| `F` | Flips the board |
| `R` | Resets the board for both players |
| `Q` | Disconnects and quits |

## Setup

Install Python and Pygame.

```bash
pip install pygame
```

Clone the repository:

```bash
git clone https://github.com/NeefMan/chess.git
cd chess
```

## Running the Server

Start the server first:

```bash
python server.py
```

The server listens on:

```text
0.0.0.0:5000
```

If the server is hosted on AWS EC2 or another cloud server, make sure TCP port `5000` is open in the firewall/security group.

## Running the Client

Start the game client:

```bash
python main.py
```

The client currently connects to the server IP address set in `client.py`.

```python
self.HOST = "18.218.245.80"
self.PORT = 5000
```

To use a different server, change `HOST` to your server's public IP address.

## Networking Format

The game sends JSON data over TCP sockets.

Each message is converted to JSON and then has this delimiter added to the end:

```text
*&^%
```

This lets the receiver know when a full message has arrived.

Example move request:

```json
{
  "username": "player1",
  "task": "move",
  "to_user": "player2",
  "move": [[0, 6], [0, 4]]
}
```

## Current Limitations

This project is still in development. Some chess rules and networking features may need more work.

Current limitations include:

- Server IP is hardcoded in `client.py`
- No menu screen for entering server settings
- No automatic matchmaking
- No reconnect system
- No check or checkmate detection
- No castling
- No en passant
- No pawn promotion
- No move history
- No packaged executable
- The server stores user state in memory only

## Possible Future Improvements

- Add a start menu
- Let the user type the server IP in the game window
- Add check and checkmate detection
- Add castling
- Add pawn promotion
- Add en passant
- Add move history
- Add captured piece display
- Add timers
- Add usernames on screen
- Add reconnect support
- Add better disconnect handling
- Package the game into an executable
- Add sound effects and animations

## What I Learned

This project demonstrates how a real-time multiplayer game can be built from scratch using sockets. It combines game programming, networking, threading, JSON data formatting, and object-oriented design.

The most important part of the project is the custom networking system. Instead of relying on a game networking framework, the server and clients communicate directly using TCP sockets and manually formatted messages.

## License

This project is for educational and portfolio purposes.
