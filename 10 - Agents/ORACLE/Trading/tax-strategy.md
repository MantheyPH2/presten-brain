---
tags: [oracle, trading, tax, strategy, section-475, wash-sale]
created: 2026-04-21
---

# Tax Strategy

> The government is a silent partner in every trade. Understanding the rules means keeping more of what you make — and using losses intelligently instead of letting them evaporate.

---

## The Core Tax Framework for Traders

All trading gains fall into one of two buckets. Which bucket determines how much you keep.

### Short-Term vs. Long-Term Capital Gains

| Holding Period | Tax Treatment | Effective Rate |
|---------------|--------------|---------------|
| < 1 year | Ordinary income (same as salary) | Up to 37% |
| ≥ 1 year (366+ days) | Long-term capital gains | 0%, 15%, or 20% depending on income |

**The math:** On a $100,000 gain, the difference between short-term (37%) and long-term (20%) treatment is $17,000 in taxes. Holding a winner for one more day — the 366th day — saves $17,000.

**Net Investment Income Tax (NIIT):** Add an additional 3.8% surcharge on investment income above $200K single / $250K married. Effective top rate on short-term gains is ~40.8%. Long-term can reach ~23.8%.

**Strategy:** For positions with large unrealized gains, if you're in week 11 of holding, seriously evaluate whether waiting another month to cross the 1-year mark is worth it vs. the risk of giving the gain back.

---

## The Wash Sale Rule

### What It Is

IRC Section 1091 disallows a capital loss if you purchase a "substantially identical" security within 30 days *before or after* the sale at a loss. The 30-day window extends in both directions.

**Example:**
- You sell NVDA at a $5,000 loss on March 1
- You buy NVDA back on March 15 (14 days later, within the 30-day window)
- The $5,000 loss is **disallowed** — you cannot claim it
- The disallowed loss is added to the cost basis of your new NVDA shares (deferred, not destroyed)

### What Triggers the Wash Sale

- Selling a stock at a loss and buying the **same stock** within the window
- Selling a stock at a loss and buying a **call option** on the same stock within the window (options count)
- Selling an option at a loss and buying **substantially identical** options within the window
- Selling in a taxable account at a loss while **simultaneously holding in an IRA** — the IRA purchase triggers wash sale, and the loss is *permanently* lost (not just deferred)

### What Does NOT Trigger the Wash Sale

- Selling a stock at a loss and buying a **different but correlated** stock (e.g., sell NVDA, buy AMD — not identical)
- Selling an ETF at a loss and buying a similar but not identical ETF (e.g., sell SPY, buy IVV — different fund, similar exposure — this is a gray area; sell SPY, buy VTI is cleaner)
- Crypto (currently) — see crypto section below

### Wash Sale + Options

The wash sale applies to options under IRC §1091(c), which includes "contracts or options to acquire or sell stock or securities."

**Dangerous scenario:**
- You sell a call option on AAPL at a loss
- You buy another AAPL call option within 30 days
- If the options are "substantially identical" (same or similar strike/expiry), wash sale applies

**Safer practice:** When harvesting an options loss, don't re-enter the identical strike/expiry for 30+ days. You can re-enter with a meaningfully different strike or expiry after the window.

---

## Section 475 Mark-to-Market (The Trader's Nuclear Option)

### What It Is

Traders who qualify for **Trader Tax Status (TTS)** can elect Section 475(f) Mark-to-Market (MTM) accounting. This fundamentally changes how your trading activity is taxed.

**Under Section 475:**
- All open positions are treated as *sold at fair market value* on December 31 each year
- Gains and losses are treated as **ordinary income/losses** (not capital)
- The $3,000 annual cap on net capital losses is **eliminated**
- The wash sale rule is **completely eliminated**
- Losses reported on Form 4797 (ordinary), not Schedule D (capital)

### Who Qualifies for Trader Tax Status

The IRS requires *all* of the following:
1. **Seek to profit from daily market movements** (not dividends, interest, or long-term appreciation)
2. **Trade with continuity and regularity** — substantial activity throughout the year
3. **Trading is your primary business or a significant part of your economic activity**

