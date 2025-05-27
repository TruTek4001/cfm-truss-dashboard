import streamlit as st
import pandas as pd
import numpy as np
import plotly.express as px

# Load data (for deployment, replace this with actual file path or database call)
df = pd.read_csv("Projects backlog.csv")

# Clean numeric columns
def clean_currency(col):
    return pd.to_numeric(col.replace('[\$,]', '', regex=True).replace('', np.nan), errors='coerce')

money_columns = ['$ Deck Only', '$ Total Furnish + Install', '$ Est. GP (Furnish)', '$ Est. GP  (Install)']
for col in money_columns:
    df[col] = clean_currency(df[col])

df['Steel Weight'] = pd.to_numeric(df['Steel Weight'], errors='coerce')
df['Est. Man Hours'] = pd.to_numeric(df['Est. Man Hours'], errors='coerce')
df['# of Loads'] = pd.to_numeric(df['# of Loads'], errors='coerce')

# Title
st.title("Light Gauge Steel Truss Executive Dashboard")

# KPI Metrics
col1, col2, col3 = st.columns(3)
col1.metric("Total Projects", len(df))
col2.metric("Total Project Value ($)", f"${df['$ Total Furnish + Install'].sum():,.0f}")
col3.metric("Gross Profit ($)", f"${df['$ Est. GP (Furnish)'].sum() + df['$ Est. GP  (Install)'].sum():,.0f}")

col4, col5, col6 = st.columns(3)
col4.metric("Steel Weight (lbs)", f"{df['Steel Weight'].sum():,.0f}")
col5.metric("Est. Man Hours", f"{df['Est. Man Hours'].sum():,.0f}")
col6.metric("Total Loads", f"{df['# of Loads'].sum():,.0f}")

# Filters
phases = st.multiselect("Filter by Project Phase", options=df['Project Phase'].dropna().unique(), default=df['Project Phase'].dropna().unique())
states = st.multiselect("Filter by Region", options=df['Project Location: State/Region'].dropna().unique(), default=df['Project Location: State/Region'].dropna().unique())

filtered_df = df[df['Project Phase'].isin(phases) & df['Project Location: State/Region'].isin(states)]

# Charts
st.subheader("Project Value by Phase")
value_by_phase = filtered_df.groupby('Project Phase')['$ Total Furnish + Install'].sum().reset_index()
st.plotly_chart(px.bar(value_by_phase, x='Project Phase', y='$ Total Furnish + Install', title="Total Value by Project Phase", labels={'$ Total Furnish + Install': 'Total Value ($)'}))

st.subheader("Top 5 Projects by Value")
top_projects = filtered_df[['Project Name', '$ Total Furnish + Install']].sort_values(by='$ Total Furnish + Install', ascending=False).head(5)
st.table(top_projects)

st.subheader("Estimated Delivery Timeline")
df['Est Delivery Start'] = pd.to_datetime(df['Est Delivery Start'], errors='coerce')
timeline_df = filtered_df.dropna(subset=['Est Delivery Start'])
st.plotly_chart(px.timeline(timeline_df, x_start='Est Delivery Start', x_end='Est Delivery Start', y='Project Name', color='Project Phase'))
