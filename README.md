# Diesel Price Dashboard

Live diesel price dashboard for Rajasthan and Punjab from a single Google Sheet.

## Setup (5 Minutes)

### 1. Prepare Your Google Sheet

Create one Google Sheet with two tabs:
- **Tab 1 name:** `Rajasthan`
- **Tab 2 name:** `Punjab`

Each tab has 3 columns:
| Column A | Column B | Column C |
|----------|----------|----------|
| Date | Price (₹) | Image URL |
| 2026-04-23 | 91.75 | https://example.com/bill.jpg |

### 2. Deploy Apps Script

1. Open your Google Sheet
2. Click **Extensions** → **Apps Script**
3. Delete default code and paste this:

```javascript
function doGet() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const rajasthanSheet = ss.getSheetByName('Rajasthan');
  const punjabSheet = ss.getSheetByName('Punjab');
  
  function getSheetData(sheet) {
    const data = sheet.getDataRange().getValues();
    const result = [];
    for (let i = 1; i < data.length; i++) {
      const date = data[i][0] instanceof Date ? data[i][0].toISOString().split('T')[0] : data[i][0];
      const price = parseFloat(data[i][1]);
      let image = data[i][2];
      if (image && typeof image === 'string' && image.includes('=IMAGE')) {
        const match = image.match(/=IMAGE\("([^"]+)"\)/);
        if (match) image = match[1];
      }
      if (date && !isNaN(price)) result.push({ date, price, image: image || null });
    }
    return result;
  }
  
  const result = {
    rajasthan: getSheetData(rajasthanSheet),
    punjab: getSheetData(punjabSheet)
  };
  
  return ContentService.createTextOutput(JSON.stringify(result))
    .setMimeType(ContentService.MimeType.JSON);
}