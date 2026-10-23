# Networked Chess: Two-Player Chess over TCP

Playing chess from two separate programs requires a way to pair players and keep their boards in sync. Networked Chess is a Python prototype that pairs two players by username through a TCP relay and displays each player's board with Pygame. It demonstrates socket communication, concurrent connections, and client-side game state without a chess engine or networking framework.

## Team Members

- [Ethan Lahr (@NeefMan)](https://github.com/NeefMan) — author and maintainer.

## Setup and Installation

**Requirements:** [uv](https://docs.astral.sh/uv/getting-started/installation/) and a desktop environment that can open a Pygame window. The repository pins Python 3.14 and `pygame-ce==2.5.8` in `.python-version`, `pyproject.toml`, and `uv.lock`. On the first `uv run`, uv sets up the Python environment and installs the locked dependency automatically.

```bash
git clone https://github.com/NeefMan/networked-chess.git
cd networked-chess
```

The included configuration connects two players on the same computer through `127.0.0.1:5001`. Keep the server running in the first terminal:

```bash
uv run server.py
```

In **each of two other terminals**, start a player:

```bash
uv run main.py
```

Use different usernames and enter each other's username as the opponent. For example, enter `alice` then `bob` in one client, and `bob` then `alice` in the other. The first client waits for the second before opening the board. Run `main.py`, not `client.py`. `client.py` only defines the networking class. Press **Ctrl+C** in the server terminal when finished.

## Project Highlights

- **One server, two clients:** a threaded server pairs players, randomly assigns the first turn, and relays moves between two graphical clients.
- **A small custom protocol:** JSON messages carry usernames, moves, and reset requests; a delimiter separates messages on the TCP stream.
- **Interactive 8 × 8 board:** the clients display piece sprites, highlight available moves, flip the board, and reset both boards.

## How It Works

1. **Connect and pair.** `main.py` creates a `Client`, asks for a username and opponent, and connects to `server.py`. The server handles each connection in a separate thread so one player can wait while the other joins. When both usernames are registered, it randomly chooses who moves first; the clients map that choice to white and black.
2. **Frame messages.** Client and server serialize requests as JSON followed by `*&^%`. TCP delivers a stream of bytes, so the delimiter lets the receiver identify the end of a message. For example:

   ```text
   {"username":"alice","task":"move","to_user":"bob","move":[[4,6],[4,4]]}*&^%
   ```

3. **Calculate and relay a move.** `main.py` calculates available destinations from its local board state. When a player moves, `client.py` sends the source and destination coordinates. `server.py` stores that move for the named opponent; it does not check whether the move is legal.
4. **Update the other board.** Each client polls the server for a pending move about once per second. On receipt, it applies the coordinates to its own board and switches turns. `R` sends a reset request through the server so both clients recreate their boards.

This split keeps the server small and makes the socket protocol easy to inspect, but it trusts clients to maintain valid game state.
