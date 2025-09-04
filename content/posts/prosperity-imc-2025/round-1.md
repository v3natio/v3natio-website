---
title: "Round 1: Market-Making"
date: 2025-09-12
draft: true
summary: "foo"
tags: ["Python", "CS", "Statistics", "Market-Making"]
---

### Preface

### Products

#### Rainforest Resin

Rainforest Resin was the simplest and most beginner-friendly product in Prosperity 3, perfectly suited to teach the fundamentals of [market making](https://www.investopedia.com/terms/m/marketmaker.asp). The product’s true price was permanently fixed at 10,000, meaning there were no intrinsic price movements to worry about. This setup clearly demonstrated the roles of [makers](https://www.cmegroup.com/education/courses/trading-and-analysis/market-makers-vs-market-takers.html#market-maker) and [takers](https://www.cmegroup.com/education/courses/trading-and-analysis/market-makers-vs-market-takers.html#market-taker): takers would cross the true price by either buying above 10,000 or selling below it, while makers posted passive orders hoping to be [matched](https://www.investopedia.com/terms/m/matchingorders.asp). The only thing that mattered for profitability here was the distance between the trade price and the true price — commonly referred to as the "edge." In short, the further you could buy below 10,000 or sell above 10,000, the better.

A key insight not just for Rainforest Resin but for all Prosperity products was understanding how the simulation handled order flow. At the start of every new timestep, the simulation first cleared all previous orders. Then, it sequentially processed new submissions: first some deep-liquidity makers, then occationally some takers, then our own bot’s actions (take or make), followed by other bots — usually more takers. This structure meant that speed and order cancellation were irrelevant: you had a full snapshot of the book and could submit any combination of passive or aggressive orders without racing against anyone. For Rainforest Resin, this confirmed that all focus should be on carefully optimizing the edge versus fill probability trade-off.

<table>
<tr valign="top">
<td width="100%" align="center">
  <strong>Figure 1: Rainforest Resin Orderbook over Time</strong>
</td>
</tr>

<tr valign="top">
<td width="100%" align="center">
  <img src="https://github.com/user-attachments/assets/54363d35-63ac-406f-b2de-ad6a06e7433d"
       alt="Dynamic dashboard"
       width="100%" />
</td>
</tr>

<tr valign="top">
<td width="100%" align="center">
  <em>Snippet of orderbook over time for Rainforest Resin.  
  Black stars are our quotes. Orange crosses are fills we got, profitable opportunities we immediately took, or trades at 10,000 we used to unwind inventory.</em>
</td>
</tr>
</table>

##### Final Strategy

Our final strategy for Rainforest Resin was straightforward. Each timestep, we first immediately took any favorable trades available — buying below 10,000 or selling above it. Afterward, we placed passive quotes slightly better than any existing liquidity (existing orders in orderbook): overbidding on [bids](https://www.investopedia.com/terms/a/bid.asp) and undercutting on [asks](https://www.investopedia.com/terms/a/ask.asp) while maintaining positive edge. If inventory became too skewed, we flattened it at exactly 10,000 to free up risk capacity for the next opportunities. No sophisticated logic or aggressiveness was needed due to the stable true price and the clean snapshot-based trading model.

Anyone could have come up with this approach by carefully studying the competition's matching rules and observing the environment during the tutorial round. Realizing that the true price was constant, fills were processed sequentially, and that orders only lived for one timestep simplified the problem dramatically. Having a basic visualization of price levels and logging fill quality would have made it even more obvious. Rainforest Resin alone consistently contributed around 39,000 SeaShells per round to our total PnL.

<br>

#### Kelp

Kelp was very similar in nature to Rainforest Resin, with the only major difference being that its price could move slightly from one timestep to the next. Instead of a fixed true price like Rainforest Resin, Kelp's true price followed a slow [random walk](https://www.investopedia.com/terms/r/randomwalktheory.asp). However, this movement was minor enough that the basic structure of the problem remained unchanged. Buyers and sellers still interacted as takers when crossing the fair price, and makers earned profits based on how far their trades deviated from the true price at the moment of execution.

The critical insight for Kelp was recognizing that, despite small movements, the future price was essentially unpredictable. Once teams realized that takers lacked predictive power and that the next true price could not be systematically forecasted, it became clear that the best available estimate for the true price was simply the current one. In fact, while there was a minor technical edge — stemming from the fact that the true price was internally a floating-point value and orders could only be posted at integer levels (creating slight mean-reversion tendencies after ticks) — this effect was too small to materially alter strategy. Just like with Rainforest Resin, the optimal approach was to treat the [WallMid](#what-is-wall-mid-and-why-did-we-use-it) as the fair price and quote around it.

<table>
<tr valign="top">
<td width="100%" align="center">
  <strong>Figure 2a: Kelp Orderbook over Time (Raw)</strong>
</td>
</tr>

<tr valign="top">
<td width="100%" align="center">
  <img src="https://github.com/user-attachments/assets/2a7c36dc-76b8-482d-934b-c9ee7ff527f6"
       alt="Dynamic dashboard"
       width="100%" />
</td>
</tr>

<tr valign="top">
<td width="100%" align="center">
  <em>Same as Figure 1, but showing Kelp's price movement over time.</em>
</td>
</tr>
</table>


<table>
<tr valign="top">
<td width="100%" align="center">
  <strong>Figure 2b: Kelp Orderbook over Time (Normalized)</strong>
</td>
</tr>

<tr valign="top">
<td width="100%" align="center">
  <img src="https://github.com/user-attachments/assets/5b7828a5-df9b-44ae-ab11-6461ee026a51"
       alt="Static, normalized dashboard"
       width="100%" />
</td>
</tr>

<tr valign="top">
<td width="100%" align="center">
  <em>Same as Figure 2a, but with prices normalized by the Wall Mid indicator to make the series stationary.  
  Notice how it resembles Rainforest Resin, but with a tighter bid-ask spread.</em>
</td>
</tr>
</table>

##### Final Strategy

Our final strategy for Kelp was nearly identical to that for Rainforest Resin. At each timestep, we first immediately took any favorable trades available relative to the current wall mid, then placed slightly improved passive orders (overbidding and undercutting) around the fair price. If inventory became too large, we neutralized it by trading at zero edge relative to the current price estimate. No major changes were needed compared to the first product.

Teams that approached Kelp correctly would have first verified whether takers or the market exhibited any predictability, either through simple empirical analysis or by observing that naive strategies (like quoting around the current price) worked well. Realizing that there was no meaningful adverse selection risk meant that treating Kelp identically to Rainforest Resin was the optimal path. On average, Kelp generated around 5,000 SeaShells per round, primarily limited by the tighter spreads compared to the first product.

<br>

#### Squid Ink

Squid Ink differed from the previous two products mainly in that it had a tighter bid-ask spread relative to its average movement, combined with occasional sharp price jumps. This made pure market-making less attractive, not because of systematic losses, but because it introduced higher variance in realized PnL. In other words, fills could swing more widely in value depending on unpredictable price jumps, even if there was no predictable [adverse selection](https://www.investopedia.com/terms/a/adverseselection.asp) in the classic sense. Officially, the product was described as mean-reverting in the short term, suggesting that mean-reversion strategies might work. However, after investigating the market dynamics more carefully, we discovered an entirely different and more reliable opportunity.

Our main insight was that one of the anonymous bot traders consistently exhibited a strikingly predictable pattern: buying 15 lots at the daily low and selling 15 lots at the daily high. We observed this behavior early on, without initially knowing who the trader was. It was only in the final round — when trader IDs were temporarily visible — that we learned this trader was named Olivia. Anticipating this kind of behavior and designing logic to detect it gave us a clear edge. Without revealing our exact identification method (to avoid encouraging blind copying), the general approach involved tracking the daily running minimum and maximum. When a trade occurred at a daily extreme — and in the expected direction relative to the mid price — we flagged it as a signal and positioned accordingly. False positives were managed by monitoring for corresponding new extrema that contradicted earlier signals.

<table>
<tr valign="top">
<td width="100%" align="center">
  <strong>Figure 3a: Squid Ink Prices with Informed Trader</strong>
</td>
</tr>

<tr valign="top">
<td width="100%" align="center">
  <img src="https://github.com/user-attachments/assets/9f552b07-98e9-4488-b4b9-95b2e1435747"
       alt="Dynamic dashboard"
       width="100%" />
</td>
</tr>

<tr valign="top">
<td width="100%" align="center">
  <em>This plot shows that Olivia bought exactly at the daily minimum and sold exactly at the daily maximum.</em>
</td>
</tr>
</table>

<table>
<tr valign="top">
<td width="100%" align="center">
  <strong>Figure 3b: Squid Ink Prices with Anonymous Trades</strong>
</td>
</tr>

<tr valign="top">
<td width="100%" align="center">
  <img src="https://github.com/user-attachments/assets/b6e23225-fd1f-4971-ad00-729ec2bdef8f"
       alt="Static, normalized dashboard"
       width="100%" />
</td>
</tr>

<tr valign="top">
<td width="100%" align="center">
  <em>This plot filters all anonymous trades to only show trades with quantity = 15, as it appeared during early rounds.  
  Careful teams could have spotted this pattern and identified Olivia's behavior during Rounds 1–4.</em>
</td>
</tr>
</table>

##### Final Strategy

Our final strategy for Squid Ink focused purely on following this daily-extrema trading behavior, dynamically updating our positions based on detected trades and resetting when invalidations occurred. No active market making or mean reversion trading was used for this product. The result was a low-risk, high-reliability PnL contributor that did not rely on predicting price moves directly.

Anyone who carefully analyzed historical Prosperity 2 data or public write-ups — such as [Stanford Cardinal’s](https://github.com/ShubhamAnandJain/IMC-Prosperity-2023-Stanford-Cardinal) or [Jasper's](https://github.com/jmerle/imc-prosperity-2) — could have anticipated similar behaviors and prepared detection logic in advance. We also discovered and executed this strategy on another product in Prosperity 2 without having participated in Prosperity 1. Early identification of this behavior consistently netted us on average 8,000 SeaShells per round, providing a stable and important edge in Round 1.

<br>
