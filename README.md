# How to run

## Step 1 — Install Python
Make sure Python 3.10 or higher is installed.  

## Step 2 — Make sure the project files are in one folder
Place these files together in the same project folder:

- `dashboard.py`
- `requirements.txt`
- `README.md`
- `sales.csv`

**Important:** the project needs the file `sales.csv` to run if you do not upload another dataset manually in the dashboard.  
So make sure `sales.csv` is present in the same folder as `dashboard.py`.

## Step 3 — Install the required packages
Open your terminal (or Command Prompt / PowerShell on Windows) in the project folder and run: ##If pip does not work, try:

py -m streamlit run dashboard.py

If that doesn't work try:

python -m streamlit run dashboard.py

Required project files

The project should contain the following files:

dashboard.py → the main Streamlit dashboard
requirements.txt → the required Python packages
README.md → explanation of the project and how to run it
sales.csv → the dataset used by the dashboard
Important note about the dataset

The file sales.csv is required if no other CSV file is uploaded manually in the dashboard.

That means:

if you want the dashboard to start immediately, include sales.csv in the same folder
if sales.csv is missing, the user must upload a CSV file manually
