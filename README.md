The goal of this assignment is to analyze organizational data related to clients, industries, financial trends, and subscriptions.
 We aim to answer key business questions using Python (v3.7+) and derive actionable insights through data wrangling, filtering, and aggregation techniques.

Datasets Used:
industry_client_details.csv – Contains industry types mapped to client IDs.
finanical_information.csv – Includes inflation rates within specified date ranges.
payment_information.csv – Records payment transactions along with methods and amounts.
subscription_information.csv – Contains subscription periods and renewal statuses for each client.

Questions Answered:
How many Finance Lending and Blockchain clients does the organization have?
→ By filtering the industry-specific data and counting the unique client IDs.

Which industry in the organization has the highest renewal rate?
→ By merging industry and subscription data, grouping by industry, and calculating average renewal rates.

What was the average inflation rate when their subscriptions were renewed?
→ By filtering only the renewed subscriptions and mapping their end dates to the correct inflation period, we calculated the mean inflation rate.

What is the median amount paid each year for all payment methods?
→ By extracting the year from payment dates and grouping by payment method and year, we computed median values for payments.


Code for the above questions.

# Importing files from the system
from google.colab import files
import pandas as pd
uploaded = files.upload()

# Reading files
finance_df = pd.read_csv("finanical_information.csv")
industry_df = pd.read_csv("industry_client_details.csv")
payment_df = pd.read_csv("payment_information.csv")
subscription_df = pd.read_csv("subscription_information (1).csv")
# Converting date fields will help us to analyse data easily.
finance_df["start_date"] = pd.to_datetime(finance_df["start_date"])
finance_df["end_date"] = pd.to_datetime(finance_df["end_date"])
subscription_df["end_date"] = pd.to_datetime(subscription_df["end_date"])
payment_df["payment_date"] = pd.to_datetime(payment_df["payment_date"])


# Question 1: How many finance lending and blockchain clients does the organization have?
# Filter the industries
finance_lending_clients = industry_df[industry_df["industry"] == "Finance Lending"]
blockchain_clients = industry_df[industry_df["industry"] == "Block Chain"]
# Count unique clients, to remove duplicate values
num_finance_lending = finance_lending_clients["client_id"].nunique()
num_blockchain = blockchain_clients["client_id"].nunique()
print("Ques 1:")
print("Finance Lending Clients:", num_finance_lending)
print("Blockchain Clients:", num_blockchain)


#Question 2:Which industry in the organization has the highest renewal rate? 
# Merge to get industry + renewal status
merged_df = pd.merge(industry_df, subscription_df, on="client_id")
# Calculate renewal rates by industry
renewal_rates = merged_df.groupby("industry")["renewed"].mean().sort_values(ascending=False)
# Top industry by renewal rate
highest_renewal_industry = renewal_rates.idxmax()
highest_rate = renewal_rates.max()
print("\nQues 2:")
print("Industry with Highest Renewal Rate:", highest_renewal_industry)
print("Renewal Rate:", round(highest_rate * 100, 2), "%")


#Ques3:- What was the average inflation rate when their subscriptions were renewed?
# Filter only renewed subscriptions
renewed_subs = subscription_df[subscription_df["renewed"] == True]
# Function to find matching inflation rate
def find_inflation(date):
    row = finance_df[(finance_df["start_date"] <= date) & (finance_df["end_date"] >= date)]
    return row.iloc[0]["inflation_rate"] if not row.empty else None
# Apply function
renewed_subs["inflation_rate"] = renewed_subs["end_date"].apply(find_inflation)
# Average inflation
avg_inflation = renewed_subs["inflation_rate"].mean()
print("\nQues 3:")
print("Average Inflation Rate at Time of Renewals:", round(avg_inflation, 2))


#Ques4: What is the median amount paid each year for all payment methods? 
# Extract year
payment_df["year"] = payment_df["payment_date"].dt.year
# Group by year and payment method, then compute median
median_payments = payment_df.groupby(["year", "payment_method"])["amount_paid"].median().reset_index()
print("\nQues 4:")
print("Median Amount Paid Each Year by Payment Method:")
print(median_payments)
