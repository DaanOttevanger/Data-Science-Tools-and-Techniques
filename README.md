# Data-Science-Tools-and-Techniques
# Sales Decision Support Dashboard
#
# For this individual project, I developed a dynamic dashboard that helps businesses
# make decisions based on sales data.
#
# At the start of the project, I first thought about building the dashboard for a
# snack bar or small canteen. Later, I decided to broaden the concept because the
# dataset I used looked more like a general retail or store dataset. That made the
# dashboard more flexible and gave me more possibilities.
#
# The goal of the dashboard is not only to visualize sales data, but also to translate
# that data into practical business actions.
#
# The dashboard focuses on:
# - understanding sales performance,
# - identifying strong and weak products,
# - supporting operational decisions for today,
# - supporting planning decisions for next week,
# - and suggesting business actions based on the data.

# Why I chose this project
#
# I chose this project because I wanted to build something practical and interactive.
# I did not want to make a project that only contained static analysis or a few graphs.
# Instead, I wanted to create something that feels more like a real business tool.
#
# This project combines several important parts of data-driven decision making:
# - data cleaning,
# - data analysis,
# - visualization,
# - business interpretation,
# - and application development.

# Inspiration from the selected videos
#
# The most important videos for me were:
# - AI Python for Beginners
# - Effortless Data Analysis and Cleaning with Data Wrangler in VS Code
# - Become a Data Storyteller with Streamlit
#
# The Python video helped me with the technical foundation.
# The Data Wrangler video was useful because data cleaning turned out to be a major part
# of the project.
# The Streamlit video inspired me to turn the analysis into an actual dashboard.

# Method and approach
#
# I worked in a way that is similar to CRISP-DM:
# 1. understand the business problem,
# 2. explore the dataset,
# 3. prepare the data,
# 4. build the analysis and dashboard,
# 5. improve the layout and decision support features.
#
# I built the dashboard step by step. I started with simple analysis and later added:
# - dynamic filters,
# - forecasts,
# - scenario planning,
# - ABC analysis,
# - product portfolio analysis,
# - and stock suggestions.

import streamlit as st
import pandas as pd
import numpy as np
import altair as alt

# Layout and styling
#
# I added simple styling to improve readability and make the dashboard look more professional.

st.set_page_config(
    page_title="Sales Decision Support Dashboard",
    layout="wide"
)

st.markdown("""
<style>
.block-container {
    padding-top: 1.2rem;
    padding-bottom: 1.2rem;
}
h1, h2, h3 {
    letter-spacing: -0.02em;
}
[data-testid="stMetricValue"] {
    font-size: 1.6rem;
}
.small-note {
    color: #9aa0a6;
    font-size: 0.9rem;
}
</style>
""", unsafe_allow_html=True)

# Helper functions
#
# To keep the code structured and readable, I created multiple helper functions.
# These functions make the dashboard easier to maintain and make it clearer why certain
# steps were taken.

def normalize_columns(df: pd.DataFrame) -> pd.DataFrame:
    # Standardize column names so the dashboard can work more easily with different datasets.
    df = df.copy()
    df.columns = (
        df.columns.astype(str)
        .str.strip()
        .str.lower()
        .str.replace(" ", "_", regex=False)
        .str.replace("-", "_", regex=False)
        .str.replace("/", "_", regex=False)
    )
    return df


def pretty_label(text):
    # Convert technical labels such as product_line into Product Line.
    return str(text).replace("_", " ").title()


def prettify_dataframe(df: pd.DataFrame) -> pd.DataFrame:
    # Apply cleaner labels to dataframes shown in the dashboard.
    return df.rename(columns=lambda x: pretty_label(x))


def make_ranked(df: pd.DataFrame) -> pd.DataFrame:
    # Add a rank column so tables start at 1 instead of 0.
    ranked = df.reset_index(drop=True).copy()
    ranked.insert(0, "Rank", range(1, len(ranked) + 1))
    return ranked


def detect_columns(df: pd.DataFrame):
    # Detect useful columns automatically so the dashboard becomes more reusable.
    cols = df.columns.tolist()

    date_candidates = ["date", "transaction_date", "order_date", "invoice_date"]
    time_candidates = ["time", "transaction_time", "order_time"]
    product_candidates = ["product", "product_line", "item", "product_name", "description"]
    category_candidates = ["category", "product_category", "product_line", "group"]
    quantity_candidates = ["quantity", "qty", "units_sold"]
    revenue_candidates = ["revenue", "total", "sales", "total_amount", "amount"]
    unit_price_candidates = ["unit_price", "price", "unitcost", "unit_cost"]

    def find_first(candidates):
        for c in candidates:
            if c in cols:
                return c
        return None

    return {
        "date_col": find_first(date_candidates),
        "time_col": find_first(time_candidates),
        "product_col": find_first(product_candidates),
        "category_col": find_first(category_candidates),
        "quantity_col": find_first(quantity_candidates),
        "revenue_col": find_first(revenue_candidates),
        "unit_price_col": find_first(unit_price_candidates),
    }