General thresholds (not IRS-official, but practical):
- 720+ trades per year
- Trading on 75%+ of available market days
- Holding periods predominantly short-term (days, not months)
- Substantial capital involved relative to your overall financial picture

**Important:** The IRS looks at the full picture. A few hundred trades with casual involvement likely won't qualify. A few thousand trades with consistent daily activity almost certainly will.

### The 2026 Election Process

- **Deadline:** April 15, 2026 — file a Section 475 election statement with your 2025 tax return (or extension)
- **Step 2:** File Form 3115 with your 2026 tax return (filed in 2027)
- **New Rule (Rev. Proc. 2025-23):** Once you elect Section 475, revoking it requires IRS consent and is now locked for **5 years**. This is a major commitment — only elect if you intend to trade professionally long-term.

### The Trade-Off

| Factor | With Section 475 | Without Section 475 |
|--------|-----------------|-------------------|
| Losses | Fully deductible as ordinary loss | Capped at $3K/year above gains |
| Wash sale rule | Eliminated | Applies |
| Long-term gains rates | Forfeited (everything ordinary) | Available for positions held 1+ year |
| Business expenses | Deductible (home office, data, software) | Limited |

**Bottom line:** Section 475 is best for active traders who primarily have short-term positions and who take frequent losses that need full deductibility. It's harmful if you have large long-term winners — you'd pay 37% instead of 20% on those gains.

---

## Crypto-Specific Tax Rules (2026)

### Current Status

As of 2026, crypto is classified as **property** by the IRS (not a security). This has major implications:

- Every crypto-to-crypto swap is a **taxable event** (you calculate gain/loss on each trade)
- Selling crypto for fiat is a taxable event
- Using crypto to buy goods/services is a taxable event
- Receiving crypto as payment is ordinary income at fair market value on receipt date

### Wash Sale Rule Does NOT Apply to Crypto

Currently, because crypto is property (not a security), IRC Section 1091 does not apply. This means:
- You can sell Bitcoin at a loss today and buy it back tomorrow — the loss is still valid
- This allows **year-round tax loss harvesting** in crypto with no 30-day waiting period

**Critical caveat:** Congress has proposed applying the wash sale rule to crypto multiple times. This loophole is likely to close eventually. Use it aggressively while it exists, but do not build a long-term strategy dependent on it remaining in place.

### Bitcoin ETF Exception

Spot Bitcoin ETFs (IBIT, FBTC, etc.) ARE securities, not property. The wash sale rule **does apply** to Bitcoin ETF shares. Selling a Bitcoin ETF at a loss and buying it back within 30 days will trigger a wash sale.

### DeFi, Staking, and Other Crypto Events

| Event | Tax Treatment |
|-------|--------------|
| Staking rewards received | Ordinary income at FMV on date received |
| Lending interest received | Ordinary income |
| LP position entry | Not a taxable event at entry |
| LP position exit | Taxable event — calculate gain/loss on each token |
| Airdrop received | Ordinary income at FMV on date received |
| Fork tokens received | Ordinary income at FMV when you have dominion |
| Swap on DEX (e.g., ETH → USDC) | Taxable event — capital gain/loss |

**Record-keeping imperative:** Every single DeFi transaction must be logged with date, amount, and USD value at time of transaction. Use Koinly, CoinTracker, or similar tools. Reconstructing DeFi history retroactively is extraordinarily painful.

---

## Tax-Efficient Trading Strategies

### 1. Strategic Holding for Long-Term Rates

When a position shows a large gain approaching the 1-year mark, model the break-even:

```
Tax savings from holding to long-term = (Short-term rate - Long-term rate) × Gain
Risk of holding = Potential loss × Probability of decline

If tax savings > expected loss from holding: hold to long-term
```

Example: $50,000 unrealized gain at day 355. Holding 11 more days saves ~$8,500 in taxes (17% difference). If you think the probability-adjusted risk of losing that $8,500 or more in 11 days is low — hold.

### 2. Tax-Loss Harvesting

**Mechanics:**
- Identify positions with unrealized losses in October/November
- Sell them to realize the loss before year-end
- Reinvest in a correlated but not substantially identical security (to maintain market exposure)
- Wait 31 days, then switch back if desired

