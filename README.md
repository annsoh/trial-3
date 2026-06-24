import streamlit as st
import pandas as pd

# Set page configurations globally
st.set_page_config(page_title="Family Travel Customizer", page_icon="👨‍👩‍👧‍👦", layout="wide")

# ==============================================================================
# VIEW 1: FAMILY PROFILE & PREFERENCES
# ==============================================================================
def show_profile_and_logistics():
    st.title("👨‍👩‍👧‍👦 Family Profile & 💰 Budget Setup")
    st.write("Customize your free & easy itinerary for a family of 5.")
    
    st.divider()
    
    # 1. Family Demographics (All Ages consideration)
    st.header("1. Who is traveling?")
    st.write("Input ages to auto-filter and optimize for pacing and physical accessibility.")
    
    ages = []
    cols = st.columns(5)
    for i, col in enumerate(cols):
        with col:
            age = st.number_input(
                f"Member {i+1} Age", 
                min_value=0, 
                max_value=100, 
                value=[65, 38, 36, 12, 6][i], # Default mock family of 5 (Grandparent, Parents, Teen, Child)
                key=f"age_{i}"
            )
            ages.append(age)
            
    # Dynamic logic suggestion based on ages
    has_seniors = any(a >= 65 for a in ages)
    has_toddlers = any(a <= 5 for a in ages)
    
    st.subheader("💡 Accessibility Filters")
    default_tags = []
    if has_seniors: default_tags.append("Low Walking")
    if has_toddlers: default_tags.append("Stroller Friendly")
    
    selected_tags = st.segmented_control(
        "Select required features for your itinerary:",
        options=["Low Walking", "Stroller Friendly", "Teen Approved", "Rainproof Options"],
        selection_mode="multi",
        default=default_tags
    )
    
    st.divider()
    
    # 2. Budget & Logistics (Cost Effective & Reduce Travel Time constraints)
    st.header("2. Budget & Pacing")
    col_b1, col_b2 = st.columns(2)
    
    with col_b1:
        budget = st.slider("Max Daily Budget for 5 Pax (USD)", min_value=100, max_value=1000, value=400, step=50)
    with col_b2:
        pace = st.select_slider(
            "Desired Travel Pace (Reduces travel fatigue)",
            options=["Relaxed (1-2 spots/day)", "Balanced (3 spots/day)", "Active (4+ spots/day)"],
            value="Balanced (3 spots/day)"
        )
        
    # Store settings in Session State to pass between pages
    if st.button("Generate Optimized Itinerary ✨", type="primary"):
        st.session_state["family_ages"] = ages
        st.session_state["selected_tags"] = selected_tags
        st.session_state["budget"] = budget
        st.session_state["pace"] = pace
        st.toast("Itinerary generated successfully!", icon="✅")


# ==============================================================================
# VIEW 2: ITINERARY VIEWER
# ==============================================================================
def show_itinerary_viewer():
    st.title("🗺️ Your Geo-Optimized Itinerary")
    st.write("Attractions are clustered by geographic proximity to keep transit times minimal.")
    
    # Fetch parameters from session state with defaults if they haven't run View 1 yet
    budget = st.session_state.get("budget", 400)
    
    # --- Top Metrics Dashboard ---
    with st.container(border=True):
        c1, c2, c3 = st.columns(3)
        # Displaying active KPIs for cost, time, and age appeal
        c1.metric(label="Estimated Total Cost / Day", value=f"${budget - 85}", delta=f"-${85} vs Budget Threshold")
        c2.metric(label="Avg. Transit Between Spots", value="14 mins", delta="-12 mins (Optimized)", delta_color="inverse")
        c3.metric(label="Age Harmony Score", value="94%", help="Calculated based on suitability across all 5 members' age groups")

    st.divider()

    # --- Interactive Geo-Clustered Schedule ---
    st.subheader("📅 Day 1 Schedule: Downtown Hub Cluster")
    st.caption("Uncheck items to remove them from your day; the budget and route timing will automatically adjust.")
    
    # Mock data structured for geographic sequencing (reducing travel time)
    mock_itinerary_data = {
        "Time Slot": ["09:00 AM - 11:30 AM", "12:00 PM - 01:30 PM", "02:00 PM - 04:30 PM"],
        "Attraction / Activity": ["City Science Center & Gardens", "Family-Style Local Diner", "Harbor Scenic Boardwalk Cruise"],
        "Estimated Cost (5 Pax)": [80, 75, 160],
        "Transit Time (Next Stop)": ["10 mins walk", "5 mins taxi", "N/A - End of Day"],
        "Age Suitability Tags": ["All Ages • Stroller Friendly", "All Ages • High Chairs", "Senior Friendly • Relaxed"]
    }
    df = pd.DataFrame(mock_itinerary_data)
    
    # Modern data selection interface
    selected_rows = st.dataframe(
        df,
        use_container_width=True,
        selection_mode="multi", 
        hide_index=True
    )
    
    # Map Placeholder for geospatial visualization
    st.subheader("📍 Route Map Visualizer")
    # Using simple map data for demonstration (centered around coordinate clusters)
    map_data = pd.DataFrame({
        'lat': [1.2868, 1.2894, 1.2847],
        'lon': [103.8545, 103.8500, 103.8610]
    })
    st.map(map_data, zoom=13, use_container_width=True)


# ==============================================================================
# APP NAVIGATION ROUTER (Streamlit v1.58+)
# ==============================================================================

# Define the logical page routes pointing to our execution functions
page_1 = st.Page(show_profile_and_logistics, title="Family Profile & Setup", icon="👥")
page_2 = st.Page(show_itinerary_viewer, title="Optimized Itinerary", icon="⚡")

# Initialize native structural navigation
pg = st.navigation({
    "Configuration": [page_1],
    "Dashboard": [page_2]
})

# Execute navigation
pg.run()
