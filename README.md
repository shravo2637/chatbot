!pip install -q google-generativeai pdfplumber

import google.generativeai as genai
import pdfplumber
from google.colab import files

uploaded = files.upload()
pdf_path = list(uploaded.keys())[0]


def extract_text_from_pdf(path):
    text = ""
    with pdfplumber.open(path) as pdf:
        for page in pdf.pages:
            text += page.extract_text() + "\n"
    return text.strip()

pdf_text = extract_text_from_pdf(pdf_path)


max_characters = 100000  # THAT IS AROUND 25,000 tokens
pdf_text = pdf_text[:max_characters]


genai.configure(api_key="AIzaSyCRd6i3Vajclfx1deM3i7rZQMQPPuBYxjU")

model = genai.GenerativeModel("models/gemini-1.5-flash")


def generate_response(user_prompt):
    full_prompt = f"""You are a helpful assistant. Use the following PDF content to answer:

--- BEGIN PDF CONTENT ---
{pdf_text}
--- END PDF CONTENT ---

User question: {user_prompt}
Answer:"""

    response = model.generate_content(full_prompt)
    return response.text.strip()

def chat():
    print("Bot: Ask questions based on your PDF. Type 'bye' to exit.")
    while True:
        user_input = input("You: ")
        if user_input.lower() in ['bye', 'exit']:
            print("Bot: Goodbye!")
            break
        response = generate_response(user_input)
        print("Bot:", response)

chat()
