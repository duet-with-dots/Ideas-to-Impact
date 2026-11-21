Create an n8n workflow called "Resume Fitment Analyzer" that takes two uploaded files, a Word resume (.docx) and a job description (.txt), and returns a weighted-skills fitment rubric to the user in the browser.

TRIGGER
- Use a Form Trigger with the path "resume-fitment-analyzer".
- Add two required single-file upload fields: "resume" (accept .docx only) and "job_description" (accept .txt only).
- Set the form's response mode to "When form is submitted" (onReceived). Do NOT use "Using Response Node" (that mode needs a Respond to Webhook node, which this workflow does not have).
- End the workflow with a Form node, operation "Completion", respond with "Show Text", and show the final HTML report.

STEPS (linear)
1. Code node "Read Job Description":
   - Read the "job_description" binary as UTF-8 text into json.jdText.
   - Pass the resume binary on as binary property "data".
   - Relabel that binary as a zip, because n8n's Compression node rejects the ".docx" extension even though a docx is a zip archive. Set fileName "resume.zip", fileExtension "zip" and mimeType "application/zip".
2. Compression node "Unzip Resume (docx)": operation Decompress, binary property "data", output prefix "file_".
3. Code node "Extract Resume Text":
   - Find the unzipped binary whose fileName ends with "word/document.xml". Throw a clear error if it is not found.
   - Convert the XML to plain text. Turn </w:p> into newlines, <w:tab/> into tabs and <w:br/> into newlines. Strip the remaining tags, decode the XML entities (&amp; &lt; &gt; &quot; &apos;) and collapse 3 or more blank lines.
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
