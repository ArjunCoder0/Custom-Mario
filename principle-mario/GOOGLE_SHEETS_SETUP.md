# Google Sheets Leaderboard Setup

## How to Set Up Google Sheets for Leaderboard Storage

### Step 1: Create a Google Sheet
1. Go to [Google Sheets](https://sheets.google.com)
2. Create a new spreadsheet named "Custom Mario Leaderboard"
3. In the first row, add these headers:
   - A1: `name`
   - B1: `score`
   - C1: `distance`
   - D1: `coins`
   - E1: `date`

### Step 2: Create Google Apps Script
1. In your Google Sheet, go to **Extensions** → **Apps Script**
2. Delete the default code and paste this:

```javascript
function doPost(e) {
  try {
    const sheet = SpreadsheetApp.getActiveSheet();
    const data = JSON.parse(e.postData.contents);
    
    // Add the new entry
    sheet.appendRow([
      data.name,
      data.score,
      data.distance,
      data.coins,
      data.date
    ]);
    
    return ContentService.createTextOutput(JSON.stringify({success: true}))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (error) {
    return ContentService.createTextOutput(JSON.stringify({success: false, error: error.toString()}))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function doGet(e) {
  try {
    const sheet = SpreadsheetApp.getActiveSheet();
    const data = sheet.getDataRange().getValues();
    
    // Skip header row and convert to JSON
    const leaderboard = data.slice(1).map(row => ({
      name: row[0],
      score: row[1],
      distance: row[2],
      coins: row[3],
      date: row[4]
    })).filter(entry => entry.name); // Filter out empty rows
    
    // Sort by score descending and keep top 10
    leaderboard.sort((a, b) => b.score - a.score);
    
    return ContentService.createTextOutput(JSON.stringify(leaderboard.slice(0, 10)))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (error) {
    return ContentService.createTextOutput(JSON.stringify({error: error.toString()}))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

3. Click **Deploy** → **New Deployment**
4. Select **Type: Web app**
5. Set:
   - Execute as: Your email
   - Who has access: Anyone
6. Click **Deploy**
7. Copy the deployment URL (it will look like: `https://script.google.com/macros/d/AKfycbw.../ususercontent`)

### Step 3: Update Game Code
1. Open `game.js` in your project
2. Find this line:
   ```javascript
   const GOOGLE_SHEETS_API = 'https://script.google.com/macros/d/AKfycbwXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/usercontent';
   ```
3. Replace it with your deployment URL

### Step 4: Test
1. Deploy the game to Vercel
2. Enter your name and play
3. When you die, your score should be saved to Google Sheets
4. Check your Google Sheet - you should see your entry!

### Features:
- ✅ Scores stored in Google Sheets
- ✅ Visible in a spreadsheet
- ✅ Automatic sorting by score
- ✅ Top 10 leaderboard
- ✅ Fallback to localStorage if API fails

### Troubleshooting:
- If scores aren't saving, check the browser console (F12) for errors
- Make sure the Google Apps Script is deployed as "Web app"
- Make sure "Who has access" is set to "Anyone"
- The game will automatically fallback to localStorage if Google Sheets fails
