---

Create an n8n workflow called "Resume Fitment Analyzer-beta" that takes two uploaded files, a Word resume (.docx) and a job description (.txt), and returns a weighted-skills fitment rubric to the user in the browser.

TRIGGER
- Use a Form Trigger with the path "resume-fitment-analyzer".
- Add two required single-file upload fields: "resume" (accept .docx only) and "job_description" (accept .txt only).
- Set the form's response mode to "When form is submitted" (onReceived). Do NOT use "Using Response Node" (that mode needs a Respond to Webhook node, which this workflow does not have).
- End the workflow with a Form node, operation "Completion", respond with "Show Text", and show the final HTML report.

STEPS (linear)
1. Code node "Read Job Description":
   - Read the "job_description" binary as UTF-8 text into json.jdText.
   - Read the "resume" binary into a buffer with `this.helpers.getBinaryDataBuffer(0, 'resume')`. Then create a brand-new binary from that buffer with `this.helpers.prepareBinaryData(buffer, 'resume.zip', 'application/zip')`. Output it as binary property "data". Do NOT copy or spread the original resume binary object and override its fields: the Compression node still rejects it as ".docx".
   - This relabels the docx as a zip, because n8n's Compression node rejects the ".docx" extension even though a docx is a zip archive. Also set fileName "resume.zip", fileExtension "zip" and mimeType "application/zip" on the new binary.
2. Compression node "Unzip Resume (docx)": operation Decompress, binary property "data", output prefix "file_".
3. Code node "Extract Resume Text":
   - The Compression node stores each archive entry under its base file name only (for example `document.xml`) and puts the folder in a separate `directory` field (for example `word`). So the fileName never ends with "word/document.xml" on its own.
   - For each unzipped binary, build the full path as `directory + '/' + fileName`, with leading slashes stripped, and find the one that ends with "word/document.xml". Also accept a fileName that already ends with "word/document.xml".
   - If it is not found, throw a clear error that lists the entry paths that were found.
   - Convert the XML to plain text. Turn </w:p> into newlines, <w:tab/> into tabs and <w:br/> into newlines. Strip the remaining tags, decode the XML entities (&amp; &lt; &gt; &quot; &apos;) and collapse 3 or more blank lines.
   - Write regexes so they survive Code-node string escaping. For example, use a character class like `/^[/]+/` instead of `/^\/+/`.
   - Output json.resumeText, and carry jdText forward from the "Read Job Description" node.
   - Use no external libraries or network calls.
4. Basic LLM Chain "Extract Weighted Skills":
   - The input is the resume text.
   - System prompt: act as a technical recruiter and list every distinct skill. Give each a weight from 1 to 100 that reflects how strongly the resume evidences it (years, recency, depth, achievements). The weights must sum to 100.
   - Return only JSON in the form {"skills":[{"skill","category","weight","evidence"}]}, sorted by weight descending, with no markdown fences.
5. Basic LLM Chain "Build Fitment Rubric":
   - The input is the weighted skills JSON plus the job description (jdText from "Extract Resume Text").
   - Output a clean HTML fragment, with no markdown and no html/body tags, containing:
     (a) an overall score out of 100 and a verdict (Strong / Good / Partial / Weak Fit);
     (b) a rubric table with columns Criterion, JD Weight %, Resume Evidence, Score 0-10, Weighted Score and Rationale. Cover required technical skills, preferred skills, experience level, domain relevance, leadership and soft skills, education, and achievements;
     (c) matched skills;
     (d) gaps;
     (e) extra strengths;
     (f) risks;
     (g) a recommendation and interview questions.
   - Cite evidence only from the inputs.
6. Form Completion node "Show Fitment Report": show the rubric HTML ({{ $json.text }}).

MODELS
- Both chains use one OpenAI chat model each. Use a currently available model, not a deprecated one.
- Use Gateway credits if available. Do not enable built-in tools or the Responses API.

NOTES
- Verification can't simulate file uploads, so test by submitting real files through the form.
- To debug the Code nodes without re-uploading, replay a failed execution through the single node. A Code node with no network calls is safe to re-run.

---
