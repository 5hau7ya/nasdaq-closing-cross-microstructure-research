# Predicting Nasdaq Closing Auction Price Movements: A Market Microstructure Study of Order Imbalance, Liquidity and Temporal Dynamics

## Introduction

The main question that I wish to answer is as follows:
- What information in the final 10 minutes of the Nasdaq closing auction actually predicts short-horizon relative price movements and how does that information change as the auction approaches the close

For full explanation, results and analysis please refer to the [report.pdf](report.pdf)
For the complete code and output log please refer to the [kaggle run](https://www.kaggle.com/code/shaury4/research-notebook)

The data is from the [Optiver - Trading at the Close](https://www.kaggle.com/competitions/optiver-trading-at-the-close/overview) Kaggle competition (200 stocks, 481 days, about 5.2 million rows). The target is the 60 second future move in the WAP of a stock minus the same move of a synthetic index, in basis points. The data is split by date so that future information does not get into the training. We use Ridge regression and LightGBM.

The project is based on the following hypotheses:

- H1 - Auction imbalance carries predictive information about 60-second forward relative returns.
- H2 - Informativeness is not constant, it changes as the close approaches.
- H3 - Cross-sectional and within stock normalization improves generalization relative to raw units.
- H4 - Temporal dynamics (ex: Δimbalance) provide greater predictive power than absolute levels.
- H5 - Predictability is dependent upon market regimes such as volatility, liquidity, and imbalance magnitude.

## Results

### H1 - Auction imbalance predicts the 60 second forward return

**Result: Yes**

- More unmatched buy side interest in the auction is followed by higher future returns and more unmatched sell side interest by lower returns. The effect is small (under 1 basis point) but consistent
- Auction imbalance is the second most important group of features in the model (about 15% of its importance). The ratio of matched to unmatched size shows no relation with the target
- The order book imbalance looks even stronger, but this is mostly the WAP effect. Market urgency (the most important feature) is the same as twice the gap between WAP and the mid price. This gap has a correlation of -0.17 with the target but only +0.03 with the future change in the mid price. So it works because WAP moves back towards the mid price and not because it shows where the real price is going

### H2 - Informativeness changes as the close approaches

**Result: Yes**

- Auction imbalance becomes more important as the close approaches. Its share of the model's importance doubles from 6.6% to 13% between the start and the end of the window, and from 300 seconds onwards its correlation with the target is about 3.4 times stronger than the average
- Order book signals (liquidity imbalance and market urgency) are the strongest at the start of the window and fade as the close approaches (the share of market urgency falls from 41% to 22%). So the information moves from the order book to the auction
- The change is not a straight line. There is a jump at 300 seconds when Nasdaq starts to update the auction information every second. The model finds this 300 second split on its own from the seconds in the auction
- The effect of imbalance depends on time in a non-linear way, which LightGBM captures better than a simple signal * time term

### H3 - Normalization improves generalization

**Result: Cross-sectional normalization - Yes. Within stock normalization - No**

- Standardizing the raw features improves the Ridge model (MAE 6.467 to 6.441)
- Features relative to other stocks (cross-sectional) work well. They give the lowest RMSE and the highest directional accuracy in the Ridge model, and the cross-sectional imbalance is as important as the raw imbalance in LightGBM
- Features relative to the stock's own history give no gain. Their MAE is the same as predicting zero

### H4 - Temporal dynamics are more predictive than levels

**Result: Yes, but the gain is small**

- The change in imbalance is a stronger signal than the level of imbalance (Spearman correlation 0.031 vs 0.022) and it adds to the level when both are used
- The recent move in WAP gives the biggest gain and it is negative. If WAP went up in the last 10 seconds it tends to come down, most likely because of the same WAP effect. Volatility and the second order change in imbalance are weak (volatility tells how much the price moves and not in which direction)
- All the temporal features together improve the correlation from 0.014 to 0.049 and the MAE only from 6.486 to 6.476, and they mostly carry separate information

### H5 - Predictability depends on the market regime

**Result: No. The stock matters more than the regime, H5 is only weakly supported**

- The model works about the same in all the regimes. The correlation stays between 0.165 and 0.187 in all the volatility, liquidity and imbalance magnitude groups
- Volatile auctions are slightly better, the least liquid auctions are slightly worse, and the size of the imbalance makes no difference. These differences are too small to matter
- The model works for almost all the stocks (correlation is positive for 99%) and the difference between stocks is stable over time
- Wide spread stocks are the most predictable (correlation 0.13 for the narrowest third vs 0.21 for the widest third), since the WAP effect is bigger when the spread is wider. Volatile stocks look more predictable only because they have wide spreads. Stocks that usually have large imbalances are less predictable and matched size shows no relation
- The order book information matters more for low liquidity stocks, while auction imbalance matters about the same for all stocks

TO BE CONTINUED...