**Harvest calendar:**
- October–November: Review all positions for harvest opportunities
- Before December 31: Execute all planned harvests (must be settled, not just executed)
- January 31+: Can re-enter harvested positions after 31-day window

**Pairing gains with losses:**
- If you have $30,000 in short-term gains, you need $30,000 in harvested losses to offset
- Prioritize harvesting **short-term losses** first (they offset short-term gains taxed at higher rates)
- Long-term losses offset long-term gains first under IRS netting rules

### 3. Specific Lot Identification

When you've bought the same stock at different times and prices, you can choose **which lot to sell** for maximum tax benefit.

- **Highest cost basis first** = smallest taxable gain (or largest deductible loss)
- **Specific ID method** = you tell your broker exactly which shares to sell
- Must elect specific ID *before* the sale, not after
- Default IRS method is FIFO (first in, first out) — almost never tax-optimal for active traders

**Setup:** Confirm with your broker that you're on Specific ID method for every account. This is a one-time setup per account.

### 4. Qualified Opportunity Zones

For very large capital gains (six figures+), investing proceeds into a Qualified Opportunity Zone (QOZ) fund can defer and partially reduce the capital gains tax. This is a complex strategy requiring a CPA — the summary:

- Defer gain recognition until 2026 (the deferral cliff year) or until fund exit
- Eliminate capital gains on the QOZ *appreciation* itself after 10-year hold
- Best for: traders with one very large gain who have time horizon flexibility

### 5. Retirement Account Trading

Options trading inside an IRA:
- Gains are tax-deferred (Traditional IRA) or tax-free (Roth IRA)
- No wash sale issues within the same IRA
- **DANGER:** wash sales triggered by IRA purchases when you sell at a loss in a taxable account — those losses are **permanently** lost (not deferred)
- Never buy back in an IRA what you sold at a loss in a taxable account within the 30-day window

---

## Account Size × Tax Priority Matrix

| Account Size | Primary Focus | Tax Priority |
|-------------|--------------|-------------|
| Under $50K | Making money, improving skill | Minimal — focus on trading, not taxes |
| $50K–$100K | Consistent returns | Start tracking wash sales; think about holding periods on big winners |
| $100K–$500K | Capital preservation + growth | Active tax-loss harvesting, holding period management, consider CPA |
| $500K+ | Tax-efficient compounding | Full strategy: lot ID, MTM election analysis, QOZs, CPA essential |

---

## ORACLE's Tax Record-Keeping Protocol

Every trade in ORACLE's journal serves as the tax record. Required fields:

```
Trade Log Entry (Tax-Required Fields):
- Ticker / instrument
- Open date and time
- Close date and time  
- Number of shares / contracts
- Entry price (per share/contract)
- Exit price (per share/contract)
- Gross gain/loss in USD
- Short-term or long-term (holding period)
- Lot identification (which specific shares)
- Wash sale flag (Y/N — did this trigger or get triggered by a wash sale?)
- Notes (strategy, reason for trade)
```

**Export annually:** Run a complete trade history export from every broker before February 15 to give to your CPA or software. Brokers issue Form 1099-B by February 15.

**Platform reconciliation:** Always reconcile your personal records against broker 1099-B. Brokers frequently make cost basis errors (especially on options, stock splits, and wash sales). Your records are the ground truth.

---

## Quick Tax Reference Card

```
HOLDING PERIOD CHECK
< 1 year = ordinary income (up to 40.8% effective)
≥ 366 days = long-term rates (0%, 15%, or 20% + 3.8% NIIT)

WASH SALE CHECK
Same or substantially identical security?
Within 30 days before OR after the loss sale?
Yes to both = WASH SALE — loss disallowed, added to new basis

CRYPTO WASH SALE
Does NOT apply (property, not security)
Bitcoin ETFs = DOES apply (they are securities)

SECTION 475 ELECTION
Deadline: April 15 of the election year
Benefit: unlimited loss deduction, no wash sales
Trade-off: lose long-term capital gains rates forever
Lock-in: 5 years minimum under 2025 IRS rules
```