def build_template_csv():
    # Create an example template file so another user can understand what input is expected.
    template_df = pd.DataFrame({
        "date": ["2026-05-01", "2026-05-01", "2026-05-02"],
        "time": ["12:00", "13:15", "17:30"],
        "product": ["Fries", "Cola", "Burger"],
        "category": ["Food", "Drinks", "Food"],
        "quantity": [3, 2, 4],
        "revenue": [9.00, 5.00, 24.00],
        "unit_price": [3.00, 2.50, 6.00]
    })
    return template_df.to_csv(index=False).encode("utf-8")


def manual_column_mapper(df: pd.DataFrame, auto_map: dict):
    # Manual fallback in case automatic detection is not correct.
    st.sidebar.markdown("### Manual Column Mapping")
    st.sidebar.caption("Use this if automatic detection is incorrect.")

    all_columns = ["None"] + df.columns.tolist()

    def idx(value):
        return all_columns.index(value) if value in all_columns else 0

    date_col = st.sidebar.selectbox("Date Column", all_columns, index=idx(auto_map.get("date_col")))
    time_col = st.sidebar.selectbox("Time Column (Optional)", all_columns, index=idx(auto_map.get("time_col")))
    product_col = st.sidebar.selectbox("Product Column", all_columns, index=idx(auto_map.get("product_col")))
    category_col = st.sidebar.selectbox("Category Column (Optional)", all_columns, index=idx(auto_map.get("category_col")))
    quantity_col = st.sidebar.selectbox("Quantity Column", all_columns, index=idx(auto_map.get("quantity_col")))
    revenue_col = st.sidebar.selectbox("Revenue / Total Column", all_columns, index=idx(auto_map.get("revenue_col")))
    unit_price_col = st.sidebar.selectbox("Unit Price Column (Optional)", all_columns, index=idx(auto_map.get("unit_price_col")))

    return {
        "date_col": None if date_col == "None" else date_col,
        "time_col": None if time_col == "None" else time_col,
        "product_col": None if product_col == "None" else product_col,
        "category_col": None if category_col == "None" else category_col,
        "quantity_col": None if quantity_col == "None" else quantity_col,
        "revenue_col": None if revenue_col == "None" else revenue_col,
        "unit_price_col": None if unit_price_col == "None" else unit_price_col,
    }


def prepare_data_auto(df: pd.DataFrame):
    # Automatically prepare the dataset.
    df = normalize_columns(df)
    detected = detect_columns(df)

    date_col = detected["date_col"]
    time_col = detected["time_col"]
    product_col = detected["product_col"]
    category_col = detected["category_col"]
    quantity_col = detected["quantity_col"]
    revenue_col = detected["revenue_col"]
    unit_price_col = detected["unit_price_col"]

    missing = []
    if date_col is None:
        missing.append("date")
    if product_col is None:
        missing.append("product")
    if quantity_col is None:
        missing.append("quantity")

    if missing:
        return None, detected, f"Missing required column(s): {', '.join(missing)}"

    if revenue_col is None and unit_price_col is not None:
        df["revenue"] = pd.to_numeric(df[quantity_col], errors="coerce") * pd.to_numeric(df[unit_price_col], errors="coerce")
        revenue_col = "revenue"
        detected["revenue_col"] = revenue_col

    if revenue_col is None:
        return None, detected, "No revenue/total column found, and no unit price column available to calculate revenue."

    df[date_col] = pd.to_datetime(df[date_col], errors="coerce")
    df = df.dropna(subset=[date_col])

    df[quantity_col] = pd.to_numeric(df[quantity_col], errors="coerce")
    df[revenue_col] = pd.to_numeric(df[revenue_col], errors="coerce")
    df = df.dropna(subset=[quantity_col, revenue_col])

    if time_col is not None:
        parsed_time = pd.to_datetime(df[time_col], errors="coerce")
        df["hour"] = parsed_time.dt.hour
    else:
        df["hour"] = np.nan

    df["date_only"] = df[date_col].dt.date
    df["day_name"] = df[date_col].dt.day_name()
    df["day_num"] = df[date_col].dt.dayofweek
    df["week"] = df[date_col].dt.isocalendar().week.astype(int)
    df["month"] = df[date_col].dt.month_name()
    df["year"] = df[date_col].dt.year

    if product_col is not None:
        df[product_col] = df[product_col].astype(str).str.strip()

    if category_col is not None:
        df[category_col] = df[category_col].astype(str).str.strip()

    return df, detected, None


