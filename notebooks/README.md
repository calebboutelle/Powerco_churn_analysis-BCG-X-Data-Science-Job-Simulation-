# PowerCo Customer Churn Analysis

Independent analysis completed as part of the **BCG X Data Science Job Simulation on Forage**.
PowerCo is a simulated client case. This project is not BCG client work.

## Executive Summary

This project examines whether customer price sensitivity is a meaningful driver of churn for PowerCo.

The analysis found that annual off-peak price changes had almost no direct relationship with churn, while broader customer characteristics such as economics, consumption patterns, tenure, and pricing context provided a stronger overall risk picture.

A Random Forest classifier achieved a ROC-AUC of **0.715**, indicating meaningful ability to rank churn risk, but recall was only **8.45%**, so the model is not yet suitable for automated retention decisions.

## Business Question

Is customer price sensitivity a significant driver of churn, and what customer characteristics could help PowerCo identify customers at higher risk of leaving?

## Key Findings

- Price sensitivity alone does not explain customer churn.
- January-to-December off-peak price change had near-zero correlation with churn.
- Broader customer economics, consumption behavior, and tenure showed larger signals.
- The Random Forest model contained meaningful predictive signal but missed most actual churners.
- Further validation is needed before using the model operationally.

## Deliverables

- Business presentation
- Python analysis notebook
- Supporting visualizations

## Tools
Python, Pandas, NumPy, Matplotlib, Seaborn, SciPy, Scikit-learn, Google Colab
