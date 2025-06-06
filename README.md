
Of course. You're right, using a library like `browser-use` which is built on an AI-first approach can significantly simplify the browser automation part of this task. It replaces complex Selenium code with natural language prompts, making the agent's logic more intuitive and easier to maintain.

Here is a revised README that builds the project around the `browser-use` library.

***

# AI-Powered LinkedIn Deal Flow Agent (with browser-use)

This project creates an AI-powered agent that automates sourcing angel investment deals from your LinkedIn messages. It intelligently uses natural language to control a web browser, identifies potential investment opportunities in your inbox, extracts key information, and organizes it all in a Google Sheet.

This approach leverages the `browser-use` library to translate high-level tasks like "go to my messages and find me new deals" into concrete browser actions, dramatically simplifying the automation process.

## Features

-   **Natural Language Browser Control:** Utilizes `browser-use` to navigate and interact with LinkedIn using simple English prompts instead of brittle CSS selectors or XPath.
-   **Intelligent Deal Identification:** Employs a Large Language Model (LLM) to analyze message content and pinpoint conversations that represent potential investment opportunities.
-   **Structured Data Extraction:** Once a deal is identified, the agent extracts key data points like:
    -   Company Name
    -   Founder(s) & LinkedIn Profile URL
    -   Location
    -   Industry / Sector
    -   One-Line Pitch / Company Description
    -   Deal Type (e.g., Pre-seed, Seed)
    -   Stated Traction or Key Metrics
    -   Date of Inquiry
-   **Automated Google Sheets Logging:** Seamlessly populates a designated Google Sheet with the extracted data, creating a real-time, structured deal flow pipeline.
-   **High Readability & Maintainability:** The agent's core logic is written in easy-to-understand prompts, making it simple to modify or extend.

## High-Level Workflow

The agent operates in a clear, multi-step process:

1.  **Launch & Login:** The agent launches a browser and logs into your LinkedIn account using prompts.
2.  **Navigate to Messages:** It navigates to your LinkedIn messaging inbox with a simple command.
3.  **Iterate and Extract:** The agent iterates through your messages, using `browser-use` to extract the text content of each relevant conversation.
4.  **Analyze and Qualify:** The extracted text from each message is sent to an LLM (e.g., GPT-4) to determine if it's a cold inbound deal and to extract the structured data fields.
5.  **Store Data:** If a deal is qualified, the structured data is appended as a new row in your designated Google Sheet.

## Tech Stack

-   **Python:** The core programming language for the agent's logic.
-   **browser-use:** The primary library for AI-powered, natural language browser automation.
-   **Playwright:** The underlying browser automation framework used by `browser-use`.
-   **Large Language Model (LLM):** An API from a provider like OpenAI is required for both `browser-use` and the content analysis.
-   **Google Sheets API:** To programmatically send data to your spreadsheet.
-   **gspread:** A Python library to simplify interactions with the Google Sheets API.
-   **python-dotenv:** To manage environment variables for credentials and API keys.
-   **instructor (Optional but Recommended):** A library to get structured, Pydantic-validated output from LLMs, ensuring the data is clean.

## Prerequisites

-   A LinkedIn account.
-   An **OpenAI API key** (or another compatible LLM provider key) with access to a powerful model (e.g., GPT-4o).
-   Python 3.8+ and pip.
-   A Google Cloud Platform (GCP) project with the **Google Sheets API** and **Google Drive API** enabled.
-   A Service Account key from your GCP project.
-   A Google Sheet to act as your database.

## Installation and Setup

### 1. Clone the Repository

```bash
git clone https://github.com/potalora/linkedin-deal-flow-agent.git
cd linkedin-deal-flow-agent
```

### 2. Set Up a Virtual Environment

```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

### 3. Install Dependencies

Install the required Python packages.

```bash
pip install browser-use gspread oauth2client google-api-python-client python-dotenv openai instructor
```

Then, install the necessary browser binaries for Playwright.

```bash
playwright install
```

### 4. Configure Google Cloud & Sheets

1.  **Create Google Sheet:** Create a new Google Sheet. The first row should be the header with your desired fields (e.g., `Company Name`, `Founder`, `Description`, `Location`, etc.).
2.  **Enable APIs & Create Credentials:** Follow the official Google Cloud documentation to enable the Drive and Sheets APIs and download a `credentials.json` file for a service account.
3.  **Share Sheet:** Share your Google Sheet with the `client_email` found in your `credentials.json` file, granting it "Editor" permissions.
4.  **Place Credentials:** Move the downloaded `credentials.json` file to the root of your project directory.

### 5. Set Up Environment Variables

Create a `.env` file in the project's root directory. **This file should never be committed to version control.**

```
# LinkedIn Credentials
LINKEDIN_EMAIL="your_linkedin_email@example.com"
LINKEDIN_PASSWORD="your_linkedin_password"

# OpenAI API Key (Required for browser-use and analysis)
OPENAI_API_KEY="sk-..."

# Google Sheet Name
GOOGLE_SHEET_NAME="Your Google Sheet Title"
```

## Example Usage Logic

Your main Python script (`main.py`) would orchestrate the process. Here is a conceptual example of the core logic:

```python
import browser_use
import os
from dotenv import load_dotenv
from llm_analyzer import analyze_message_for_deal # A separate module you'd create
from gsheet_handler import save_to_gsheet       # A separate module you'd create

load_dotenv()

# browser-use will automatically use the OPENAI_API_KEY from the environment
b = browser_use.Browser()

# 1. Login to LinkedIn
login_prompt = f"""
Go to linkedin.com/login.
Type '{os.getenv("LINKEDIN_EMAIL")}' into the email or phone number field.
Type '{os.getenv("LINKEDIN_PASSWORD")}' into the password field.
Then click the 'Sign in' button.
"""
b.act(login_prompt)

# 2. Navigate to Messages
b.act("Click on the 'Messaging' icon in the header.")

# 3. Process Messages (Conceptual Loop)
# This part requires a strategy, e.g., get a list of message elements and loop through them.
b.act("Wait for the message list to load. Then, get a list of all message list items.")
message_elements = b.get("a list of all message list items") # Fictional high-level get

for msg_element in message_elements:
    b.act(f"Click on the message element described as: {msg_element.short_description}")
    
    # 4. Extract Text
    message_text = b.get("the full text content of the currently open conversation thread")
    
    # 5. Analyze with your separate LLM logic
    deal_data = analyze_message_for_deal(message_text)
    
    # 6. Save to Google Sheets if a deal is found
    if deal_data:
        print(f"Deal Found: {deal_data['company_name']}")
        save_to_gsheet(deal_data)

b.close()
```

## Disclaimer

-   **LinkedIn's Terms of Service:** Automating access to LinkedIn carries inherent risks. Use this agent responsibly, incorporate human-like delays, and be aware that it may violate LinkedIn's User Agreement. Use at your own risk.
-   **Data Privacy:** This agent processes your private message data. Ensure you understand the privacy implications. Securely manage your credentials and API keys using environment variables and a `.gitignore` file.

## Contributing

Contributions are welcome! If you have ideas for improving the prompts, the data extraction logic, or the overall efficiency of the agent, please feel free to fork the repository and submit a pull request.