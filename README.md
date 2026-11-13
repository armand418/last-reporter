# $LAST reporter (GitHub Actions)

The always-on reporter bot of **$LAST**, the NEAR mainnet memecoin whose trading fees fund an on-chain jackpot.
Main project: https://github.com/nielslecodeur/LAST · Live jackpot: https://last-jackpot.vercel.app · Buy: https://nearly.trade/last-2.nearlytrade.near

## What it does

Every 10 seconds it lists the new transactions of the token `last-2.nearlytrade.near` (FastNEAR transactions API) and:

1. records every buy of at least 0.5 NEAR on the jackpot contract `pot.heavycat9990.near` with `record_buy(buyer, near_in, tx_hash, buy_timestamp_ns)` (the buyer takes the crown, the 60-minute timer restarts from the buy time);
2. calls `settle_round` once a round is past its deadline + 2 minutes. The contract then pays 90% of the jackpot to the winner by itself.

A buy made after the current deadline is only recorded after that round is settled, and expired rounds are settled only after every earlier buy is recorded. The bot keeps its position in `state/cursor.json`, so a restart never skips or reorders buys.

## How it runs

`.github/workflows/reporter.yml` starts every 6 hours (and on demand from the Actions tab). Each run lasts about 5 h 45 min, then commits the cursor and exits; the next run resumes from it. GitHub may start a scheduled run a few minutes late: a buy reported late still counts as long as its round is not settled yet.

## Key and permissions

The workflow signs with a **function-call access key** of `heavycat9990.near` that can only call `record_buy` and `settle_round` on `pot.heavycat9990.near` (it cannot transfer NEAR or call anything else). It is stored as the repository secrets `REPORTER_PRIVATE_KEY` and `REPORTER_PUBLIC_KEY`, written to disk only for the run and deleted at the end. Nobody, including this key, can withdraw the jackpot: it only leaves the contract as a prize.

## Run it locally

```bash
python3 reporter.py --pot pot.heavycat9990.near --token last-2.nearlytrade.near --reporter heavycat9990.near --dry-run
```

Python 3 standard library only; signing uses [near-cli-rs](https://github.com/near/near-cli-rs) with the legacy keychain.

## License

MIT
