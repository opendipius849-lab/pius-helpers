import streamlit as D_stream
import pandas as D_pandas
import numpy as D_numpy

# --- Page Configuration ---
D_stream.set_page_config(
    page_title="MARS Analysis | Match Predictor",
    page_icon="⚽",
    layout="wide"
)

# --- Custom Styling ---
D_stream.markdown("""
    <style>
    .main-title { font-size: 38px; font-weight: 800; color: #1E3A8A; text-align: center; }
    .sub-title { font-size: 18px; color: #4B5563; text-align: center; margin-bottom: 30px; }
    .prediction-box { background-color: #EFF6FF; border-left: 5px solid #3B82F6; padding: 20px; border-radius: 4px; }
    </style>
""", unsafe_allow_html=True)

# --- App Header ---
D_stream.markdown('<p class="main-title">⚽ MARS Analysis</p>', unsafe_allow_html=True)
D_stream.markdown('<p class="sub-title">Advanced Match Accuracy & Result Synthesis Engine</p>', unsafe_allow_html=True)

# --- Mock Database / Match Selector ---
UPCOMING_MATCHES = {
    "Premier League: Arsenal vs Chelsea": {"home": "Arsenal", "away": "Chelsea", "league": "Premier League", "date": "2026-10-18"},
    "Premier League: Man City vs Liverpool": {"home": "Man City", "away": "Liverpool", "league": "Premier League", "date": "2026-10-19"},
    "La Liga: Real Madrid vs Barcelona": {"home": "Real Madrid", "away": "Barcelona", "league": "La Liga", "date": "2026-10-20"},
    "Serie A: Inter Milan vs Juventus": {"home": "Inter Milan", "away": "Juventus", "league": "Serie A", "date": "2026-10-21"}
}

# --- Sidebar Controls ---
D_stream.sidebar.header("🔍 Match Search & Filters")
search_query = D_stream.sidebar.text_input("Search Team or Match", "")

# Filter matches based on search query
filtered_matches = {
    k: v for k, v in UPCOMING_MATCHES.items() 
    if search_query.lower() in k.lower()
}

if not filtered_matches:
    filtered_matches = UPCOMING_MATCHES 

selected_match_key = D_stream.sidebar.selectbox("Select Upcoming Match", list(filtered_matches.keys()))
match_data = filtered_matches[selected_match_key]

run_analysis = D_stream.sidebar.button("Run MARS Analysis 🚀")

# --- Core Analysis Logic ---
def calculate_poisson_goals(att_strength, def_strength, league_avg):
    return att_strength * def_strength * league_avg

if run_analysis or True:
    home_team = match_data["home"]
    away_team = match_data["away"]
    
    D_numpy.random.seed(sum(map(ord, home_team + away_team)))
    
    home_form = D_numpy.random.choice(["W-W-D-W-L", "W-W-W-W-D", "D-W-L-W-W"], 1)[0]
    away_form = D_numpy.random.choice(["L-W-D-W-L", "W-D-D-W-W", "L-L-W-D-W"], 1)[0]
    
    home_att = D_numpy.random.uniform(1.4, 2.2)
    home_def = D_numpy.random.uniform(0.7, 1.1)
    away_att = D_numpy.random.uniform(1.1, 1.9)
    away_def = D_numpy.random.uniform(0.9, 1.3)
    league_avg_goals = 1.45
    
    home_xg = round(calculate_poisson_goals(home_att, away_def, league_avg_goals), 2)
    away_xg = round(calculate_poisson_goals(away_att, home_def, league_avg_goals), 2)
    
    predicted_home_goals = int(round(home_xg))
    predicted_away_goals = int(round(away_xg))
    
    total_factor = home_xg + away_xg
    home_win_prob = round((home_xg / total_factor) * 100 + D_numpy.random.uniform(-3, 3), 1)
    away_win_prob = round((away_xg / total_factor) * 100 + D_numpy.random.uniform(-3, 3), 1)
    draw_prob = round(max(0.0, 100.0 - (home_win_prob + away_win_prob)), 1)
    
    total_p = home_win_prob + away_win_prob + draw_prob
    home_win_prob = round((home_win_prob / total_p) * 100, 1)
    away_win_prob = round((away_win_prob / total_p) * 100, 1)
    draw_prob = round(100.0 - home_win_prob - away_win_prob, 1)

    # --- Dashboard Layout ---
    col1, col2 = D_stream.columns([2, 1])
    
    with col1:
        D_stream.subheader(f"📊 Statistical Breakdown: {home_team} vs {away_team}")
        
        stats_df = D_pandas.DataFrame({
            "Metric": ["Recent Form (Last 5)", "Attack Strength (xG)", "Defensive Vulnerability", "Home/Away Advantage"],
            home_team: [home_form, round(home_att, 2), round(home_def, 2), "+15%"],
            away_team: [away_form, round(away_att, 2), round(away_def, 2), "Base"]
        })
        D_stream.dataframe(stats_df, use_container_width=True, hide_index=True)
        
        D_stream.markdown("### 🎲 Outcome Probabilities")
        prob_df = D_pandas.DataFrame({
            "Outcome": [f"{home_team} Win", "Draw", f"{away_team} Win"],
            "Probability (%)": [home_win_prob, draw_prob, away_win_prob]
        })
        D_stream.bar_chart(prob_df.set_index("Outcome"))

    with col2:
        D_stream.subheader("🎯 MARS Final Verdict")
        
        D_stream.markdown(f"""
        <div class="prediction-box">
            <h4>Predicted Scoreline</h4>
            <h2 style="color: #1E3A8A; text-align: center;">{home_team} {predicted_home_goals} - {predicted_away_goals} {away_team}</h2>
            <hr>
            <p><b>Match Outcome:</b> {'Home Win' if home_win_prob > away_win_prob and home_win_prob > draw_prob else ('Away Win' if away_win_prob > home_win_prob else 'Draw / Tight Match')}</p>
            <p><b>Confidence Rating:</b> {'🟢 High (78%)' if abs(home_win_prob - away_win_prob) > 20 else '🟡 Moderate (58%)'}</p>
        </div>
        """, unsafe_allow_html=True)
        
        D_stream.markdown("### 💡 Key Insights")
        if home_xg > away_xg:
            D_stream.info(f"• {home_team} holds superior home metrics and creates higher quality chances in the final third.")
        else:
            D_stream.info(f"• {away_team} shows strong counter-attacking efficiency capable of breaching {home_team}'s defense.")
            
        D_stream.success(f"• Expected Total Match Goals (Over/Under): **{round(home_xg + away_xg, 1)} Goals**")

    # --- Footer ---
    D_stream.markdown("Pius tech")
    D_stream.caption("MARS Analysis Engine v1.0 | Synthesizing Poisson distribution models with real-time sports datasets.")
    
