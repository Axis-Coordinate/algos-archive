# Small Trader Alpha #7: Perpetual Arbitrage Pt. 2

Source HTML: [`html/2024-10-01-small-trader-alpha-7-perpetual-arbitrage.html`](../html/2024-10-01-small-trader-alpha-7-perpetual-arbitrage.html)

# Small Trader Alpha #7: Perpetual Arbitrage Pt. 2

| 항목 | 값 |
| --- | --- |
| 날짜 | 2024-10-01 |
| 접근 | 유료 |
| URL | https://www.algos.org/p/small-trader-alpha-7-perpetual-arbitrage |
| 부제 | Advanced perpetual arbitrage strategy |

---

[![Create a more glorious and grand cover image for a Substack article about implementing a trading algorithm, in landscape format. The cover should resemble a prestigious, 'holy grail' textbook on perpetual arbitrage for cryptocurrency, sitting on a luxurious wooden table. The book design should be ornate, with gold accents, intricate borders, and a bold title that reads 'Perpetual Arbitrage Trading' in an elegant, commanding font. Include subtle, refined crypto-themed symbols like Bitcoin and Ethereum logos, and trading charts. The lighting should be dramatic, casting a warm, radiant glow on the book, emphasizing its grandeur and importance.](images/5747853ff488.webp)](images/e0cec79ad900.webp)

### Introduction

---

In the previous article in the small trader alpha series, we walked through a taker strategy quite broadly. Now, we will dive into the implementation details of that providing the *FULL* code for a perpetual arbitrage scanner in Rust that can be put into production.

This article will explain the code, but the appendix will contain the entire code which can easily be copy-pasted. This is the easiest method instead of everyone having to email me for access to a GitHub. In making this project, we were also able to contribute to barter-rs, an open-source project similar to CCXT but for Rust.

This code is presented with the intention that the comments and naming should be self-documenting. Thus, it is up to the reader to build that understanding of how it works by reading it. This is a simple expectation I hope.

### Using the Scanner

---

We have two binaries, this is the scanner searching for arbitrages after startup. If we run “cargo run --bin data --release”, then we can see the below output. It even found an arb! Sadly, this one likely isn’t real as we haven’t set fees so by default it is 0, but this is a short sample.

[![](images/4b0b4940c600.png)](images/5925db18909a.png)

### Code

---

#### File Structure:

Starting with file structure:

[![](images/7356395200a0.png)](images/d5ef8f31d94e.png)

You can run cargo new perpscan-draft, or whatever you want to call it (but with the names changed out in .toml later).

#### Cargo Files:

This is Cargo.toml

```
[package]
name = "perpscan"
version = "0.1.0"
edition = "2021"

[dependencies]
barter = {git = "https://github.com/imbrem/barter-rs", branch = "okx-pong"}
barter-data = {git = "https://github.com/imbrem/barter-rs", branch = "okx-pong"}
barter-integration = {git = "https://github.com/imbrem/barter-rs", branch = "okx-pong"}
chrono = "0.4.38"
clap = { version = "4.5.16", features = ["derive"] }
color-eyre = "0.6.3"
csv = "1.3.0"
flate2 = "1.0.31"
futures = "0.3.30"
fxhash = "0.2.1"
indexmap = "2.4.0"
itertools = "0.13.0"
rand = "0.8.5"
rand_xoshiro = "0.6.0"
serde = { version = "1.0.208", features = ["derive"] }
tokio = "1.39.2"
tracing = "0.1.40"
tracing-subscriber = "0.3.18"
```

This is .gitignore:

```
/target
.data
secrets.json
D:/
```

#### Rust Files:

Now, time for the fun part. Let’s start with lib.rs:

[![](images/7bf3bdeb740f.png)](images/41f42f0129b8.png)