def prepare_data_manual(df: pd.DataFrame, mapping: dict):
    # Manual version of the data preparation.
    df = normalize_columns(df)

    date_col = mapping["date_col"]
    time_col = mapping["time_col"]
    product_col = mapping["product_col"]
    category_col = mapping["category_col"]
    quantity_col = mapping["quantity_col"]
    revenue_col = mapping["revenue_col"]
    unit_price_col = mapping["unit_price_col"]

    if revenue_col is None and quantity_col is not None and unit_price_col is not None:
        df["revenue"] = pd.to_numeric(df[quantity_col], errors="coerce") * pd.to_numeric(df[unit_price_col], errors="coerce")
        revenue_col = "revenue"
        mapping["revenue_col"] = revenue_col

    missing = []
    if date_col is None:
        missing.append("date")
    if product_col is None:
        missing.append("product")
    if quantity_col is None:
        missing.append("quantity")
    if revenue_col is None:
        missing.append("revenue")

    if missing:
        return None, mapping, f"Missing required columns after manual mapping: {', '.join(missing)}"

    df[date_col] = pd.to_datetime(df[date_col], errors="coerce")
    df[quantity_col] = pd.to_numeric(df[quantity_col], errors="coerce")
    df[revenue_col] = pd.to_numeric(df[revenue_col], errors="coerce")
    df = df.dropna(subset=[date_col, quantity_col, revenue_col])

    if time_col is not None:
        parsed_time = pd.to_datetime(df[time_col], errors="coerce")
        df["hour"] = parsed_time.dt.hour
    else:
        df["hour"] = np.nan

    df["date_only"] = df[date_col].dt.date
    df["day_name"] = df[date_col].dt.day_name()
    df["day_num"] = df[date_col].dt.dayofweek
    df["week"] = df[date_col].dt.isocalendar().week.astype(int)
    df["month"] = df[date_col].dt.month_name()
    df["year"] = df[date_col].dt.year

    df[product_col] = df[product_col].astype(str).str.strip()
    if category_col is not None:
        df[category_col] = df[category_col].astype(str).str.strip()

    return df, mapping, None


def fmt_currency(x):
    return f"€{x:,.2f}"


def compute_product_summary(df, product_col, revenue_col, quantity_col):
    # Build the product performance summary.
    return (
        df.groupby(product_col)
        .agg(
            total_quantity=(quantity_col, "sum"),
            total_revenue=(revenue_col, "sum"),
            transactions=(product_col, "count"),
            avg_quantity=(quantity_col, "mean"),
            avg_revenue=(revenue_col, "mean"),
        )
        .sort_values("total_quantity", ascending=False)
    )


def compute_category_summary(df, category_col, revenue_col, quantity_col):
    # Build category-level summaries when category data is available.
    if category_col is None:
        return None
    return (
        df.groupby(category_col)
        .agg(
            total_quantity=(quantity_col, "sum"),
            total_revenue=(revenue_col, "sum"),
            transactions=(category_col, "count"),
        )
        .sort_values("total_revenue", ascending=False)
    )


def create_daily_product_demand(df, product_col, quantity_col):
    # Calculate product demand per day.
    return (
        df.groupby(["date_only", "day_name", product_col])[quantity_col]
        .sum()
        .reset_index()
    )


def create_next_week_forecast(df, product_col, quantity_col):
    # Create a simple next-week forecast based on average historical weekday demand.
    daily = create_daily_product_demand(df, product_col, quantity_col)

    weekday_avg = (
        daily.groupby(["day_name", product_col])[quantity_col]
        .mean()
        .reset_index(name="expected_qty")
    )

    weekday_order_local = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"]

    forecast_table = (
        weekday_avg.pivot(index=product_col, columns="day_name", values="expected_qty")
        .reindex(columns=weekday_order_local)
        .fillna(0)
    )

    forecast_table["next_week_total"] = forecast_table.sum(axis=1)
    forecast_table = forecast_table.sort_values("next_week_total", ascending=False)

    return forecast_table.round(1), weekday_avg.round(1)


def create_weekday_product_matrix(weekday_avg: pd.DataFrame, product_col: str) -> pd.DataFrame:
    # Show the weekday forecast as a cleaner matrix.
    weekday_order_local = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"]

    matrix = (
        weekday_avg.pivot(index=product_col, columns="day_name", values="expected_qty")
        .reindex(columns=weekday_order_local)
        .fillna(0)
        .round(1)
        .reset_index()
    )

    return matrix


def create_today_hourly_profile(df, product_col, quantity_col):
    # Analyze hourly demand if time data is available.
    if "hour" not in df.columns or df["hour"].isna().all():
        return None, None

    hourly = (
        df.groupby(["hour", product_col])[quantity_col]
        .sum()
        .reset_index()
    )

    total_hourly = (
        df.groupby("hour")[quantity_col]
        .sum()
        .reset_index(name="total_qty")
        .sort_values("hour")
    )

    return hourly, total_hourly


