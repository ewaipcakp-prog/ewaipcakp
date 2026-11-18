# MCQEXAM Admin — Exam Import

Exam Papers → Import Exam Paper → select JSON/CSV → preview → Save Draft to Firestore → review Questions → Publish.

JSON format:
```json
{
  "subject": "Science", "title": "Science – 2026", "year": 2026,
  "grade": "Grade 10", "language": "Tamil", "duration": 60,
  "questions": [
    {"text":"Question?", "options":["A1","B1","C1","D1"], "correctIndex":1,"explanation":"Explanation"}
  ]
}
```

CSV headers: `text,optionA,optionB,optionC,optionD,correctIndex,explanation` (correctIndex 0-3 or A-D). Use UTF-8, quote fields containing commas or line breaks. Fill paper metadata in import dialog.

The imported paper is a new unpublished draft. Firestore rules must permit active admins to write papers and questions. No service account key is needed in the browser. This import does not translate questions automatically.

## Deploy (PowerShell)
From extracted `mcqexam_admin_2026` folder: `firebase.cmd deploy --only hosting --project mcq-exam-project`.

IMPORTANT: Deploy hosting only. Do not deploy included firestore.rules because live rules may have been updated since this ZIP was made. Back up your current live hosting source first.
