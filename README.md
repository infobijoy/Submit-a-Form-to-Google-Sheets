# Submit-a-Form-to-Google-Sheets
How to create an HTML form that stores the submitted form data in Google Sheets using plain 'ol JavaScript (ES6), Google Apps Script, Fetch and FormData.

1. Create a new Google Sheet
First, go to Google Sheets and Start a new spreadsheet with the Blank template.
Rename it Email Subscribers. Or whatever, it doesn't matter.
Put the following headers into the first row:





2. Create a Google Apps Script
Click on Tools > Script Editor… which should open a new tab.
Rename it Submit Form to Google Sheets. Make sure to wait for it to actually save and update the title before editing the script.
Now, delete the function myFunction() {} block within the Code.gs tab.
Paste the following script in it's place and File > Save:


var sheetName = 'Sheet1'
var scriptProp = PropertiesService.getScriptProperties()

function intialSetup () {
  var activeSpreadsheet = SpreadsheetApp.getActiveSpreadsheet()
  scriptProp.setProperty('key', activeSpreadsheet.getId())
}

function doPost (e) {
  var lock = LockService.getScriptLock()
  lock.tryLock(10000)

  try {
    var doc = SpreadsheetApp.openById(scriptProp.getProperty('key'))
    var sheet = doc.getSheetByName(sheetName)

    var headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0]
    var nextRow = sheet.getLastRow() + 1

    var newRow = headers.map(function(header) {
      return header === 'timestamp' ? new Date() : e.parameter[header]
    })

    sheet.getRange(nextRow, 1, 1, newRow.length).setValues([newRow])

    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'success', 'row': nextRow }))
      .setMimeType(ContentService.MimeType.JSON)
  }

  catch (e) {
    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'error', 'error': e }))
      .setMimeType(ContentService.MimeType.JSON)
  }

  finally {
    lock.releaseLock()
  }
}






3. Run the setup function
Next, go to Run > Run Function > initialSetup to run this function.
In the Authorization Required dialog, click on Review Permissions.
Sign in or pick the Google account associated with this projects.
You should see a dialog that says Hi {Your Name}, Submit Form to Google Sheets wants to...
Click Allow
4. Add a new project trigger
Click on Edit > Current project’s triggers.
In the dialog click No triggers set up. Click here to add one now.
In the dropdowns select doPost
Set the events fields to From spreadsheet and On form submit
Then click Save
5. Publish the project as a web app
Click on Publish > Deploy as web app….
Set Project Version to New and put initial version in the input field below.
Leave Execute the app as: set to Me(your@address.com).
For Who has access to the app: select Anyone, even anonymous.
Click Deploy.
In the popup, copy the Current web app URL from the dialog.
And click OK.
IMPORTANT! If you have a custom domain with Gmail, you might need to click OK, refresh the page, and then go to Publish > Deploy as web app… again to get the proper web app URL. It should look something like https://script.google.com/a/yourdomain.com/macros/s/XXXX….








HTML script
6. Input your web app URL
Open the file named index.html. On line 12 replace <SCRIPT URL> with your script url:
<script>
      const scriptURL = 'https://script.google.com/macros/s/AKfycbzb6siFbce3PvpgXR2xTJ-DAXQtY5Apa8MC2QKxvfriv424VFC57PJ-edvW0W7XUwNuCA/exec'
      const form = document.forms['submit-to-google-sheet']
      const msg = document.getElementById("msg")
    
      form.addEventListener('submit', e => {
        e.preventDefault()
        fetch(scriptURL, { method: 'POST', body: new FormData(form)})
          .then(response => {
            msg.innerHTML = "Sorry your account not available this update. Try again after a few days"
            setTimeout(function(){
              msg.innerHTML =""
            }, 5000)
            form.reset()
          })
          .catch(error => console.error('Error!', error.message))
      })
    </script>