def compute_abc_analysis(product_summary: pd.DataFrame) -> pd.DataFrame:
    # Build ABC analysis based on cumulative revenue share.
    abc = product_summary.reset_index().copy()
    abc = abc.sort_values("total_revenue", ascending=False).reset_index(drop=True)

    abc["revenue_share"] = abc["total_revenue"] / abc["total_revenue"].sum()
    abc["cumulative_revenue_share"] = abc["revenue_share"].cumsum()

    def classify(value):
        if value <= 0.80:
            return "A"
        elif value <= 0.95:
            return "B"
        return "C"

    abc["abc_class"] = abc["cumulative_revenue_share"].apply(classify)
    return abc


def compute_product_portfolio(product_summary: pd.DataFrame) -> pd.DataFrame:
    # Classify products into strategic groups.
    portfolio = product_summary.reset_index().copy()

    qty_mean = portfolio["total_quantity"].mean()
    rev_mean = portfolio["total_revenue"].mean()

    def classify(row):
        high_qty = row["total_quantity"] >= qty_mean
        high_rev = row["total_revenue"] >= rev_mean

        if high_qty and high_rev:
            return "Star Product"
        elif high_qty and not high_rev:
            return "Traffic Builder"
        elif not high_qty and high_rev:
            return "Premium Niche"
        return "Underperformer"

    portfolio["portfolio_type"] = portfolio.apply(classify, axis=1)
    return portfolio


def calculate_stock_status(action_table: pd.DataFrame) -> pd.DataFrame:
    # Translate recommendation logic into a traffic-light style stock status.
    status_df = action_table.copy()

    stock_status = []
    stock_emoji = []

    for action in status_df["suggested_action"]:
        if action == "Increase Stock And Prep":
            stock_status.append("Urgent Restock")
            stock_emoji.append("🔴")
        elif action == "Prepare Extra For Next Week":
            stock_status.append("Monitor Closely")
            stock_emoji.append("🟠")
        else:
            stock_status.append("Sufficient")
            stock_emoji.append("🟢")

    status_df["stock_status"] = stock_status
    status_df["stock_indicator"] = stock_emoji
    return status_df


def build_action_table(df, product_summary, forecast_table, product_col, revenue_col, quantity_col):
    # Build the recommendation table.
    max_date = pd.to_datetime(df["date_only"]).max()
    min_cutoff = max_date - pd.Timedelta(days=6)

    recent_df = df[pd.to_datetime(df["date_only"]) >= min_cutoff]

    recent_product = (
        recent_df.groupby(product_col)
        .agg(
            recent_qty=(quantity_col, "sum"),
            recent_revenue=(revenue_col, "sum"),
        )
    )

    actions = product_summary.join(recent_product, how="left").fillna(0)
    actions = actions.join(forecast_table[["next_week_total"]], how="left").fillna(0)

    avg_total_qty = actions["total_quantity"].mean()
    avg_recent_qty = actions["recent_qty"].mean()

    action_list = []
    priority_list = []
    par_levels = []

    for _, row in actions.iterrows():
        total_q = row["total_quantity"]
        recent_q = row["recent_qty"]
        next_week = row["next_week_total"]

        par_level = int(np.ceil(next_week * 1.15))
        par_levels.append(par_level)

        if recent_q > avg_recent_qty * 1.25 and total_q > avg_total_qty:
            action = "Increase Stock And Prep"
            priority = "High"
        elif next_week > avg_total_qty * 1.10:
            action = "Prepare Extra For Next Week"
            priority = "Medium"
        elif recent_q < avg_recent_qty * 0.60 and total_q < avg_total_qty * 0.75:
            action = "Reduce Stock Or Promote"
            priority = "Medium"
        else:
            action = "Maintain Normal Stock"
            priority = "Low"

        action_list.append(action)
        priority_list.append(priority)

    actions["suggested_action"] = action_list
    actions["priority"] = priority_list
    actions["suggested_par_level_next_week"] = par_levels

    priority_order = {"High": 1, "Medium": 2, "Low": 3}
    actions["priority_rank"] = actions["priority"].map(priority_order)

    actions = actions.sort_values(
        by=["priority_rank", "next_week_total", "total_revenue"],
        ascending=[True, False, False]
    ).drop(columns=["priority_rank"])

    return actions


def generate_management_insights(df, product_summary, category_summary, sales_by_day, total_hourly):
    # Generate short business-style insights.
    insights = []

    best_product = product_summary["total_quantity"].idxmax()
    worst_product = product_summary.sort_values("total_quantity", ascending=True).index[0]
    best_day = sales_by_day.idxmax()
    worst_day = sales_by_day.idxmin()

    insights.append(f"Restock **{best_product}** more aggressively, because it is currently the highest-volume product.")
    insights.append(f"Review **{worst_product}** carefully. It is the weakest seller and may need a promotion, lower stock level, or replacement.")
    insights.append(f"Plan more staff and prep for **{best_day}**, because that day produces the highest revenue.")
    insights.append(f"Use **{worst_day}** for promotions or combo deals, because it is the weakest-performing day.")

    if category_summary is not None and not category_summary.empty:
        best_category = category_summary["total_revenue"].idxmax()
        weakest_category = category_summary["total_revenue"].idxmin()
        insights.append(f"Feature **{best_category}** more prominently, because it contributes the most revenue.")
        insights.append(f"Re-evaluate **{weakest_category}**, because it contributes the least revenue.")

    if total_hourly is not None and not total_hourly.empty:
        peak_hour = int(total_hourly.loc[total_hourly["total_qty"].idxmax(), "hour"])
        quiet_hour = int(total_hourly.loc[total_hourly["total_qty"].idxmin(), "hour"])
        insights.append(f"Prepare ahead of **{peak_hour}:00**, because this is the busiest hour in the data.")
        insights.append(f"Use quieter periods around **{quiet_hour}:00** for cleaning, replenishment, or short promotional pushes.")

    return insights

