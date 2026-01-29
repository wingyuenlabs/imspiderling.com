---
Title: Solved: Sync HubSpot Deals to a Google Sheet for Sales Visualization
Description: 
Author: Darian Vance
Date: 2026-01-29T21:11:18.000Z
Robots: noindex,nofollow
Template: index
---
<h3>
  
  
  🚀 Executive Summary
</h3>

<p><strong>TL;DR:</strong> The article addresses the pain point of manually exporting stale HubSpot deal data to Google Sheets for sales visualization. It provides a Python script solution that automates fetching live deal data from the HubSpot API and pushing it directly into a Google Sheet, enabling real-time dashboards.</p>

<h4>
  
  
  🎯 Key Takeaways
</h4>

<ul>
<li>HubSpot Private Apps are the recommended secure method for API access, requiring the <code>crm.objects.deals.read</code> scope for fetching deal data.</li>
<li>Google Cloud Platform Service Accounts with JSON keys are used to authenticate and grant programmatic ‘Editor’ access to the Google Sheets API and Google Drive API.</li>
<li>The Python script utilizes <code>requests</code> for HubSpot API calls and <code>google-api-python-client</code> for Google Sheets API interactions, with <code>python-dotenv</code> for secure environment variable management.</li>
<li>Data synchronization involves clearing the existing Google Sheet data before writing new deal information to ensure data freshness and prevent stale rows.</li>
<li>Automated scheduling via cron jobs (e.g., <code>0 2 \* \* 1 python3 sync\_deals.py</code>) ensures periodic, hands-off data updates for continuous sales visualization.</li>
</ul>

<h1>
  
  
  Sync HubSpot Deals to a Google Sheet for Sales Visualization
</h1>

<p>Hey folks, Darian Vance here. Let’s talk about a common pain point: getting timely sales data in front of the right people without the manual drudgery. For a while, our sales ops team was manually exporting HubSpot deal data to a CSV and then importing it into a Google Sheet every Monday morning. It was a tedious process, prone to errors, and the data was instantly stale. The leadership team wanted a live dashboard, but didn’t want another login to remember.</p>

<p>I built a simple Python script to automate this entire workflow. It pulls deal data from the HubSpot API and pushes it directly into a designated Google Sheet. That sheet then powers a Looker Studio dashboard that’s always up-to-date. This little automation saves us a few hours a week and has completely eliminated the manual reporting step. It’s a classic DevOps win: automate the boring stuff so your team can focus on what matters.</p>

<p>Today, I’m going to walk you through how to build it yourself.</p>

<h3>
  
  
  Prerequisites
</h3>

<p>Before we dive in, make sure you have the following ready. We’re busy people, so getting this sorted out first will save you a headache later.</p>

<ul>
<li>
<strong>HubSpot Account:</strong> You’ll need Admin permissions to create a Private App.</li>
<li>
<strong>Google Cloud Platform (GCP) Account:</strong> A project with the Google Sheets API and Google Drive API enabled is essential.</li>
<li>
<strong>Python Environment:</strong> Python 3 installed on whatever machine will run the script.</li>
<li>
<strong>Required Python Libraries:</strong> You’ll need to install a few packages. I usually run these commands in my terminal: <code>pip install requests python-dotenv google-api-python-client google-auth-httplib2 google-auth-oauthlib</code>.</li>
</ul>

<h3>
  
  
  The Step-by-Step Guide
</h3>

<h4>
  
  
  Step 1: Create a HubSpot Private App
</h4>

<p>First, we need to get an API key from HubSpot. Their preferred method is using Private Apps, which is more secure than the old API keys.</p>

<ol>
<li>Log into your HubSpot account and navigate to <strong>Settings</strong> (the gear icon) &gt; <strong>Integrations</strong> &gt; <strong>Private Apps</strong>.</li>
<li>Click “Create a private app” and give it a descriptive name like “GoogleSheets_Deal_Sync”.</li>
<li>Go to the <strong>Scopes</strong> tab. This is where you grant permissions. For this task, you only need one: <code>crm.objects.deals.read</code>. Always follow the principle of least privilege.</li>
<li>Click “Create app”. HubSpot will show you your app’s Access Token. <strong>Copy this token immediately.</strong> This is your API key, so treat it like a password. We’ll store it securely in a moment.</li>
</ol>

<h4>
  
  
  Step 2: Configure Your Google Cloud Service Account
</h4>

<p>Next, we need to give our script permission to talk to Google Sheets. We’ll do this using a Service Account, which is like a robot user for your GCP project.</p>