```
use core::f64;
use std::collections::VecDeque;

use barter_data::{event::MarketEvent, subscription::book::OrderBookL1};
use barter_integration::model::{instrument::Instrument, Exchange};
use chrono::{DateTime, TimeDelta, Utc};
// use tokio::try_join;
use indexmap::IndexMap;
use tracing::info;

/// Statistics and order book for a given exchange
#[derive(Debug)]
pub struct ExchangeData {
    /// Exponential moving average of latency of messages from the exchange
    latency_ema: TimeDelta,
    /// Latency of the most recent message from the exchange
    latency: TimeDelta,
    /// Time between the most recent two updates from the exchange
    time_between_updates: TimeDelta,
    /// Exponential moving average of time between updates from the exchange
    time_between_updates_ema: TimeDelta,
    /// Order book data from the exchange
    book: OrderBookL1,
    /// Midpoint of the order book
    midpoint: f64,
    /// Number of updates from the exchange so far
    updates: u64,
    /// Whether the most recent update was a repeat of the previous update
    repeat: bool,
}

impl ExchangeData {
    pub fn new() -> ExchangeData {
        ExchangeData {
            latency_ema: Default::default(),
            latency: Default::default(),
            time_between_updates: Default::default(),
            time_between_updates_ema: Default::default(),
            book: OrderBookL1 {
                best_bid: Default::default(),
                best_ask: Default::default(),
                last_update_time: Default::default(),
            },
            midpoint: f64::NAN,
            updates: 0,
            repeat: false,
        }
    }
}

impl ExchangeData {
    pub fn update(&mut self, book: MarketEvent<Instrument, OrderBookL1>, k: i32) {
        let latency = book.received_time - book.exchange_time;
        let kl = k.min(self.updates.try_into().unwrap_or(i32::MAX));
        self.latency = latency;
        self.latency_ema = ((self.latency_ema * kl) + latency) / (kl + 1);

        if book.kind.last_update_time > self.book.last_update_time && self.updates > 0 {
            let time_between_updates = book.kind.last_update_time - self.book.last_update_time;
            self.time_between_updates = time_between_updates;
            let kl = k.min(self.updates.try_into().unwrap_or(i32::MAX) - 1);
            self.time_between_updates_ema =
                ((self.time_between_updates_ema * kl) + time_between_updates) / (kl + 1);
            self.repeat = false;
        } else {
            self.repeat = true;
        }

        self.book = book.kind;

        // TODO: use NaN?
        self.midpoint = if self.book.best_ask.amount == 0.0 {
            self.book.best_bid.price
        } else if self.book.best_bid.amount == 0.0 {
            self.book.best_ask.price
        } else {
            (self.book.best_ask.price + self.book.best_bid.price) / 2.0
        };

        self.updates += 1;
    }
}

#[derive(Debug)]
pub struct PerpArbScanner {
    /// Threshold for entering a trade
    pub entry_threshold: f64,
    /// Total profit and loss
    pub pnl: f64,
    /// Trading fees for each exchange; defaults to zero
    pub trading_fees: IndexMap<Exchange, f64>,
    /// Number of updates processed so far for all exchanges
    pub updates: u64,
    /// Execution latency for trades
    pub execution_latency: TimeDelta,
    /// Data for each exchange; see [`ExchangeData`]
    pub exchange_data: IndexMap<Exchange, ExchangeData>,
    /// Pending trades that have been identified but not yet executed
    pub pending_trades: VecDeque<PendingTrade>,
    /// Live trades that have been executed but not yet exited
    pub live_trades: Vec<LiveTrade>,
    /// Completed trades that have been exited
    pub completed_trades: Vec<CompletedTrade>,
    /// Time of the most recent update
    pub latest_update: DateTime<Utc>,
}

#[derive(Debug, PartialEq)]
pub struct PendingTrade {
    pub short_exchange: usize,
    pub long_exchange: usize,
    pub time: DateTime<Utc>,
    pub attempted_short_sell_price: f64,
    pub attempted_long_buy_price: f64,
}

#[derive(Debug, PartialEq)]
pub struct LiveTrade {
    pub short_exchange: usize,
    pub long_exchange: usize,
    pub amount: f64,
    pub short_sell_price: f64,
    pub long_buy_price: f64,
    pub attempted_short_sell_price: f64,
    pub attempted_long_buy_price: f64,
}

#[derive(Debug, PartialEq)]
pub struct CompletedTrade {
    pub short_exchange: Exchange,
    pub long_exchange: Exchange,
    pub amount: f64,
    pub long_buy_price: f64,
    pub long_sell_price: f64,
    pub short_sell_price: f64,
    pub short_buy_price: f64,
    pub attempted_short_sell_price: f64,
    pub attempted_long_buy_price: f64,
}

impl CompletedTrade {
    pub fn fee_free_pnl(&self) -> f64 {
        let long_pnl = self.amount * (self.long_sell_price - self.long_buy_price);
        let short_pnl = self.amount * (self.short_sell_price - self.short_buy_price);
        long_pnl + short_pnl
    }

    pub fn total_fees(&self, trading_fees: &IndexMap<Exchange, f64>) -> f64 {
        let long_fee = self.amount
            * trading_fees
                .get(&self.short_exchange)
                .copied()
                .unwrap_or_default();
        let short_fee = self.amount
            * trading_fees
                .get(&self.long_exchange)
                .copied()
                .unwrap_or_default();
        long_fee + short_fee
    }

    pub fn pnl(&self, trading_fees: &IndexMap<Exchange, f64>) -> f64 {
        self.fee_free_pnl() - self.total_fees(trading_fees)
    }
}

impl PerpArbScanner {
    pub fn new(
        entry_threshold: f64,
        execution_latency: TimeDelta,
        trading_fees: IndexMap<Exchange, f64>,
    ) -> PerpArbScanner {
        assert!(entry_threshold > 1.0);
        PerpArbScanner {
            entry_threshold,
            updates: 0,
            pnl: 0.0,
            execution_latency,
            exchange_data: IndexMap::default(),
            pending_trades: VecDeque::default(),
            live_trades: Vec::default(),
            latest_update: DateTime::default(),
            trading_fees,
            completed_trades: Vec::default(),
        }
    }
}

impl PerpArbScanner {
    pub fn handle_book(&mut self, book: MarketEvent<Instrument, OrderBookL1>) {
        self.update_books(book);
        self.process_pending_trades(self.latest_update);
        self.entry_opportunities();
        self.exit_opportunities();
    }

    pub fn update_books(&mut self, book: MarketEvent<Instrument, OrderBookL1>) {
        let data = self
            .exchange_data
            .entry(book.exchange.clone())
            .or_insert(ExchangeData::new());
        self.latest_update = book.received_time;

        const EMA_WINDOW: i32 = 20;
        // const DATA_RATE: u64 = 25;

        // if data.updates % DATA_RATE == 0 {
        //     info!("{exchange} ==> {book:?}");
        // }

        data.update(book, EMA_WINDOW);

        self.updates += 1;
    }

    pub fn entry_opportunities(&mut self) {
        let num_exchanges = self.exchange_data.len();
        for long_exchange in 0..num_exchanges {
            for short_exchange in 0..num_exchanges {
                if long_exchange == short_exchange {
                    continue;
                }
                let ask_data = &self.exchange_data[long_exchange];
                let bid_data = &self.exchange_data[short_exchange];
                if bid_data.book.best_bid.amount == 0.0 || ask_data.book.best_ask.amount == 0.0 {
                    continue;
                }
                let short_sell_price = bid_data.book.best_bid.price;
                let long_buy_price = ask_data.book.best_ask.price;
                let spread_ratio = short_sell_price / long_buy_price;
                if spread_ratio > self.entry_threshold
                    && !self.live_trades.iter().any(|trade| {
                        (trade.long_exchange == short_exchange
                            || trade.short_exchange == short_exchange)
                            && (trade.long_exchange == long_exchange
                                || trade.short_exchange == long_exchange)
                    })
                    && !self
                        .pending_trades
                        .iter()
                        .any(|trade| trade.long_exchange == long_exchange)
                {
                    self.pending_trades.push_back(PendingTrade {
                        short_exchange,
                        long_exchange,
                        time: self.latest_update,
                        attempted_short_sell_price: short_sell_price,
                        attempted_long_buy_price: long_buy_price,
                    });
                }
            }
        }
    }

    pub fn exit_opportunities(&mut self) {
        self.live_trades.retain(|trade| {
            let short_book = self.exchange_data[trade.short_exchange].book;
            let long_book = self.exchange_data[trade.long_exchange].book;
            let midpx_spread = (short_book.best_bid.price + short_book.best_ask.price)
                / (long_book.best_ask.price + long_book.best_bid.price);
            if midpx_spread <= 1.0 {

                let completed_trade = CompletedTrade {
                    short_exchange: self
                        .exchange_data
                        .get_index(trade.short_exchange)
                        .unwrap()
                        .0
                        .clone(),
                    long_exchange: self
                        .exchange_data
                        .get_index(trade.long_exchange)
                        .unwrap()
                        .0
                        .clone(),
                    amount: trade.amount,
                    long_buy_price: trade.long_buy_price,
                    long_sell_price: long_book.best_ask.price,
                    short_sell_price: trade.short_sell_price,
                    short_buy_price: short_book.best_bid.price,
                    attempted_short_sell_price: trade.attempted_short_sell_price,
                    attempted_long_buy_price: trade.attempted_long_buy_price,
                };

                let fee_free_pnl = completed_trade.fee_free_pnl();
                let fees = completed_trade.total_fees(&self.trading_fees);
                let pnl = completed_trade.pnl(&self.trading_fees);

                info!(
                    "exiting trade {}, {} @ {midpx_spread}", 
                    completed_trade.short_exchange, completed_trade.long_exchange
                );

                self.completed_trades.push(completed_trade);
                self.pnl += pnl;

                info!("completed trade pnl: {fee_free_pnl}, fees: {fees}, net pnl: {pnl}, total pnl {}", self.pnl);

                false
            } else {
                true
            }
        });
    }

    pub fn process_pending_trades(&mut self, time: DateTime<Utc>) {
        while let Some(trade) = self.pending_trades.front() {
            if trade.time + self.execution_latency > time {
                break;
            }
            let trade = self.pending_trades.pop_front().unwrap();
            let bid = self.exchange_data[trade.short_exchange].book.best_bid;
            let ask = self.exchange_data[trade.long_exchange].book.best_ask;
            let amount = bid.amount.min(ask.amount);
            if amount != 0.0 {
                // info!(
                //     "entering trade: {}; {}",
                //     trade.short_exchange, trade.long_exchange
                // );
                self.live_trades.push(LiveTrade {
                    short_exchange: trade.short_exchange,
                    long_exchange: trade.long_exchange,
                    amount: bid.amount.min(ask.amount),
                    short_sell_price: bid.price,
                    long_buy_price: ask.price,
                    attempted_short_sell_price: trade.attempted_short_sell_price,
                    attempted_long_buy_price: trade.attempted_long_buy_price,
                });
            }
        }
    }
}

#[cfg(test)]
mod tests {
    use barter_data::{
        event::MarketEvent,
        subscription::book::{Level, OrderBookL1},
    };
    use barter_integration::model::instrument::{kind::InstrumentKind, Instrument};
    use chrono::{DateTime, TimeDelta};

    use super::*;

    const TEST_DELAY: TimeDelta = TimeDelta::milliseconds(30);

    fn test_event(
        exchange: &'static str,
        bid: f64,
        ask: f64,
        time: i64,
    ) -> MarketEvent<Instrument, OrderBookL1> {
        MarketEvent {
            exchange_time: DateTime::from_timestamp_millis(time).unwrap(),
            received_time: DateTime::from_timestamp_millis(time).unwrap() + TEST_DELAY,
            exchange: exchange.into(),
            instrument: Instrument::new("TST", "USDT", InstrumentKind::Perpetual),
            kind: OrderBookL1 {
                last_update_time: DateTime::from_timestamp_millis(time).unwrap(),
                best_bid: Level {
                    price: bid,
                    amount: 1.0,
                },
                best_ask: Level {
                    price: ask,
                    amount: 1.0,
                },
            },
        }
    }

    // #[test]
    // fn no_arb() {
    //     let trading_fees = Default::default();
    //     let mut scanner = PerpArbScanner::new(1.0005, TimeDelta::milliseconds(500), trading_fees);

    //     let events = [
    //         test_event("A", 100.0, 101.0, 0),
    //         test_event("A", 101.0, 102.0, 100),
    //         test_event("B", 101.0, 102.0, 200),
    //         test_event("C", 101.0, 102.0, 300),
    //     ];

    //     let num_events = events.len() as u64;

    //     for book in events {
    //         scanner.handle_book(book);
    //     }

    //     assert_eq!(scanner.pending_trades.len(), 0);
    //     assert_eq!(scanner.pnl, 0.0);
    //     assert_eq!(scanner.updates, num_events);
    // }

    #[test]
    fn arb() {
        let trading_fees = Default::default();
        let mut scanner = PerpArbScanner::new(1.0005, TimeDelta::milliseconds(500), trading_fees);

        let pre_trade = [
            test_event("A", 99.0, 100.0, 0),
            test_event("B", 101.0, 102.0, 100),
        ];

        let mut num_events = pre_trade.len() as u64;

        for book in pre_trade {
            scanner.handle_book(book);
        }

        // We saw exchange "A" first, _then_ B
        assert_eq!(
            scanner.exchange_data.get_index(0).unwrap().0.to_string(),
            "A"
        );
        assert_eq!(
            scanner.exchange_data.get_index(1).unwrap().0.to_string(),
            "B"
        );

        let pending_trade = vec![PendingTrade {
            long_exchange: 0,
            short_exchange: 1,
            time: DateTime::from_timestamp_millis(100).unwrap() + TEST_DELAY,
            attempted_short_sell_price: 101.0,
            attempted_long_buy_price: 100.0,
        }];

        assert_eq!(scanner.pending_trades, pending_trade);
        assert_eq!(scanner.live_trades, vec![]);
        assert_eq!(scanner.completed_trades, vec![]);
        assert_eq!(scanner.pnl, 0.0);
        assert_eq!(scanner.updates, num_events);

        let during_trade = [
            test_event("A", 100.0, 101.0, 300),
            test_event("B", 103.0, 104.0, 400),
        ];

        num_events += during_trade.len() as u64;

        for book in during_trade {
            scanner.handle_book(book);
        }

        // Pending trade does not change, since not enough time has passed for it to become live
        assert_eq!(scanner.pending_trades, pending_trade);
        assert_eq!(scanner.live_trades, vec![]);
        assert_eq!(scanner.completed_trades, vec![]);
        assert_eq!(scanner.pnl, 0.0);
        assert_eq!(scanner.updates, num_events);

        let on_execution = [
            test_event("A", 100.0, 101.0, 600),
            test_event("B", 103.0, 104.0, 700),
        ];

        num_events += on_execution.len() as u64;

        for book in on_execution {
            scanner.handle_book(book);
        }

        let live_trade = vec![LiveTrade {
            short_exchange: 1,
            long_exchange: 0,
            amount: 1.0,
            short_sell_price: 103.0,
            long_buy_price: 101.0,
            attempted_short_sell_price: 101.0,
            attempted_long_buy_price: 100.0,
        }];

        // Pending trade is now live
        assert_eq!(scanner.pending_trades, vec![]);
        assert_eq!(scanner.live_trades, live_trade);
        assert_eq!(scanner.completed_trades, vec![]);
        assert_eq!(scanner.pnl, 0.0);
        assert_eq!(scanner.updates, num_events);

        // Things don't change even if the deal gets much better, for now
        let while_live = [
            test_event("A", 100.0, 101.0, 800),
            test_event("B", 110.0, 112.0, 900),
        ];

        num_events += while_live.len() as u64;

        for book in while_live {
            scanner.handle_book(book);
        }

        // Live trade does not change
        assert_eq!(scanner.pending_trades, vec![]);
        assert_eq!(scanner.live_trades, live_trade);
        assert_eq!(scanner.completed_trades, vec![]);
        assert_eq!(scanner.pnl, 0.0);
        assert_eq!(scanner.updates, num_events);

        // We exit the trade
        let exit_trade = [
            test_event("A", 100.0, 101.5, 1000),
            test_event("B", 100.0, 101.0, 1100),
        ];

        num_events += exit_trade.len() as u64;

        for book in exit_trade {
            scanner.handle_book(book);
        }

        let completed_trade = vec![CompletedTrade {
            short_exchange: "B".into(),
            long_exchange: "A".into(),
            amount: 1.0,
            long_buy_price: 101.0,
            long_sell_price: 101.5,
            short_sell_price: 103.0,
            short_buy_price: 100.0,
            attempted_short_sell_price: 101.0,
            attempted_long_buy_price: 100.0,
        }];

        assert_eq!(scanner.pending_trades, vec![]);
        assert_eq!(scanner.live_trades, vec![]);
        assert_eq!(scanner.completed_trades, completed_trade);
        assert_eq!(scanner.pnl, 3.5);
        assert_eq!(scanner.updates, num_events);
    }
}
```

