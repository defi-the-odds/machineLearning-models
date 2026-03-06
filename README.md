# Machine Learning Model Transparency & Guide

This repository is dedicated to providing transparency around the accuracy and performance of machine learning models used for volatility and regime prediction in cryptocurrency markets. Here, you will find:

- **Model Metrics:** Detailed accuracy statistics, including RMSE, R², classification accuracy, and ROC-AUC for each model and asset.
- **Output Guides:** Clear explanations on how to interpret the predictions from each model, including volatility forecasts, expansion probabilities, and regime scores.
- **Usage Guidance:** Best practices for using model outputs in trading, risk management, and decision-making.

## Purpose

The goal of this repo is to:
- Enable users, researchers, and stakeholders to understand the strengths and limitations of each model.
- Provide confidence in the model results by sharing performance metrics and validation methods.
- Help users make informed decisions by explaining what each output means and how it can be applied.

## How to Read Model Outputs

- **Volatility Regressor:** Predicts future volatility levels. Higher values indicate more expected price movement; lower values suggest calm, range-bound markets.
- **Expansion Classifier:** Outputs the probability that volatility will expand. Values closer to 1.0 signal a high likelihood of a volatility spike; values near 0.0 suggest contraction.
- **Regime Model:** Provides a regime score indicating trend strength and direction. High scores favor trend trades; mid scores suggest range-bound conditions.

## Transparency Commitment

All model metrics are published for each asset and timeframe, allowing users to track performance over time and across market conditions. This ensures that model limitations and strengths are clear, and that users can trust the outputs for their intended purposes.

For more details, see the MODEL_GUIDE.md and metrics CSV files in the repo.