<ol>
<li>In the <a href="https://console.cloud.google.com/" rel="noopener noreferrer">Google Cloud Console</a>, select your project.</li>
<li>Navigate to <strong>APIs &amp; Services &gt; Library</strong>. Search for and enable the “Google Sheets API” and “Google Drive API”.</li>
<li>Go to <strong>APIs &amp; Services &gt; Credentials</strong>. Click “Create Credentials” and select “Service Account”.</li>
<li>Give it a name like “hubspot-sheet-updater” and click “Create and Continue”, then “Done”.</li>
<li>Find your newly created service account in the list and click on it. Go to the <strong>Keys</strong> tab, click “Add Key”, and choose “Create new key”. Select <strong>JSON</strong> as the key type. A <code>credentials.json</code> file will download automatically.</li>
</ol>

<p><strong>Crucially:</strong> Keep this JSON file secure. It grants access to your Google account services. Don’t commit it to a public Git repository.</p>

<h4>
  
  
  Step 3: Prepare the Google Sheet
</h4>

<p>This is the easy part. Create a new Google Sheet that will act as our data destination.</p>

<ol>
<li>Create a new Google Sheet. I’ll call mine “Sales Deal Pipeline”.</li>
<li>Give the first worksheet a simple name, like “HubSpotDeals”.</li>
<li>Open your <code>credentials.json</code> file and find the <code>client_email</code> address (it looks something like <code>your-service-account@your-project.iam.gserviceaccount.com</code>).</li>
<li>Back in your Google Sheet, click the <strong>Share</strong> button and paste that service account email address. Grant it <strong>Editor</strong> permissions. This is a critical step that’s easy to forget.</li>
<li>Finally, copy the <strong>Spreadsheet ID</strong> from your browser’s URL bar. It’s the long string of characters between <code>/d/</code> and <code>/edit</code>.</li>
</ol>

<h4>
  
  
  Step 4: The Python Script
</h4>

<p>Alright, let’s get to the code. I’ll skip the standard project setup steps like creating a directory and a virtual environment, since you probably have your own workflow for that. Let’s focus on the logic.</p>

<p>Create two files in your project directory: a script file named <code>sync_deals.py</code> and a configuration file named <code>config.env</code> to hold our secrets.</p>

<p>Your <code>config.env</code> file should look like this. Populate it with the credentials you gathered in the previous steps.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># config.env
HUBSPOT_API_KEY="your-private-app-token-here"
GOOGLE_SHEET_ID="your-spreadsheet-id-here"
GOOGLE_SHEET_NAME="HubSpotDeals"
</code></pre>

</div>



<p>Now for the main event, the <code>sync_deals.py</code> script. I’ve broken it down into functions to keep things clean.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># sync_deals.py
import os
import requests
from dotenv import load_dotenv
from google.oauth2 import service_account
from googleapiclient.discovery import build

# Load environment variables from config.env
load_dotenv('config.env')

# --- CONFIGURATION ---
HUBSPOT_TOKEN = os.getenv('HUBSPOT_API_KEY')
GOOGLE_SHEET_ID = os.getenv('GOOGLE_SHEET_ID')
GOOGLE_SHEET_NAME = os.getenv('GOOGLE_SHEET_NAME')
SERVICE_ACCOUNT_FILE = 'credentials.json' # Assumes the JSON key is in the same directory
GOOGLE_SCOPES = ['https://www.googleapis.com/auth/spreadsheets']

def get_hubspot_deals():
    """Fetches a list of deals from the HubSpot API."""
    print("Fetching deals from HubSpot...")

    headers = {
        'Authorization': f'Bearer {HUBSPOT_TOKEN}',
        'Content-Type': 'application/json'
    }

    # Define the properties you want to pull from each deal
    properties = "dealname,amount,dealstage,closedate,hubspot_owner_id"
    url = f"https://api.hubapi.com/crm/v3/objects/deals?properties={properties}&amp;limit=100"

    try:
        response = requests.get(url, headers=headers)
        response.raise_for_status()  # This will raise an HTTPError for bad responses (4xx or 5xx)

        deals_json = response.json().get('results', [])
        if not deals_json:
            print("No deals found.")
            return None

        # Prepare data for Google Sheets: Header row + data rows
        headers = ["Deal Name", "Amount", "Deal Stage", "Close Date", "Owner ID"]
        processed_deals = [headers]

        for deal in deals_json:
            props = deal.get('properties', {})
            row = [
                props.get('dealname', 'N/A'),
                props.get('amount', '0'),
                props.get('dealstage', 'N/A'),
                props.get('closedate', 'N/A'),
                props.get('hubspot_owner_id', 'N/A')
            ]
            processed_deals.append(row)

        print(f"Successfully fetched {len(processed_deals) - 1} deals.")
        return processed_deals

    except requests.exceptions.RequestException as e:
        print(f"Error fetching from HubSpot: {e}")
        return None

