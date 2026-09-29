# Prediction-driven Inventory Replenishment

## Executive summary

This project helps a distributor decide how much inventory to order and when to order it.

The data covers 50 products, 5 warehouses and 10 suppliers across 2024. The solution combines:

1. A demand forecast for the next 28 days.
2. A replenishment optimizer that considers supplier limits, warehouse capacity, lead times, order sizes and stockout costs.
3. A replay on later, unseen demand to compare the optimized policy with existing and greedy policies.

The main result is not simply a lower cost. The optimized plan reduces average inventory by about 42% while keeping a 99.3% fill rate and satisfying about 99.8% of checked capacity constraints. The current rule has a slightly lower calculated cost, but it holds much more stock and does not consistently respect supplier and warehouse limits.

## Business question

Can we reduce excess inventory without creating unacceptable stockouts or asking suppliers and warehouses to handle more than they can support?

## What the notebook does

- Checks the data and shows demand seasonality, promotion lift, warehouse stock cover and high-demand products.
- Adds business parameters to the same dataset, including holding cost, ordering cost, stockout penalty, supplier capacity, receiving capacity and storage capacity.
- Trains and compares Random Forest and LightGBM models against simple moving-average baselines.
- Uses only information available at the time of each forecast. The supplied `Demand_Forecast` column is excluded because it appears to contain information from actual demand.
- Uses the selected forecast in a PuLP/CBC mixed-integer replenishment model.
- Compares the MILP with the existing rule, a greedy policy and an LP relaxation with rounding.
- Tests all policies against actual demand from October 1 to December 23.
- Reports cost, fill rate, inventory, constraint satisfaction, solver gap, runtime, sensitivity, shadow prices and a supplier disruption scenario.

## Main results

| Policy | Total cost | Average stock | Fill rate |
|---|---:|---:|---:|
| Existing dataset rule | $132.4k | 92.5k units | 100.0% |
| Greedy forecast-driven | $134.4k | 54.1k units | 99.7% |
| LP plus rounding | $297.7k | 50.5k units | 94.6% |
| MILP | $145.0k | 53.9k units | 99.3% |

The MILP is not presented as a perfect answer. It is solving against forecasts, and demand rises in the evaluation period. The later MILP plans also reach the solver time limit, so their reported values are feasible solutions rather than proven global optima.

## Data note

The source file is a synthetic supply-chain dataset copied from the internet. It has no positive stockout flags and historical orders appear to arrive on the same day. Therefore, cost and capacity values added by the notebook are documented assumptions, and results should be treated as a methodology demonstration rather than a production recommendation.

## Files

- `Prediction_Optimization_Pijush_TrackC.ipynb`: complete analysis.
- `supply_chain_dataset1.csv`: original input data.
- `supply_chain_dataset1_enriched.csv`: input data plus the added business parameters.
- `results/`: policy comparison and solver gap tables after execution.
- `mlflow.db`: local experiment log.

<!-- ## How to run

1. Open `Prediction_Optimization_Pijush_TrackC.ipynb` in VS Code.
2. Select the notebook Python environment.
3. Run the cells from top to bottom.
4. To view experiment runs, use:

```text
mlflow ui --backend-store-uri sqlite:///mlflow.db
``` -->

## Important assumptions

- Holding cost is 25% of unit cost per year.
- Order fixed costs are simulated between $60 and $120 because the source data does not provide them.
- Supplier and warehouse capacities are calibrated from January through September demand.
- The evaluation period is kept separate from the calibration period.

## One-minute explanation

The forecast estimates demand. The optimizer then chooses orders while respecting real operating limits. The comparison shows the trade-off: less inventory and better capacity discipline require a small amount of extra ordering and stockout cost under the current assumptions. The next business step would be to replace the assumed costs and capacities with figures from the ERP, WMS and supplier contracts.