Now live.rs:

[![](images/dfccc8d9cb93.png)](images/ad7ed49aca21.png)

```
use core::f64;

use barter_data::{
    exchange::{
        binance::futures::BinanceFuturesUsd,
        bybit::{futures::BybitServerPerpetualsUsd, Bybit},
        okx::Okx,
    },
    streams::Streams,
    subscription::book::OrderBooksL1,
};
use barter_integration::model::instrument::{kind::InstrumentKind, Instrument};
use chrono::TimeDelta;
use color_eyre::eyre::Result;
use futures::StreamExt;
// use tokio::try_join;
use clap::Parser;
use indexmap::IndexMap;
use tracing_subscriber;

use perpscan::PerpArbScanner;

#[derive(Parser, Debug)]
#[command(version = "0.0", about = "Perpetual arbitrage scanner")]
pub struct Args {
    #[arg(long = "entry-threshold", short = 't', default_value_t = 1.0005)]
    entry_threshold: f64,
    #[arg(long = "execution-latency", short = 'l', default_value_t = 500)]
    execution_latency: u32,
}

#[tokio::main]
async fn main() -> Result<()> {
    let args = Args::parse();

    color_eyre::install()?;
    tracing_subscriber::fmt::init();

    let books = Streams::<OrderBooksL1>::builder()
        .subscribe([
            (
                BinanceFuturesUsd::default(),
                "core",
                "usdt",
                InstrumentKind::Perpetual,
                OrderBooksL1,
            ),
            (
                BinanceFuturesUsd::default(),
                "eth",
                "usdt",
                InstrumentKind::Perpetual,
                OrderBooksL1,
            ),
        ])
        .subscribe([
            (Okx, "core", "usdt", InstrumentKind::Perpetual, OrderBooksL1),
            (Okx, "eth", "usdt", InstrumentKind::Perpetual, OrderBooksL1),
        ])
        .subscribe([
            (
                Bybit::<BybitServerPerpetualsUsd>::default(),
                "core",
                "usdt",
                InstrumentKind::Perpetual,
                OrderBooksL1,
            ),
            (
                Bybit::<BybitServerPerpetualsUsd>::default(),
                "eth",
                "usdt",
                InstrumentKind::Perpetual,
                OrderBooksL1,
            ),
        ])
        .init()
        .await?;

    let mut books = books.join_map().await;
    let mut scanners: IndexMap<Instrument, PerpArbScanner> = IndexMap::new();

    let instruments = [
        Instrument {
            base: "CORE".into(),
            quote: "USDT".into(),
            kind: InstrumentKind::Perpetual,
        },
        Instrument {
            base: "ETH".into(),
            quote: "USDT".into(),
            kind: InstrumentKind::Perpetual,
        },
    ];

    for instrument in instruments {
        scanners.insert(
            instrument.clone(),
            PerpArbScanner::new(
                args.entry_threshold,
                TimeDelta::milliseconds(args.execution_latency as i64),
                IndexMap::default(),
            ),
        );
    }

    while let Some((_exchange, book)) = books.next().await {
        if let Some(scanner) = scanners.get_mut(&book.instrument) {
            // info!("handling book {:?}", book);
            scanner.handle_book(book);
        }
    }

    Ok(())
}
```

