# Squid running against an external DB

This squid captures `Transfer(address,address,uint256)` events emitted by the contract at `env.CONTRACT_ADDRESS` ([SQD token contract](https://arbiscan.io/address/0x1337420ded5adb9980cfc35f8f2b054ea86f8ab1) by default). It keeps up with network updates [in real time](https://docs.sqd.ai/sdk/resources/unfinalized-blocks/). Balances of individual accounts are computed and kept up to date.

The handler function for batches of `Transfer`s lives in a [separate module](/src/batchHandlers/transfer.ts) and has a [test](/src/batchHandlers/transfer.int.test.ts). The test can run locally or [as a GitHub Action](/.github/workflows/run_tests.yml).

Squid's [manifest](/squid.yaml) instructs SQD Cloud to use an external Postgres-compatible database. See [Quickstart/Cloud](#cloud) for deployment instructions.

Dependencies: Node.js v20 or newer, Docker.

## Quickstart

### Local

```bash
git clone https://github.com/abernatskiy/external-db-squid
cd external-db-squid
npm i
docker compose up -d
npm run build
npx squid-typeorm-migration apply
node -r dotenv/config lib/main.js
```
then in a separate terminal
```bash
npx squid-graphql-server
```
A GraphiQL playground will be available at [localhost:4350/graphql](http://localhost:4350/graphql).

### Cloud

1. Register at [app.subsquid.io](https://app.subsquid.io).
2. Install SQD CLI:
   ```bash
   npm i -g @subsquid/cli
   ```
3. Go to [API tokens](https://app.subsquid.io/profile/api-tokens), create one and use it to authenticate your CLI as directed.
4. Store the credentials of your external database as Cloud secrets:
   ```bash
   sqd secrets add MY_PGHOST <your_db_hostname>
   sqd secrets add MY_PGDATABASE <your_db_name>
   sqd secrets add MY_PGUSER <your_db_user>
   sqd secrets add MY_PGPASSWORD <your_db_password>
   ```
5. Clone the repo and deploy:
   ```bash
   git clone https://github.com/abernatskiy/external-db-squid
   cd external-db-squid
   npm i
   sqd deploy .
   ```

## Testing

The only test currently available is at [src/batchHandlers/transfer.int.test.ts](/src/batchHandlers/transfer.int.test.ts). It uses a live database and tests the behavior of the [src/batchHandlers/transfer.ts](/src/batchHandlers/transfer.ts) module.

To run it, enter the project folder and make sure that the database is up:
```bash
docker compose up -d
```
then run
```bash
npm test
```

There's also a [Github workflow](/.github/workflows/run_tests.yml) that runs the test.
