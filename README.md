## P2E Game Backend

This repository contains the backend service for the **Tronzit P2E Game**, built on **Node.js/TypeScript**, **Express**, **MongoDB/Mongoose**, **Socket.IO**, and **Solana** (`@solana/web3.js`, `@project-serum/anchor`).

The service exposes a REST API and real‑time WebSocket events to:
- manage P2E game creation and entries,
- track players, bets, and winners,
- store historical data in MongoDB,
- broadcast game state (chat, participants, timers, winners) to connected clients.

---

## Tech Stack

- **Runtime**: Node.js (TypeScript)
- **Web Framework**: Express
- **Database**: MongoDB with Mongoose
- **Real‑time**: Socket.IO
- **Blockchain**: Solana (`@solana/web3.js`, `@project-serum/anchor`)
- **Tooling**: `ts-node`, `nodemon`, `prettier`, `typescript`

---

## Getting Started

### Prerequisites

- **Node.js** (LTS recommended)
- **Yarn** or **npm**
- **MongoDB** instance (local or hosted)
- Access to a **Solana RPC endpoint** (devnet or mainnet, depending on your setup)

### Installation

```bash
# Install dependencies
yarn install

# or
npm install
```

### Environment Configuration

Create a `.env` file in the project root (alongside `package.json`) and define at least:

```bash
DB_CONNECTION=<your-mongodb-connection-string>
PORT=3002                          # optional, defaults to 3002
```

> **Note**: Existing environment files (such as `grave.env`, `infinite.env`) are typically used for deployment/hosting; do **not** commit secrets and private keys.

---

## Scripts

Defined in `package.json`:

- **`yarn start`**  
  Compiles TypeScript and starts the development server with `nodemon` on `src/index.ts`.

- **`yarn build`**  
  Cleans `dist` and compiles the TypeScript sources with `tsc`.

- **`yarn watch-ts`**  
  Runs the TypeScript compiler in watch mode.

- **`yarn watch-node`**  
  Runs `nodemon` on the compiled JavaScript (`dist/index.js`).

- **`yarn lint` / `yarn lint:fix`**  
  Runs Prettier in check or write mode over the codebase.

---

## API Overview

The backend exposes HTTP endpoints over Express (default port: **3002**) and WebSocket events via Socket.IO.

### Game Lifecycle

- **POST** `/requestCreate`  
  Request to create a new P2E game (protected by an internal mutex to avoid concurrent creations).

- **POST** `/endRequest`  
  Marks the end of a create request / processing cycle.

- **POST** `/createGame`  
  Body parameters:
  - `txId`: Solana transaction ID
  - `encodedTx`: encoded transaction payload  
  Starts a new game round and triggers on‑chain interaction.

- **POST** `/enterGame`  
  Body parameters:
  - `txId`: Solana transaction ID
  - `encodedTx`: encoded transaction payload  
  Adds a player entry into the current game pool.

- **GET** `/getRecentGame`  
  Returns the most recent game PDA, end timestamp, and player data.

### Chat & Stats

- **POST** `/writeMessage`  
  Persists a chat message and broadcasts the updated chat log.

- **GET** `/getMessage`  
  Fetches the latest chat messages.

- **GET** `/getTimes`  
  Returns how many games have been played.

- **GET** `/getTotalSum`  
  Aggregates and returns the total payout across winners.

- **GET** `/getWinners`  
  Returns the latest P2E winners.

- **GET** `/getBetCount`  
  Returns the current in‑memory bet counter for the running process.

---

## Real‑time Events (Socket.IO)

When connected to the Socket.IO server, clients can listen to events such as:

- **`connectionUpdated`** – current number of active socket connections.
- **`chatUpdated`** – updated chat messages list.
- **`gameStarting`** – emitted when a game round is starting.
- **`startGame`** – emitted with initial game data when a game is created.
- **`endTimeUpdated`** – countdown/end timestamp updates with current players.
- **`newGameReady`** – indicates that a new game can be joined.
- **`heartbeat`** – periodic heartbeat (every second) with the current timestamp.

Exact payloads can be inspected in the corresponding handlers in `src/index.ts` and `src/db.ts`.

---

## Development Notes

- Ensure MongoDB is running and `DB_CONNECTION` is valid before starting the server.
- The service currently connects to a configured Solana network via `SOLANA_NETWORK` in `src/index.ts`; adjust this for your environment if needed.
- Timers and cooldown behavior are configured in `config.ts` (`FIRST_COOLDOWN`, `NEXT_COOLDOWN`, `CLEAR_COOLDOWN`, `REFUND_TIMEOUT`).

---

## License

This project is licensed under the **ISC License** (see `package.json` for details).