Now for data.rs:

[![](images/2c883d9012a5.png)](images/9c105b7bd33e.png)

```
use chrono::{DateTime, TimeDelta};
use clap::Parser;
use color_eyre::eyre::Result;
use core::f64;
use flate2::bufread::GzDecoder;
use indexmap::IndexMap;
use serde::{Deserialize, Serialize};
use std::{fs::File, io::BufReader};
use tracing::{info, warn};

use barter_data::{
    event::MarketEvent,
    // exchange::ExchangeId,
    subscription::book::{Level, OrderBookL1},
};
use barter_integration::model::{
    instrument::{kind::InstrumentKind, symbol::Symbol, Instrument},
    Exchange,
};
// use tracing::info;

use perpscan::PerpArbScanner;

#[derive(Parser, Debug)]
#[command(version = "0.0", about = "Perpetual arbitrage scanner")]
pub struct Args {
    #[arg(long = "entry-threshold", short = 'e', default_value_t = 1.0005)]
    entry_threshold: f64,
    #[arg(long = "execution-latency", short = 'l', default_value_t = 500)]
    execution_latency: u64,
    #[arg(long = "data-latency", short = 'j', default_value_t = 50)]
    data_latency: u64,
    #[arg(long = "data-dir", short = 'i', default_value = ".data")]
    data_dir: String,
    #[arg(long = "asset", short = 'a')]
    asset: String,
    #[arg(long = "date", short = 't', default_value = "2024-01-01")]
    date: String,
    // #[arg(long = "seed", short = 'r', default_value_t = 12345)]
    // seed: u64,
}

#[derive(Debug, Clone, PartialEq, Serialize, Deserialize, PartialOrd)]
pub struct Row {
    local_timestamp: i64,
    timestamp: i64,
    exchange: String,
    symbol: String,
    ask_amount: f64,
    ask_price: f64,
    bid_amount: f64,
    bid_price: f64,
}

impl Row {
    pub fn from_res<E>(r: Result<Row, E>) -> Option<Row> {
        r.ok()
    }

    pub fn into_event(
        self,
        instrument: Instrument,
        latency: u64,
    ) -> Option<MarketEvent<Instrument, OrderBookL1>> {
        Some(MarketEvent {
            exchange_time: DateTime::from_timestamp_micros(self.timestamp)?,
            received_time: DateTime::from_timestamp_micros(self.timestamp + latency as i64 * 1000)?,
            exchange: Exchange::from(self.exchange),
            instrument,
            kind: OrderBookL1 {
                last_update_time: DateTime::from_timestamp_micros(self.timestamp)?,
                best_bid: Level {
                    price: self.bid_price,
                    amount: self.bid_amount,
                },
                best_ask: Level {
                    price: self.ask_price,
                    amount: self.ask_amount,
                },
            },
        })
    }
}

fn main() -> Result<()> {
    let args = Args::parse();

    color_eyre::install()?;
    tracing_subscriber::fmt::init();

    let mut scanner = PerpArbScanner::new(
        args.entry_threshold,
        TimeDelta::milliseconds(args.execution_latency as i64),
        IndexMap::default(),
    );

    let mut sources = Vec::new();

    // let rng = Rc::new(RefCell::new(Xoshiro256PlusPlus::seed_from_u64(args.seed)));
    let from_res = Row::from_res;

    let bybit_path = format!(
        "{}/bybit/Futures/{}USDT/quotes/bybit_quotes_{}_{}USDT.csv.gz",
        args.data_dir, args.asset, args.date, args.asset
    );

    if let Ok(bybit_data) = File::open(&bybit_path) {
        sources.push(
            csv::Reader::from_reader(GzDecoder::new(BufReader::new(bybit_data)))
                .into_deserialize()
                .filter_map(from_res.clone()),
        );
    } else {
        warn!("No data found for Bybit at {bybit_path}");
    }

    let okex_path = format!(
        "{}/okex-swap/Futures/{}-USDT-SWAP/quotes/okex-swap_quotes_{}_{}-USDT-SWAP.csv.gz",
        args.data_dir, args.asset, args.date, args.asset
    );

    if let Ok(okex_data) = File::open(&okex_path) {
        sources.push(
            csv::Reader::from_reader(GzDecoder::new(BufReader::new(okex_data)))
                .into_deserialize()
                .filter_map(from_res.clone()),
        );
    } else {
        warn!("No data found for OKEx at {okex_path}");
    }

    let binance_path = format!(
        "{}/binance-futures/Futures/{}USDT/quotes/binance-futures_quotes_{}_{}USDT.csv.gz",
        args.data_dir, args.asset, args.date, args.asset
    );

    if let Ok(binance_data) = File::open(&binance_path) {
        sources.push(
            csv::Reader::from_reader(GzDecoder::new(BufReader::new(binance_data)))
                .into_deserialize()
                .filter_map(from_res.clone()),
        );
    } else {
        warn!("No data found for Binance at {binance_path}");
    }

    let gateio_path = format!(
        "{}/gate-io-futures/Futures/{}_USDT/quotes/gate-io-futures_quotes_{}_{}_USDT.csv.gz",
        args.data_dir, args.asset, args.date, args.asset
    );

    if let Ok(gateio_data) = File::open(&gateio_path) {
        sources.push(
            csv::Reader::from_reader(GzDecoder::new(BufReader::new(gateio_data)))
                .into_deserialize()
                .filter_map(from_res.clone()),
        );
    } else {
        warn!("No data found for Gate.io at {gateio_path}");
    }

    info!("deserializing {} sources", sources.len());

    let start = chrono::Utc::now();

    let asset = args.asset.clone();

    let to_event = move |row: Row| {
        row.into_event(
            Instrument {
                base: Symbol::from(asset.clone()),
                quote: Symbol::from("USDT"),
                kind: InstrumentKind::Perpetual,
            },
            args.data_latency,
        )
    };

    let data = itertools::kmerge(sources).filter_map(to_event);

    let mut count: usize = 0;

    for book in data {
        // if count % 10000 == 0 {
        //     info!("processed {} events; sample: {book:?}", count);
        // }
        scanner.handle_book(book);
        count += 1;
    }

    let end = chrono::Utc::now();

    let elapsed = end - start;

    info!(
        "processed {} events in {} ({} events/s)",
        count,
        elapsed,
        1000000.0 * count as f64 / elapsed.num_microseconds().unwrap() as f64
    );

    info!("scanner state : {:#?}", scanner);

    Ok(())
}
```
