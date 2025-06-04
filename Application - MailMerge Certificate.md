
# Mail Merge Certificate Automation

1. Download the template of certificate [Download Certificate file](samples/sample_cert.png)
2. Create a Google Slide and insert **Text Box** and format it accordingly.
3. Then, convert the Text Box text into {{content_description}} format. Make sure the conttent_description follow the column name of the Google Sheet.
4. Get the Google Slide ID from the URL. Eg. https://docs.google.com/presentation/d/**1Pb2PAla7tRyjMyvjh66bp_OPYFs1qnohaDQc3k-IJwQ**/edit#slide=id.p1 and set it as **templateId**
5. Determine the folder you want to put the file as **folderId**    
6. Modify the **templateId** and **folderId**. Then run the following code.
7. You will get the generated pdf files in the folder. Check and ensure everything is in order.

<details style="margin-bottom: 20px;">
  <summary><span style="font-weight: bold;">Sample code:</span></summary>

```js
const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
const data = sheet.getDataRange().getValues();
const headers = data[0]; // First row for placeholders
const templateId = '1Pb2PAla7tRyjMyvjh66bp_OPYFs1qnohaDQc3k-IJwQ' // Replace with your Google Slides Template ID
const folderId = '1T8H5cbn_wShcoPTBT_Fa7e0AZUst7S2V'; // Replace with your target folder ID

const dataObj = data.slice(1).map(row => {
  let obj = {};
  headers.forEach((header, index) => {
    obj[header] = row[index]; // Map each header to its corresponding value in the row
  });
  return obj;
});

function generateAndSavePDFs() {

  const folder = DriveApp.getFolderById(folderId);

  dataObj.forEach((row, index) => {
    const workerName = row['Name'];
    const workerId = row['Worker ID'];

    // Create a copy of the template
    const copy = DriveApp.getFileById(templateId).makeCopy(folder); // Replace 'xxx' with your Google Slides Template ID

    // Rename the copied file
    const newFileName = `${workerName}_${workerId}.pdf`;
    copy.setName(newFileName);

    // Open the copied presentation and replace placeholders
    const presentation = SlidesApp.openById(copy.getId());
    const slides = presentation.getSlides();

    slides.forEach(slide => {
      headers.forEach((header) => {
        const textToReplace = `{{${header}}}`;
        const value = row[header] || '';
        slide.replaceAllText(textToReplace, value);
      });
    });
    presentation.saveAndClose();

    // Get the file URL and file ID
    const fileUrl = copy.getUrl();
    const fileId = copy.getId();

    // Update the Google Sheet in columns G and H
    sheet.getRange(index + 2, 7).setValue(fileUrl); // Column G (File URL)
    sheet.getRange(index + 2, 8).setValue(fileId);  // Column H (File ID)

    // Optionally log the process
    Logger.log(`Generated file for ${workerName} (${workerId}): ${fileUrl}`);
  });
}
```
</details>

The prompt involved is 

```
create a google app script function, generate the pdf file using the template with templateID=xxx. Save the file with file name `${workerName}_${workerId}.pdf and put in the folder with folderID = xxxx. Get the url of the file, and fileID, then put in column G and H respectively in google sheet.
```


8. Then send the email using the following code.

<details style="margin-bottom: 20px;">
  <summary><span style="font-weight: bold;">Sample code:</span></summary>

```js
const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();

function sendCertificates() {
    const data = sheet.getDataRange().getValues(); // Get all data from the sheet
  
    const nameColumnIndex = 2; // Column B
    const emailColumnIndex = 3; // Column C
    const fileIdColumnIndex = 8; // Column H
    const statusColumnIndex = 9; // Column I
  
    for (let i = 1; i < data.length; i++) { // Start from row 2 (skip headers)
      const name = data[i][nameColumnIndex - 1]; // Name in column B
      const email = data[i][emailColumnIndex - 1]; // Email in column C
      const fileId = data[i][fileIdColumnIndex - 1]; // File ID in column H
  
      if (email && fileId) { // Ensure both email and file ID exist
        try {
          const file = DriveApp.getFileById(fileId); // Get the file by ID
          const subject = "Certificate of Attendance";
          const body = `
            Dear ${name},<br><br>
            Please find attached your Certificate of Attendance.<br><br>
            Thank you for attending.<br><br>
            Best regards,<br>
            Event Organizer
          `;
  
          // Send the email with the PDF attachment
          GmailApp.sendEmail(email, subject, "", {
            htmlBody: body,
            attachments: [file.getAs(MimeType.PDF)]
          });
  
          // Update the status in column I
          sheet.getRange(i + 1, statusColumnIndex).setValue("Sent");
          Logger.log(`Email sent to ${email} with file ID ${fileId}`);
        } catch (error) {
          // Log the error and update the status in column I
          Logger.log(`Failed to send email to ${email}: ${error.message}`);
          sheet.getRange(i + 1, statusColumnIndex).setValue("Failed");
        }
      } else {
        // If email or file ID is missing, update the status in column I
        sheet.getRange(i + 1, statusColumnIndex).setValue("Missing Data");
      }
    }
  }
```
</details>

The prompt involved is 

```
create a new function. get the fileid from column H and the name of participant from column B of the current google sheet. the file is pdf file of a certificate of attendance. Send it to the email stated in column C. Then put the status of sending in Column I.
```