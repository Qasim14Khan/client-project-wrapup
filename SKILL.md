---
name: client-project-wrapup
description: Client project wrap-up for photographers, editors and freelancers, producing a delivery folder, a delivery album and a project summary from files on the NAS. Use when the user says things like "wrap up the Ahmed wedding project", "prepare delivery for my client" or "close out this shoot".
---

# Goal

Help a photographer, editor or freelancer close out one client project in a single request. The result is a short project summary saved on the NAS, a delivery folder holding copies of the final files, and a delivery album for the photos, all based on files that really exist on the NAS.

Supported scope: one project at a time, using only files, photos, documents and notes already stored on the NAS. This Skill never deletes anything and never moves original files; it only copies files after the user confirms.

# Required input

Ask the user for these items in one short message and wait for the answers:

1. The client or project name, for example "Ahmed wedding".
2. Where the project files are stored on the NAS, or the date range of the shoot. If the user is unsure, ask the file expert to help the user pick the folder.
3. The name of the destination folder for the delivery. If the user gives none, propose "<project name> - Delivery" and wait for a yes.

Do not ask for anything else. Parameters belonging to the file, photo, document or knowledge experts are chosen by those experts.

# Generation-time capability evidence

- `ug-filesystem` → `file-agent`
- `ug-filemgr` → `file-agent`
- `ug-document` → `document-agent`
- `ug-photo` → `photo-agent`
- `ug-knowledge` → `knowledge-agent`

# Orchestration

Run the steps in this order. Each step uses only the real results the previous step returned. Never invent file names, paths, photo counts or amounts.

Step 1. Find the project files. Delegate to the file expert and record the real list of photos, videos, contracts and invoices it returns.

delegate --target file-agent --instruction 'Look only inside the project location or date range the user gave. List the photos, videos, PDF or office documents found there, grouped by type, with their real paths and counts. Read only: do not copy, move, rename or delete anything. If nothing matches, say so clearly and stop.'

If no files are returned, tell the user that nothing was found, ask them to check the location or date range, and stop. Do not search other folders.

Step 2. Read contracts and invoices. Only if Step 1 returned contracts or invoices, delegate to the document expert with the real paths from Step 1.

delegate --target document-agent --instruction 'Read only the contract and invoice files whose paths were returned by the file search for this project. Extract for each file: the document type, the parties, the total amount, the due date and any delivery deadline. Read only. If a value cannot be found in a file, write "not found" for it and do not guess.'

Report documents that could not be read as unread, and continue with the rest.

Step 3. Show the plan and ask for confirmation. Tell the user, in plain words: how many files will be copied into which delivery folder, and that a delivery album will be created for the project photos. Wait for an explicit yes. If the user says no, stop and report that nothing was changed.

Step 4. Create the delivery folder and copy the files. After the yes, delegate to the file expert.

delegate --target file-agent --instruction 'Create the delivery folder the user named and copy into it the final photos, videos and documents that the file search returned for this project. Copy only; keep every original where it is. Do not overwrite existing files: if a name already exists, report it and skip that file. Report which files were copied and which were skipped.'

Step 5. Create the delivery album. Delegate to the photo expert.

delegate --target photo-agent --instruction 'Create a photo album named after the project with the word Delivery added, using only the project photos that already exist on the NAS and match the date range or location the user gave. Report the album name and how many photos it contains. If no matching photos are found, report that and do not create an empty album.'

Step 6. Save the project summary. Compose a short summary in plain text from the real results of Steps 1, 2, 4 and 5: file counts by type, key contract and invoice details, what was copied, the album name, and anything skipped or not found. Then delegate to the knowledge expert.

delegate --target knowledge-agent --instruction 'Save the project summary text the user just approved as a text file inside the delivery folder, using the project name as the file name hint. Report the saved file path and confirm it was written.'

Finish by telling the user what was created and where, in a few lines, plus anything that was skipped.

# Completion and failure

For every delegation, read the returned state and handle it as follows:

- completed: report the real result and continue with the next step.
- failed: report what failed and which later steps were skipped because of it. Do not retry with a different folder or a different scope.
- cancelled: report that the step was cancelled and stop all steps that depend on it.
- waiting_user: ask the user the expert's question in plain words, then resume that step.
- waiting_approval: tell the user which change needs approval (creating the folder, copying files, creating the album or saving the summary) and wait. Never claim the change happened before it is completed.

Empty and partial results: if a search returns no results (zero matches), say so plainly, do not widen the scope or switch to similar data, and stop that branch. If only some files could be read or copied, report exactly which ones succeeded and which did not, and keep the successful results. Only report a business result after the expert returns completed.