# Title and instructions
#
# I included a short explanation and a template file so the dashboard is easier to use and reuse.

st.title("Sales Decision Support Dashboard")
st.caption("Reusable template mode: companies can upload their own sales CSV and use the same decision engine.")

with st.expander("How To Use This Dashboard", expanded=False):
    st.write("""
1. Upload a CSV file in the sidebar, or use a local `sales.csv`.  
2. The app will try to detect the right columns automatically.  
3. If needed, turn on manual column mapping.  
4. Use the tabs for insights, planning, and decision suggestions.

**Recommended columns**
- Date
- Time (optional)
- Product
- Category (optional)
- Quantity
- Revenue / Total
- Unit Price (optional)
""")

template_csv = build_template_csv()
st.download_button(
    "Download CSV Template",
    data=template_csv,
    file_name="dashboard_template.csv",
    mime="text/csv"
)

# Data input
#
# File upload makes the dashboard reusable.
# If no file is uploaded, the dashboard can use a local CSV file for testing.

st.sidebar.header("Data Input")

uploaded_file = st.sidebar.file_uploader("Upload Company CSV File", type=["csv"])

if uploaded_file is not None:
    raw_df = pd.read_csv(uploaded_file)
    st.sidebar.success("Uploaded file loaded.")
else:
    st.sidebar.info("No file uploaded. Using local sales.csv if available.")
    try:
        raw_df = pd.read_csv("sales.csv")
    except FileNotFoundError:
        st.error("No dataset found. Upload a CSV file or place sales.csv in the project folder.")
        st.stop()

raw_df = normalize_columns(raw_df)
auto_map = detect_columns(raw_df)

use_manual_mapping = st.sidebar.checkbox("Use Manual Column Mapping", value=False)

if use_manual_mapping:
    selected_map = manual_column_mapper(raw_df, auto_map)
    df, detected, error = prepare_data_manual(raw_df, selected_map)
else:
    df, detected, error = prepare_data_auto(raw_df)

if error:
    st.error(error)
    st.write("Columns found:", [pretty_label(col) for col in raw_df.columns.tolist()])
    st.stop()

date_col = detected["date_col"]
time_col = detected["time_col"]
product_col = detected["product_col"]
category_col = detected["category_col"]
quantity_col = detected["quantity_col"]
revenue_col = detected["revenue_col"]

# Filters and scenario planning
#
# The filters make the dashboard dynamic.
# The scenario slider helps estimate a busier or quieter next week.

st.sidebar.header("Filters")

min_date = pd.to_datetime(df[date_col]).min().date()
max_date = pd.to_datetime(df[date_col]).max().date()

selected_dates = st.sidebar.date_input(
    "Date Range",
    value=(min_date, max_date),
    min_value=min_date,
    max_value=max_date
)

if isinstance(selected_dates, tuple) and len(selected_dates) == 2:
    start_date, end_date = selected_dates
else:
    start_date, end_date = min_date, max_date

filtered_df = df[
    (pd.to_datetime(df[date_col]).dt.date >= start_date) &
    (pd.to_datetime(df[date_col]).dt.date <= end_date)
].copy()

if category_col is not None:
    all_categories = sorted(filtered_df[category_col].dropna().unique().tolist())
    selected_categories = st.sidebar.multiselect("Categories", all_categories, default=all_categories)
    filtered_df = filtered_df[filtered_df[category_col].isin(selected_categories)]

all_products = sorted(filtered_df[product_col].dropna().unique().tolist())
selected_products = st.sidebar.multiselect("Products", all_products, default=all_products)
filtered_df = filtered_df[filtered_df[product_col].isin(selected_products)]

weekday_order = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"]

selected_days = st.sidebar.multiselect("Weekdays", weekday_order, default=weekday_order)
filtered_df = filtered_df[filtered_df["day_name"].isin(selected_days)]

safety_factor = st.sidebar.slider("Safety Stock Factor", 1.00, 1.50, 1.15, 0.05)
today_prep_horizon = st.sidebar.slider("Rest-Of-Day Prep Horizon (Hours)", 1, 8, 3)