def update_google_sheet(data):
    """Clears and updates a Google Sheet with new data."""
    print("Connecting to Google Sheets API...")

    creds = service_account.Credentials.from_service_account_file(
        SERVICE_ACCOUNT_FILE, scopes=GOOGLE_SCOPES)

    try:
        service = build('sheets', 'v4', credentials=creds)
        sheet = service.spreadsheets()

        # 1. Clear the existing data to prevent stale rows
        print(f"Clearing sheet: '{GOOGLE_SHEET_NAME}'...")
        sheet.values().clear(
            spreadsheetId=GOOGLE_SHEET_ID,
            range=GOOGLE_SHEET_NAME
        ).execute()

        # 2. Write the new data
        print(f"Writing {len(data)} rows to the sheet...")
        body = {'values': data}
        sheet.values().update(
            spreadsheetId=GOOGLE_SHEET_ID,
            range=f"{GOOGLE_SHEET_NAME}!A1",
            valueInputOption="USER_ENTERED",
            body=body
        ).execute()

        print("Google Sheet updated successfully.")

    except Exception as e:
        print(f"An error occurred with Google Sheets API: {e}")

if __name__ == "__main__":
    print("--- Starting HubSpot to Google Sheets Sync ---")
    deal_data = get_hubspot_deals()

    if deal_data:
        update_google_sheet(deal_data)
    else:
        print("No data fetched from HubSpot. Skipping sheet update.")

    print("--- Sync complete ---")
</code></pre>

</div>



<blockquote>
<p><strong>Pro Tip:</strong> In my production setups, I abstract the API clients into their own classes for better organization and testing. I also implement more robust pagination for HubSpot, as this basic script only fetches the first 100 deals. You can do this by checking the <code>paging</code> object in the HubSpot API response and making subsequent requests until it’s empty.</p>
</blockquote>

<h4>
  
  
  Step 5: Schedule the Sync with Cron
</h4>

<p>A script is only useful if it runs automatically. On a Linux server, a cron job is the standard way to do this. You can schedule the script to run at any interval you like.</p>

<p>For example, to run the script every Monday at 2 AM, you would set up a cron job like this:</p>

<p><code>0 2 * * 1 python3 sync_deals.py</code></p>

<p>This simple command ensures your data is refreshed weekly without any manual intervention.</p>

<h3>
  
  
  Where I Usually Mess Up (Common Pitfalls)
</h3>

<p>Even simple integrations have their “gotchas.” Here are a few I’ve run into:</p>

<ul>
<li>
<strong>Forgetting to Share the Google Sheet:</strong> This is the number one cause of failure. If the service account doesn’t have “Editor” access to the sheet, the API will throw a permissions error. Always check this first.</li>
<li>
<strong>Incorrect HubSpot Scopes:</strong> If you get a 403 Forbidden error from HubSpot, it’s almost certainly because your Private App doesn’t have the right permissions. Double-check that <code>crm.objects.deals.read</code> is enabled.</li>
<li>
<strong>Environment Variables Not Loading:</strong> Make sure your <code>config.env</code> file is in the same directory as your script, or provide the correct path to <code>load_dotenv()</code>. And please, don’t commit your <code>config.env</code> or <code>credentials.json</code> files to version control. Use a <code>.gitignore</code> file.</li>
</ul>

<h3>
  
  
  Conclusion
</h3>

<p>And that’s it. You now have a resilient, automated pipeline for getting key sales data out of HubSpot and into a format perfect for visualization and wider business reporting. This is a foundational piece of automation that you can expand upon—syncing contacts, tickets, or custom objects is just a matter of changing the API endpoint and properties.</p>

<p>By investing a little time upfront, you free up your team to analyze data instead of just moving it around. Hope this helps you out.</p>

<p>Cheers,<br><br>
Darian Vance</p>




<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnwjgilechjb8hqoqlrg1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnwjgilechjb8hqoqlrg1.png" alt="Darian Vance" width="758" height="289"></a></p>

<p>👉 <a href="https://wp.me/pbK4oa-fS" rel="noopener noreferrer">Read the original article on TechResolve.blog</a></p>




<p>☕ <strong>Support my work</strong>  </p>

<p>If this article helped you, you can buy me a coffee:  </p>

<p>👉 <a href="https://buymeacoffee.com/darianvance" rel="noopener noreferrer">https://buymeacoffee.com/darianvance</a></p>

