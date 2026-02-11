# BlockScout explorer for Hardhat node

Use [BlockScout](https://www.blockscout.com/) as the block explorer for your local Hardhat node: blocks, transactions, accounts, contract verification, and read/write UI.

**Prerequisites:** Docker 20.10+, Docker Compose 2.x, and your Hardhat node running on port **8545**.

## 1. Start Hardhat node

In one terminal:

```bash
npm run node
```

Leave it running. BlockScout will connect to `http://localhost:8545` (or `host.docker.internal:8545` from inside Docker).

## 2. Run BlockScout with Docker

BlockScout provides an official Hardhat config. From a **new** terminal:

```bash
git clone https://github.com/blockscout/blockscout.git
cd blockscout/docker-compose
docker compose -f hardhat-network.yml up -d
```

- First run may take a few minutes (pulls images, starts Postgres, Redis, backend, frontend, etc.).
- The explorer UI will be at **http://localhost** (or http://127.0.0.1).
- Backend API: http://localhost/api

## 3. Stop BlockScout

```bash
cd blockscout/docker-compose
docker compose -f hardhat-network.yml down
```

To reset indexer data (optional):

```bash
docker compose -f hardhat-network.yml down -v
# and/or remove blockscout DB volume if you want a full reset
```

## Notes

- **Hardhat chain ID** is 31337; the Hardhat config sets `CHAIN_ID=31337` and `NEXT_PUBLIC_NETWORK_ID=31337`.
- **RPC from Docker:** The compose file uses `host.docker.internal:8545` so the containers can reach your node on the host. On Linux, if that fails, run the Hardhat node bound to `0.0.0.0` (e.g. in `hardhat.config.js` set the node URL to `http://0.0.0.0:8545` for the server binding) and use `host.docker.internal` or your machine’s IP in the BlockScout env if needed.
- **Contract verification:** Use BlockScout’s “Verify & Publish” in the UI, or the backend verification API, for contracts deployed on your local chain.
- Env overrides: edit `blockscout/docker-compose/envs/common-blockscout.env` and `common-frontend.env` if you need to point to another RPC or change ports.

## One-time clone, reuse

You only need to clone the BlockScout repo once. After that, start the node and run:

```bash
cd /path/to/blockscout/docker-compose
docker compose -f hardhat-network.yml up -d
```

Then open http://localhost in your browser.
