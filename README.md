import streamlit as st
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

# =========================
# TITLE
# =========================
st.title("📱 Fitness App User Growth Model")

# =========================
# INPUT FORM
# =========================
with st.form("input_form"):
    st.subheader("🧾 Enter Input Parameters")

    days = st.number_input("Number of Days", min_value=30, max_value=365, value=100)
    initial_users = st.number_input("Initial Users", min_value=50, max_value=10000, value=100)
    growth_rate = st.number_input("Growth Rate (%)", min_value=1.0, max_value=50.0, value=5.0) / 100
    retention_rate = st.number_input("Initial Retention (%)", min_value=50.0, max_value=100.0, value=85.0) / 100
    retention_improvement = st.number_input("Retention Improvement (%)", min_value=0.0, max_value=5.0, value=0.05) / 100

    submit = st.form_submit_button("🚀 Generate Analysis")

# =========================
# MODEL EXECUTION
# =========================
if submit:

    new_users = []
    active_users = []
    dropouts = []
    retention_list = []

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
        retention_list.append(current_retention)

    # =========================
    # DATAFRAME
    # =========================
    df = pd.DataFrame({
        "Day": range(1, int(days)+1),
        "New Users": new_users,
        "Active Users": active_users,
        "Dropouts": dropouts,
        "Retention Rate": retention_list
    })

    # =========================
    # GRAPH
    # =========================
    st.subheader("📈 User Growth Over Time")

    fig, ax = plt.subplots()
    ax.plot(df["Day"], df["Active Users"], label="Active Users")
    ax.plot(df["Day"], df["New Users"], label="New Users")
    ax.plot(df["Day"], df["Dropouts"], label="Dropouts")

    ax.set_xlabel("Days")
    ax.set_ylabel("Users")
    ax.legend()

    st.pyplot(fig)

    # =========================
    # PREDICTION SECTION (NEW)
    # =========================
    st.subheader("📊 Prediction of Daily Users of a Fitness App")

    st.write("""
This section represents the predicted daily user behavior of the fitness app 
based on exponential growth and retention analysis.

- The graph shows how users increase over time 📈  
- It also shows how many users remain active 🔁  
- And how many users drop out ⚠️  

This helps in understanding future growth and sustainability of the app.
""")

    # =========================
    # FORMULAS + VARIABLES
    # =========================
    st.subheader("📐 Mathematical Model & Variable Explanation")

    st.latex(r"N(t) = N_0 \cdot e^{rt}")
    st.write("""
➡️ **Exponential Growth Formula**

- N(t): Number of new users at time t  
- N₀: Initial users  
- r: Growth rate  
- t: Time (days)  
- e: Euler’s constant (~2.718)  
""")

    st.latex(r"A(t) = A(t-1) \cdot R(t) + N(t)")
    st.write("""
➡️ **Active Users Formula**

- A(t): Active users at time t  
- A(t-1): Previous day active users  
- R(t): Retention rate  
- N(t): New users  
""")

    st.latex(r"Dropouts = A(t-1) \cdot (1 - R(t))")
    st.write("""
➡️ **Dropout Formula**

- Dropouts: Users leaving the app  
- A(t-1): Previous active users  
- R(t): Retention rate  
""")

    st.latex(r"R(t) = R(t-1) + \Delta R")
    st.write("""
➡️ **Retention Improvement Formula**

- R(t): Current retention rate  
- R(t-1): Previous retention rate  
- ΔR: Retention improvement factor  
""")

    # =========================
    # TABLE
    # =========================
    st.subheader("📋 Data Table")
    st.dataframe(df)

    # =========================
    # BAR GRAPH
    # =========================
    st.subheader("📊 Final Day Comparison")

    fig2, ax2 = plt.subplots()
    ax2.bar(["Active Users", "New Users", "Dropouts"], [
        df["Active Users"].iloc[-1],
        df["New Users"].iloc[-1],
        df["Dropouts"].iloc[-1]
    ])
    st.pyplot(fig2)

    # =========================
    # CONCLUSION
    # =========================
    st.subheader("🧠 Detailed Analysis & Conclusion")

    total_new = df["New Users"].sum()
    final_active = df["Active Users"].iloc[-1]
    avg_retention = df["Retention Rate"].mean() * 100
    total_dropouts = df["Dropouts"].sum()

    growth_factor = final_active / initial_users
    churn_ratio = total_dropouts / total_new if total_new > 0 else 0

    st.write(f"🔹 Total Users Acquired: {int(total_new)}")
    st.write(f"🔹 Final Active Users: {int(final_active)}")
    st.write(f"🔹 Growth Factor: {round(growth_factor,2)}x")
    st.write(f"🔹 Average Retention Rate: {round(avg_retention,2)}%")
    st.write(f"🔹 Total Dropouts: {int(total_dropouts)}")
    st.write(f"🔹 Churn Ratio: {round(churn_ratio,2)}")

    st.write("""
📊 This model combines:
- Exponential Growth → user acquisition 📈  
- Retention Analysis → user engagement 🔁  
- Combined effect → long-term success ⚖️  
""")

    if growth_factor > 3 and avg_retention > 85:
        st.success("🚀 High Growth: Strong acquisition + excellent retention → scalable system.")
    elif growth_factor > 2 and avg_retention > 70:
        st.info("📈 Balanced Growth: Stable but can improve retention.")
    elif avg_retention < 70:
        st.warning("⚠️ Retention Problem: High churn is limiting growth.")
    else:
        st.warning("⚠️ Slow Growth: Acquisition rate is low.")

    # =========================
    # RETENTION STRATEGIES
    # =========================
    st.subheader("💡 Retention Improvement Strategies")

    st.write("""
👉 Improve onboarding experience  
👉 Add gamification (badges, streaks)  
👉 Send push notifications & reminders  
👉 Personalize user experience  
👉 Provide regular content updates  
👉 Offer rewards & loyalty programs  
👉 Improve app UI/UX for better engagement  
""")

    # =========================
    # DOWNLOAD
    # =========================
    csv = df.to_csv(index=False).encode('utf-8')
    st.download_button(
        "📥 Download Data",
        csv,
        "fitness_data.csv",
        "text/csv"
    )
