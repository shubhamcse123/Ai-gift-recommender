import streamlit as st 
import google.generativeai as genai 
 
# Config Gemini 
genai.configure(api_key="AIzaSyAOUeQcJrcqdiNb0HR3htpDzWQnhD5by04") 
 
# Load the model 
model = genai.GenerativeModel(model_name="gemini-1.5-pro") 
 
# Page Config 
st.set_page_config(page_title="Gift Recommender", page_icon="🎁", layout="wide") 
 
# Apply custom styles 
st.markdown( 
    """ 
    <style> 
        body { 
            background-color: #f4f4f9; 
            color: #2e2e48; 
        } 
        .sidebar .sidebar-content { 
            background: #ffedf6; 
        } 
        .css-1d391kg { 
            font-family: 'Arial', sans-serif; 
        } 
        .stButton>button { 
            background: linear-gradient(45deg, #ff6ec4, #7873f5); 
            color: white; 
            font-size: 16px; 
            font-weight: bold; 
            border-radius: 8px; 
        } 
        .stMarkdown h1 { 
            font-family: 'Comic Sans MS', sans-serif; 
            text-align: center; 
            color: #ff69b4; 
        } 
    </style> 
    """, 
    unsafe_allow_html=True, 
) 
 
# Header 
st.markdown("<h1>🎁 Gift Recommender Chatbot</h1>", unsafe_allow_html=True) 
st.markdown( 
    "<p style='text-align: center; font-size: 20px; color: #2e2e48;'>Helping you choose the perfect 
gift for any occasion</p>", 
    unsafe_allow_html=True, 
) 
 
# Sidebar for personalization 
with st.sidebar: 
    st.markdown("### 🎯 Personalize Your Search") 
    recipient = st.selectbox( 
        "Who is this gift for?", 
        ["Friend", "Parent", "Child", "Partner", "Colleague", "Other"], 
        index=0, 
    ) 
    occasion = st.selectbox( 
        "Occasion", ["Birthday", "Anniversary", "Holiday", "Graduation", "Other"], index=0 
    ) 
    budget = st.slider("🎁 Budget (in $)", 10, 500, 50) 
    preferences = st.text_area( 
        "✨ Any specific interests or hobbies?", placeholder="E.g., loves reading, tech gadgets, 
sports" 
    ) 
 
# Suggested prompts 
suggested_prompts = [ 
    "What should I gift my mom on Mother's Day? 👨👧", 
    "What should I gift my dad on Father's Day? 👨👧", 
    "Need a creative gift for a best friend 💝", 
    "Gift suggestions for a 10-year-old boy 🚀", 
] 
 
st.markdown("### 💡 Try a Suggested Prompt:") 
cols = st.columns(len(suggested_prompts)) 
for i, prompt in enumerate(suggested_prompts): 
    with cols[i]: 
        if st.button(prompt): 
            st.session_state["user_input"] = prompt 
 
# Input box 
st.markdown("### 🗣️ Ask Your Question Below:") 
user_input = st.text_input("Type your query here 👇", key="user_input", placeholder="What 
are you looking for?") 
 
# Function to filter relevant gift-related queries 
def is_relevant_query(query): 
    keywords = ["gift", "present", "surprise", "buy", "suggest", "idea", "recommend"] 
    return any(word in query.lower() for word in keywords) 
 
# Main logic 
if user_input: 
    st.markdown("----") 
    with st.spinner("Thinking of great gift ideas... 🎁"): 
        if is_relevant_query(user_input): 
            prompt = ( 
                f"You are a helpful gift recommendation assistant. " 
                f"Recipient: {recipient}, Occasion: {occasion}, Budget: ${budget}, Preferences: 
{preferences}. " 
                f"User query: {user_input}. " 
                f"Suggest 2-3 thoughtful gift ideas with brief descriptions and links (if appropriate)." 
            ) 
            response = model.generate_content(prompt) 
            st.success("✅ Gift Suggestions:") 
            suggestions = response.text.split("\n\n") 
            for suggestion in suggestions: 
                st.markdown( 
                    f""" 
                    <div style='border: 2px solid #ff69b4; border-radius: 10px; padding: 15px; margin: 
10px 0; background-color: #fce4ec;'> 
                        <h4 style='color: #2e2e48;'>🎁 {suggestion.strip()}</h4> 
                    </div> 
                    """, 
                    unsafe_allow_html=True, 
                ) 
        else: 
            st.warning("❗ Please ask something related to gifts or recommendations.") 
 
# Add a history tracker 
if "history" not in st.session_state: 
    st.session_state["history"] = [] 
 
if user_input and is_relevant_query(user_input): 
    st.session_state["history"].append({"query": user_input, "response": response.text}) 
 
# Display previous queries 
if st.session_state["history"]: 
st.markdown("### 🕰️ Query History") 
for entry in st.session_state["history"]: 
with st.expander(f"Query: {entry['query']}"): 
st.markdown(f"**Response:** {entry['response']}")
