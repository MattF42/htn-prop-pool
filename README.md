# HTN Proportional (PROP) Mining Pool

This repository contains the backend and API for a Proportional (PROP) mining pool for HTN. It is designed to work in conjunction with the [`Hoosat-Oy/htn-stratum-bridge`](https://github.com/Hoosat-Oy/htn-stratum-bridge).

## Architecture

This pool operates by consuming metrics exposed by the `htn-stratum-bridge` via its Prometheus endpoint. It does not need to be a fork and requires no modification of the bridge itself.

1.  **Stratum Bridge**: The `htn-stratum-bridge` handles all communication with miners and the HTN daemon. It is configured to expose a Prometheus metrics endpoint (`/metrics`).
2.  **Prometheus**: A Prometheus instance scrapes the bridge's metrics endpoint, collecting real-time data on shares, blocks, and worker activity.
3.  **Pool Backend (This Repo)**: This Node.js application provides the core pool logic:
    *   **Round Manager**: Detects when a new block is found by the pool by monitoring the `htn_blochtn_mined` metric.
    *   **Reward Processor**: When a round ends, it queries Prometheus for the `htn_valid_share_diff_counter` data for that round, calculates each miner's proportional reward, and credits their account balance in Redis.
    *   **Payout Processor**: A scheduled job that pays out miners whose balances have reached the minimum payout threshold.
    *   **API**: A simple public API to allow miners to check their stats (hashrate, shares, balance).
4.  **Database**: Redis is used for its speed to manage share data for the current round and miner account balances.
5.  **Web UI**: A minimal web interface that consumes the API to display miner statistics.
