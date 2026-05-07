# research-partner
It will help me 
import streamlit as st
import google.generativeai as genai
import os

# 1. Setup
st.set_page_config(page_title="AI Research Partner", layout="wide")
st.title("🔬 Personal Research Partner")

# Use secret API key from Streamlit Cloud
api_key = st.secrets["GEMINI_API_KEY"]
genai.configure(api_key=api_key)
model = genai.GenerativeModel('gemini-1.5-flash')

# 2. Sidebar for Research Files
with st.sidebar:
    st.header("Research Materials")
    uploaded_file = st.file_uploader("Upload a PDF for context", type=("pdf"))
    if st.button("Clear Chat History"):
        st.session_state.messages = []

# 3. Chat Logic
if "messages" not in st.session_state:
    st.session_state.messages = []

for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])

if prompt := st.chat_input("Ask about your research topic..."):
    st.session_state.messages.append({"role": "user", "content": prompt})
    with st.chat_message("user"):
        st.markdown(prompt)

    with st.chat_message("assistant"):
        # If a file is uploaded, send it with the prompt
        if uploaded_file:
            content = [prompt, {"mime_type": "application/pdf", "data": uploaded_file.getvalue()}]
            response = model.generate_content(content)
        else:
            response = model.generate_content(prompt)
            
        st.markdown(response.text)
        st.session_state.messages.append({"role": "assistant", "content": response.text})
        
