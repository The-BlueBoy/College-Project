# College-Project

import streamlit as st
import numpy as np
import matplotlib.pyplot as plt

st.title("📱 Fitness App User Growth Model")

# FORM (Input Box System)
with st.form("user_input_form"):
    st.subheader("Enter Input Parameters")

    days = st.number_input("Number of Days", min_value=30, max_value=180, value=100)
    initial_users = st.number_input("Initial Users", min_value=50, max_value=1000, value=100)
    growth_rate = st.number_input("Growth Rate (%)", min_value=1.0, max_value=20.0, value=5.0) / 100
    retention_rate = st.number_input("Initial Retention (%)", min_value=50.0, max_value=100.0, value=85.0) / 100
    retention_improvement = st.number_input("Retention Improvement (%)", min_value=0.0, max_value=2.0, value=0.05) / 100

    submit = st.form_submit_button("Generate Graph 📊")

# Only run model AFTER button click
if submit:

    new_users = []
    active_users = []
    dropouts = []

    current_active = initial_users
    current_retention = retention_rate

    for day in range(int(days)):
        new = initial_users * np.exp(growth_rate * day)
        current_retention = min(current_retention + retention_improvement, 0.95)

        retained = current_active * current_retention
        dropped = current_active * (1 - current_retention)

        current_active = retained + new

        new_users.append(new)
        active_users.append(current_active)
        dropouts.append(dropped)

    # 📈 Line Graph
    st.subheader("📈 User Growth Over Time")

    fig, ax = plt.subplots()
    ax.plot(active_users, label="Active Users")
    ax.plot(new_users, label="New Users")
    ax.plot(dropouts, label="Dropouts")
    ax.set_xlabel("Days")
    ax.set_ylabel("Users")
    ax.legend()

    st.pyplot(fig)

    # 📊 Bar Graph
    st.subheader("📊 Final Day Comparison")

    labels = ["Active Users", "New Users", "Dropouts"]
    values = [active_users[-1], new_users[-1], dropouts[-1]]

    fig2, ax2 = plt.subplots()
    ax2.bar(labels, values)

    st.pyplot(fig2)

    # 🧠 Conclusion
    st.subheader("🧠 Conclusion")

    if active_users[-1] > 2 * initial_users:
        st.success("Strong growth observed 🚀 - Retention strategy is effective.")
    elif active_users[-1] > initial_users:
        st.info("Moderate growth 📈 - Consider improving retention or marketing.")
    else:
        st.warning("Low growth ⚠️ - High dropouts detected. Improve user engagement.")

    # Extra insights
    st.write(f"Final Active Users: {int(active_users[-1])}")
    st.write(f"Final Retention Rate: {round(current_retention*100,2)}%")
