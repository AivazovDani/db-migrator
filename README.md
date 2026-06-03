# Taking-a-file-and-uploading-it-from-one-to-another-database

This automation is built with Selenium and the pyairtable library (mainly). The main idea is to take a file from a database and create a phone book (plus some other things) inside another database.

For these purposes we are going to use Airtable as our source, because the Airtable API key is easier to set up rather than Google Drive which needs OAuth2 verification, and it's easier to work with in general. We'll need only one function where we'll do everything since the script isn't that complex.

---

## How do we determine which files to download?

We use a checkbox inside Airtable:
- **Checkbox unchecked** → file needs to be downloaded
- **Checkbox checked** → file already processed, skip it

When a file is downloaded and uploaded successfully, we check the checkbox.

---

## How it works — step by step

1. Import the libraries we need
2. Create 4 constant variables for our URL and Airtable credentials
3. Inside our `fetch_and_upload` function:
   - Start Chrome with headless options so we don't open a browser window on our machine
   - Create an API and table instance to fetch the records we want by formula
   - Loop through the records and open the second database URL where we'll be uploading the files
   - Wait 2 seconds for the full content of the page to load
   - Get all files from the Attachments column in Airtable
   - Loop through them in case there are multiple files in one field
   - When we fetch a file from Airtable via API it comes with multiple JSON properties — we extract the download URL and the original filename
   - Send an HTTP GET request to that URL, just like when your browser visits a webpage — except instead of HTML back, we get the raw file content (bytes): `response = requests.get(url)`
   - To upload a file we need it locally on our machine, but we don't need it long-term after uploading, so we store it in a temp folder in our OS and delete it later
   - Create the file in the OS temp folder, pour the downloaded bytes into it with `.write()`, and save the full path into a variable — because once we exit the `with` block the file closes and `tmp` is no longer accessible
   - Check the checkbox in Airtable for that record
   - Delete the temp file: `os.remove(tmp_path)` ← really important not to skip this

---

## Upload path inside the platform (Selenium flow)

```
Sign In → Phone Book → New Book
→ Country: US
→ Name: {file_name}
→ Import Phones → Excel/CSV → Upload → Next → Next → Finish
```

---

## Main structure of the Selenium automation

- Log in to the platform **once** at the start — not on every file, since the browser saves the session
- Use `WebDriverWait` until elements appear all the way through the upload flow
- After the last step: check the checkbox in Airtable, remove the temp file, and reload the platform dashboard
- Everything is wrapped in a `try/except` so if one file fails it doesn't break the entire run — it just continues to the next file

---

## Tricks used

- Inputting credentials for the login page once so we don't have to log in every time
- Saving the file into a temp folder in the OS and deleting it after upload
- Reopening the browser dashboard at the end of each file so we reuse the session from the browser
- Error handling to prevent the whole script from crashing — instead it skips to the next file
- Using a checkbox to determine which file to process instead of relying on the date created