st.sidebar.header("Scenario Planning")
scenario_change_pct = st.sidebar.slider("Expected Demand Change Next Week (%)", -30, 50, 0, 5)

if filtered_df.empty:
    st.warning("No data is left after filtering.")
    st.stop()

# KPI section
#
# I placed the KPI cards near the top so the user first gets a quick overall view.

st.subheader("Dashboard Overview")

total_revenue = filtered_df[revenue_col].sum()
total_quantity = filtered_df[quantity_col].sum()
total_transactions = len(filtered_df)
avg_transaction_value = filtered_df[revenue_col].mean()
avg_quantity_per_transaction = filtered_df[quantity_col].mean()

c1, c2, c3, c4, c5 = st.columns(5)
c1.metric("Revenue", fmt_currency(total_revenue))
c2.metric("Quantity Sold", f"{total_quantity:,.0f}")
c3.metric("Transactions", f"{total_transactions:,}")
c4.metric("Avg Transaction Value", fmt_currency(avg_transaction_value))
c5.metric("Avg Quantity / Transaction", f"{avg_quantity_per_transaction:.2f}")

# Core calculations
#
# These summaries are used across the whole dashboard.

product_summary = compute_product_summary(filtered_df, product_col, revenue_col, quantity_col)
category_summary = compute_category_summary(filtered_df, category_col, revenue_col, quantity_col)

sales_by_day = (
    filtered_df.groupby("day_name")[revenue_col]
    .sum()
    .reindex(weekday_order)
    .fillna(0)
)

sales_over_time = (
    filtered_df.groupby("date_only")[revenue_col]
    .sum()
    .sort_index()
)

forecast_table, weekday_avg = create_next_week_forecast(filtered_df, product_col, quantity_col)
weekday_product_matrix = create_weekday_product_matrix(weekday_avg, product_col)

forecast_table["next_week_total"] = np.ceil(forecast_table["next_week_total"] * safety_factor)

scenario_multiplier = 1 + (scenario_change_pct / 100)
forecast_table["scenario_next_week_total"] = np.ceil(forecast_table["next_week_total"] * scenario_multiplier)

hourly, total_hourly = create_today_hourly_profile(filtered_df, product_col, quantity_col)

action_table = build_action_table(filtered_df, product_summary, forecast_table, product_col, revenue_col, quantity_col)
action_table["suggested_par_level_next_week"] = np.ceil(action_table["next_week_total"] * safety_factor).astype(int)
action_table["scenario_next_week_total"] = np.ceil(action_table["next_week_total"] * scenario_multiplier).astype(int)
action_table["scenario_par_level"] = np.ceil(action_table["suggested_par_level_next_week"] * scenario_multiplier).astype(int)
action_table = calculate_stock_status(action_table)

management_insights = generate_management_insights(filtered_df, product_summary, category_summary, sales_by_day, total_hourly)
abc_analysis = compute_abc_analysis(product_summary)
portfolio_analysis = compute_product_portfolio(product_summary)

# Dashboard tabs
#
# I divided the dashboard into multiple tabs so the user moves from general understanding
# to more specific decision support.

tab1, tab2, tab3, tab4, tab5, tab6 = st.tabs([
    "Dashboard Overview",
    "Product Performance",
    "Today’s Decisions",
    "Next Week Planning",
    "Recommended Actions",
    "Data & Settings"
])

with tab1:
    st.caption("This section gives a general overview of sales performance across time, products, and categories.")

    left, right = st.columns(2)

    with left:
        st.markdown("### Revenue By Weekday")

        weekday_chart_df = sales_by_day.reset_index()
        weekday_chart_df.columns = ["day_name", "revenue"]

        weekday_chart = alt.Chart(weekday_chart_df).mark_bar().encode(
            x=alt.X("day_name:N", sort=weekday_order, title="Weekday"),
            y=alt.Y("revenue:Q", title="Revenue"),
            tooltip=[
                alt.Tooltip("day_name:N", title="Weekday"),
                alt.Tooltip("revenue:Q", title="Revenue", format=",.2f")
            ]
        ).properties(height=400)

        st.altair_chart(weekday_chart, use_container_width=True)

        st.markdown("### Revenue Over Time")
        st.line_chart(sales_over_time)

    with right:
        st.markdown("### Top Products By Quantity")
        st.bar_chart(product_summary["total_quantity"].head(15))

        st.markdown("### Top Products By Revenue")
        st.bar_chart(product_summary["total_revenue"].head(15))

    if category_summary is not None:
        st.markdown("### Category Performance")
        st.dataframe(
            make_ranked(prettify_dataframe(category_summary.reset_index())),
            use_container_width=True,
            hide_index=True
        )

