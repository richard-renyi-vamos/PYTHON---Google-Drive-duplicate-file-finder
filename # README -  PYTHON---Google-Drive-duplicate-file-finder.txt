CHATGPT CONVERSATION

***********************

create a PYTHON code that checks Google drive for duplicates


ChatGPT said:
Here’s a Python script that checks for duplicate files in a Google Drive account using the Google Drive API. It scans files by name and hash (content) to detect duplicates.

Google Drive Duplicates

Answer in chat instead
How It Works:
Authentication: Uses a service account JSON file for authentication.

Listing Files: Fetches all files from Google Drive.

Detecting Duplicates:

Compares file names to find duplicates.

Computes file hashes (MD5) to detect content duplicates.

Reporting Duplicates: Prints duplicate files found.

You'll need to replace 'path/to/your/service-account.json' with your actual service account credentials. Let me know if you need help setting that up! 🚀🐍

***********************

from googleapiclient.discovery import build
from googleapiclient.http import MediaIoBaseDownload
from google.oauth2 import service_account
import io
import hashlib

# Set up Google Drive API authentication
SCOPES = ['https://www.googleapis.com/auth/drive']
SERVICE_ACCOUNT_FILE = 'path/to/your/service-account.json'

creds = service_account.Credentials.from_service_account_file(
    SERVICE_ACCOUNT_FILE, scopes=SCOPES)

drive_service = build('drive', 'v3', credentials=creds)

def get_file_hash(file_id):
    """Download file content and compute hash."""
    request = drive_service.files().get_media(fileId=file_id)
    file_stream = io.BytesIO()
    downloader = MediaIoBaseDownload(file_stream, request)
    done = False
    while not done:
        status, done = downloader.next_chunk()
    file_stream.seek(0)
    return hashlib.md5(file_stream.read()).hexdigest()

def find_duplicates():
    """Find duplicate files in Google Drive."""
    results = drive_service.files().list(
        pageSize=1000, fields="files(id, name, mimeType)").execute()
    files = results.get('files', [])
    
    name_dict = {}
    hash_dict = {}
    duplicates = []
    
    for file in files:
        file_id = file['id']
        file_name = file['name']
        
        if file_name in name_dict:
            duplicates.append((file_name, file_id, name_dict[file_name]))
        else:
            name_dict[file_name] = file_id
        
        file_hash = get_file_hash(file_id)
        if file_hash in hash_dict:
            duplicates.append((file_name, file_id, hash_dict[file_hash]))
        else:
            hash_dict[file_hash] = file_id
    
    if duplicates:
        print("Duplicate files found:")
        for dup in duplicates:
            print(f"Name: {dup[0]}, ID1: {dup[1]}, ID2: {dup[2]}")
    else:
        print("No duplicates found.")

if __name__ == "__main__":
    find_duplicates()
