# Sales Decision Support Dashboard

A Streamlit-based interactive dashboard for sales analysis and business decision support.

---

# ▶️ How to run

## Step 1 — Install Python
Make sure Python 3.10 or higher is installed:  
https://www.python.org/downloads/

## Step 2 — Install the required packages
Open your terminal (or Command Prompt / PowerShell on Windows) and run:

```bash
pip install -r requirements.txt

If pip does not work, try:

py -m pip install -r requirements.txt
Step 3 — Run the dashboard

Navigate to the folder where you saved dashboard.py and run:

py -m streamlit run dashboard.py

If that does not work, try:

python -m streamlit run dashboard.py

Your browser should automatically open at:

http://localhost:8501
📁 Project files

This project contains the following main files:

dashboard.py → the main Streamlit dashboard
requirements.txt → the required Python packages
README.md → explanation of the project and how to run it
sales.csv → example dataset, if included
📊 What the dashboard does

This dashboard helps businesses make better decisions based on sales data.

It includes:

KPI overview
Revenue analysis by weekday
Revenue over time
Product performance analysis
Top 5 strongest products
Top 5 weakest products
ABC analysis
Product portfolio analysis
Short-term preparation advice
Next week forecast
Scenario planning
Recommended business actions

The dashboard was designed as a reusable template, so users can upload their own CSV file and use the same dashboard logic.

📥 Input data

The dashboard expects a CSV file with sales data.

Recommended columns are:

date
time (optional)
product
category (optional)
quantity
revenue or total
unit_price (optional, only needed if revenue is missing)

If column names are different, the dashboard can often still work through:

automatic column detection
manual column mapping in the sidebar