with tab2:
    st.caption("This section shows how individual products perform and which products deserve most managerial attention.")

    st.markdown("### Product Performance Table")
    st.dataframe(
        make_ranked(prettify_dataframe(product_summary.reset_index())),
        use_container_width=True,
        hide_index=True
    )

    # Correct logic:
    # Strongest products are based on the highest total quantity.
    # Weakest products are based on the lowest total quantity.
    strongest_products = product_summary.sort_values("total_quantity", ascending=False).head(5).reset_index()
    weakest_products = product_summary.sort_values("total_quantity", ascending=True).head(5).reset_index()

    col_a, col_b = st.columns(2)

    with col_a:
        st.markdown("### Top 5 Strongest Products")
        st.table(make_ranked(prettify_dataframe(strongest_products)))

    with col_b:
        st.markdown("### Top 5 Weakest Products")
        st.table(make_ranked(prettify_dataframe(weakest_products)))

    top5_revenue_share = (product_summary["total_revenue"].head(5).sum() / product_summary["total_revenue"].sum()) * 100
    st.info(f"The top 5 products generate **{top5_revenue_share:.1f}%** of total revenue in the filtered data.")

    st.markdown("### ABC Analysis")
    st.caption("ABC analysis classifies products based on cumulative revenue contribution.")
    abc_display = abc_analysis.copy()
    abc_display["revenue_share"] = (abc_display["revenue_share"] * 100).round(1).astype(str) + "%"
    abc_display["cumulative_revenue_share"] = (abc_display["cumulative_revenue_share"] * 100).round(1).astype(str) + "%"
    st.table(make_ranked(prettify_dataframe(abc_display)))

    st.markdown("### Product Portfolio Analysis")
    st.caption("Products are classified into strategic groups based on quantity sold and revenue contribution.")
    st.dataframe(
        make_ranked(prettify_dataframe(portfolio_analysis)),
        use_container_width=True,
        hide_index=True
    )

with tab3:
    st.caption("This section supports operational decisions for the rest of today.")

    if total_hourly is not None:
        st.markdown("### Hourly Demand Pattern")
        hourly_chart = total_hourly.set_index("hour")["total_qty"]
        st.line_chart(hourly_chart)

        peak_hour = int(total_hourly.loc[total_hourly["total_qty"].idxmax(), "hour"])
        st.success(f"Peak hour detected: **{peak_hour}:00**")

        current_hour = pd.Timestamp.now().hour
        upcoming_hours = list(range(current_hour, min(current_hour + today_prep_horizon, 24)))

        if len(upcoming_hours) > 0:
            upcoming = (
                hourly[hourly["hour"].isin(upcoming_hours)]
                .groupby(product_col)[quantity_col]
                .sum()
                .sort_values(ascending=False)
                .head(10)
                .reset_index()
            )
            upcoming.columns = [product_col, "suggested_prep_qty"]
            upcoming = prettify_dataframe(upcoming)

            st.markdown(f"### Suggested Prep For The Next {today_prep_horizon} Hour(s)")
            st.caption("These products have historically shown the highest expected demand in the upcoming time window.")

            prep_display = make_ranked(upcoming)

            left_col, right_col = st.columns([2, 1])
            with left_col:
                st.table(prep_display)
            with right_col:
                st.markdown("#### Quick Note")
                st.write("This table highlights the products that deserve the most preparation in the next few hours.")

            top_upcoming_names = upcoming.iloc[:5, 0].astype(str).tolist()
            if top_upcoming_names:
                st.warning(
                    f"Prepare extra units of **{', '.join(top_upcoming_names)}** for the upcoming hours based on historical hourly demand."
                )
        else:
            st.info("No upcoming hour window available.")
    else:
        st.info("No time column detected in the dataset, so hourly planning is not available.")

    st.markdown("### Operational Suggestions For Today")
    for insight in management_insights:
        st.success(insight)

with tab4:
    st.caption("This section supports planning decisions for the coming week.")

    st.markdown("### Scenario Planning")
    if scenario_change_pct == 0:
        st.info("No demand-change scenario applied. Forecast reflects the baseline expectation.")
    elif scenario_change_pct > 0:
        st.warning(f"A **{scenario_change_pct}% increase** in demand is applied to next week's forecast.")
    else:
        st.info(f"A **{abs(scenario_change_pct)}% decrease** in demand is applied to next week's forecast.")

    st.markdown("### Next Week Forecast By Product")
    forecast_display = forecast_table.reset_index().copy()
    st.dataframe(
        make_ranked(prettify_dataframe(forecast_display)),
        use_container_width=True,
        hide_index=True
    )

    st.markdown("### Average Demand Per Weekday And Product")
    st.caption("This matrix gives a clearer overview of expected average demand per product across the week.")
    weekday_matrix_display = prettify_dataframe(weekday_product_matrix)
    st.dataframe(
        make_ranked(weekday_matrix_display),
        use_container_width=True,
        hide_index=True
    )

    st.markdown("### Scenario-Based Forecast")
    scenario_display = forecast_table.reset_index()[[product_col, "next_week_total", "scenario_next_week_total"]].copy()
    st.table(make_ranked(prettify_dataframe(scenario_display)))

    top_week = forecast_table[["scenario_next_week_total"]].sort_values("scenario_next_week_total", ascending=False).head(10)
    st.markdown("### Products Requiring Most Preparation Next Week")
    st.bar_chart(top_week)

    csv_forecast = forecast_table.reset_index().to_csv(index=False).encode("utf-8")
    st.download_button(
        "Download Next Week Forecast As CSV",
        data=csv_forecast,
        file_name="next_week_forecast.csv",
        mime="text/csv"
    )

