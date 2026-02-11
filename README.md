# Journal: Oracle → Blockchain → Indexed DB → Frontend

## Overview

1. **Backend** reads the journal table (27 fields) from Oracle and **writes full records** (not hashes) to the blockchain.
2. **Smart contract** (`JournalData`) stores each record; **owner** = deployer; only owner can **add writer**; writers can **write** and **update**.
3. **Backend** listens to `RecordWritten` and `RecordUpdated` and **indexes into MongoDB** so queries are fast (e.g. “1 month of data” from chain is slow; from MongoDB is fast).
4. **Frontend** calls backend **API** with filters (time range, currency ID, slip no, etc.) and shows data from the indexed DB.

When someone calls **update** on the contract, the backend listener updates MongoDB so the indexed DB stays in sync.

## Repo layout

- **smart-contract/** – Hardhat, `JournalData.sol` (27-field struct, owner, writers, write/update).
- **backend/** – Oracle read, write-to-chain script, event listener → MongoDB, REST API.
- **frontend/** – React app: list records, filter by date, currency ID, timeline ID, slip no, etc.

## 1. Smart contract

```bash
cd smart-contract
npm install
npx hardhat compile
npx hardhat run scripts/deploy-journal-data.js --network localhost   # or your network
```

Copy the deployed address into **backend** `.env` as `JOURNAL_CONTRACT_ADDRESS`.  
Add the backend wallet as a writer: `contract.addWriter(backendWalletAddress)` (as owner).

## 2. Backend

```bash
cd backend
npm install
```

Set **.env** (see `backend/.env.example`): Oracle, `RPC_URL`, `PRIVATE_KEY`, `JOURNAL_CONTRACT_ADDRESS`, `MONGODB_URI`, `MONGODB_DB_NAME`, `PORT`.

- **Compile contract first** so `smart-contract/artifacts/contracts/JournalData.sol/JournalData.json` exists (backend loads ABI from there).
- Start API + event listener: `npm start`
- One-off sync Oracle → chain: `node src/sync/write-to-chain.js [fromDate] [toDate]` (default: today).

## 3. MongoDB

Run MongoDB locally or set `MONGODB_URI`. The backend creates the DB and indexes; the event listener upserts on every `RecordWritten` and `RecordUpdated`.

## 4. Frontend

```bash
cd frontend
npm install
```

Set **.env**: `REACT_APP_API_URL=http://localhost:3000` (or your backend URL).

```bash
npm start
```

Use the filters (from/to date, currency type ID, record ID, timeline ID, jour method ID, slip no, transaction ID, is credit, limit) and pagination to query the indexed journal data.

## API (backend)

- `GET /api/records?from=&to=&currencyTypeId=&id=&timelineId=&jourMethodId=&slipNo=&transactionId=&isCredit=&limit=50&skip=0` – filtered list from MongoDB.
- `GET /api/records/:blockchainRecordId` – one record by blockchain record ID.

## Contract (summary)

- **Owner**: deployer; only owner can `addWriter(address)`, `removeWriter(address)`, `transferOwnership(address)`.
- **Writers** (and owner): can `write(JournalRecord)` and `update(uint256 recordId, JournalRecord)`.
- **Events**: `RecordWritten(recordId, id, currencyTypeId, createdAt, writer)`, `RecordUpdated(...)`.
- Backend listens to these events and keeps MongoDB in sync; when **update** is used, indexed DB is updated automatically.