with tab5:
    st.caption("This section converts the data into concrete business actions.")

    st.markdown("### Stock Status Overview")
    status_counts = action_table["stock_status"].value_counts()
    sc1, sc2, sc3 = st.columns(3)
    sc1.metric("Urgent Restock", int(status_counts.get("Urgent Restock", 0)))
    sc2.metric("Monitor Closely", int(status_counts.get("Monitor Closely", 0)))
    sc3.metric("Sufficient", int(status_counts.get("Sufficient", 0)))

    st.markdown("### Suggested Business Decisions")
    action_display = action_table[[
        "stock_indicator",
        "stock_status",
        "total_quantity",
        "total_revenue",
        "recent_qty",
        "next_week_total",
        "scenario_next_week_total",
        "suggested_action",
        "priority",
        "suggested_par_level_next_week",
        "scenario_par_level"
    ]].reset_index()

    action_display = prettify_dataframe(action_display)
    st.dataframe(
        make_ranked(action_display),
        use_container_width=True,
        hide_index=True
    )

    st.markdown("### High-Priority Actions")
    high_priority = action_table[action_table["priority"] == "High"]
    if high_priority.empty:
        st.info("No high-priority actions detected in the current filter selection.")
    else:
        for product in high_priority.index.tolist():
            row = high_priority.loc[product]
            st.error(
                f"**{product}** → {row['suggested_action']} | Stock status: **{row['stock_status']}** | Baseline par level: **{int(row['suggested_par_level_next_week'])}** | Scenario par level: **{int(row['scenario_par_level'])}**"
            )

    st.markdown("### Management Summary")

    best_product = product_summary.index[0]
    weakest_product = product_summary.sort_values("total_quantity", ascending=True).index[0]
    best_day = sales_by_day.idxmax()
    weakest_day = sales_by_day.idxmin()

    st.write(f"""
**What happened?**  
**{best_product}** is the strongest-performing product, while **{weakest_product}** is the weakest.  
The strongest sales day is **{best_day}**, while **{weakest_day}** is the weakest.

**What does it mean?**  
Demand is concentrated on a limited group of high-performing products and stronger weekdays.  
This means the business should focus stock, staffing, and prep capacity where demand is structurally highest.

**What should the business do?**  
Increase stock for high-performing products, reduce or promote weaker products, monitor medium-risk items closely, and prepare more capacity on strong days and peak hours.  
The scenario slider can be used to test whether the company should prepare for a busier or quieter next week.
""")

    csv_actions = action_table.reset_index().to_csv(index=False).encode("utf-8")
    st.download_button(
        "Download Action Table As CSV",
        data=csv_actions,
        file_name="decision_actions.csv",
        mime="text/csv"
    )

with tab6:
    st.caption("This section shows the technical setup and reusability of the dashboard.")

    st.markdown("### Reusability / Template Support")
    st.info(
        "This dashboard can also be used as a template for other companies. "
        "Users can upload their own CSV file, rely on automatic column detection, "
        "or manually map columns when their file structure differs."
    )

    st.markdown("### Detected Columns")
    detected_display = {pretty_label(k): pretty_label(v) if v is not None else "Not Detected" for k, v in detected.items()}
    st.write(detected_display)

    st.markdown("### Cleaned Dataset Preview")
    st.dataframe(
        make_ranked(prettify_dataframe(filtered_df.head(20))),
        use_container_width=True,
        hide_index=True
    )

    st.markdown("### Data Coverage")
    dq1, dq2, dq3 = st.columns(3)
    dq1.metric("Rows", f"{len(filtered_df):,}")
    dq2.metric("Products", f"{filtered_df[product_col].nunique():,}")
    dq3.metric("Date Range", f"{start_date} to {end_date}")

    if time_col is None:
        st.warning("No time column detected. Hourly planning features are disabled.")
    if category_col is None:
        st.info("No category column detected. Category analysis is limited.")

# Final reflection in the code
#
# Looking back at the code, I think the strongest point is that the dashboard does more
# than simply visualize data. It also interprets the data and translates it into possible actions.
#
# Another strong point is the reusable setup. By adding file upload, automatic column detection,
# manual mapping, and a template CSV, I made the dashboard more flexible than one built only
# for a single file.
#
# I also spent time improving the layout and readability. The final dashboard became stronger
# not only because of the calculations, but also because of the way the information is structured
# and presented.
