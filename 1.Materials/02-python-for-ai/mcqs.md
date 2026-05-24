# Module 02 — MCQ Quiz

**Instructions:** Choose the best answer.

---

**Q1.** Which command creates a Python virtual environment named `venv`?

- A) `pip install venv`
- B) `python -m venv venv`
- C) `virtualenv --create venv`
- D) `conda create venv`

---

**Q2.** In the OpenAI chat API, which role should be used for the AI model's response?

- A) `"system"`
- B) `"user"`
- C) `"assistant"`
- D) `"model"`

---

**Q3.** What is the safest way to store your OpenAI API key in a Python project?

- A) Hardcode it in the source file
- B) Store it in a `.env` file and use `python-dotenv` to load it
- C) Store it in a public config.json
- D) Pass it as a command-line argument

---

**Q4.** In a multi-turn chatbot, why do we append each message to a `history` list?

- A) To reduce API costs
- B) Because the LLM is stateless and needs the full conversation each time
- C) To enable streaming responses
- D) To avoid rate limiting

---

**Q5.** What does `json.loads()` do?

- A) Writes a Python object to a JSON file
- B) Parses a JSON string into a Python dictionary/list
- C) Loads a JSON file from disk
- D) Converts a dict to a JSON-formatted string

---

**Q6.** Which library is used to read PDF files in Python?

- A) `pdfplumber` or `pypdf`
- B) `pdf-reader`
- C) `filereader`
- D) `docx`

---

**Q7.** What is the benefit of using `asyncio.gather()` when making multiple LLM API calls?

- A) It makes the calls cheaper
- B) It runs calls sequentially for safety
- C) It runs calls in parallel, reducing total wait time
- D) It automatically retries on failures

---

**Q8.** In a Python f-string, how would you include a variable `name` inside the string?

- A) `"Hello, {name}"`
- B) `"Hello, " + name`
- C) `f"Hello, {name}"`
- D) `"Hello, %s" % name`

---

**Q9.** Which HTTP header is required when calling the OpenAI API directly with `requests`?

- A) `"Content-Type: text/plain"`
- B) `"Authorization: Bearer YOUR_API_KEY"`
- C) `"X-API-Version: v1"`
- D) `"Accept: application/json"`

---

**Q10.** What does `pip freeze > requirements.txt` do?

- A) Installs packages from requirements.txt
- B) Lists all installed packages and saves them to requirements.txt
- C) Removes unused packages
- D) Updates all packages to their latest version

---

**Score:** ___/10

---

## Answer Key

| Question | Correct Answer |
| :---: | :---: |
| **Q1** | B |
| **Q2** | C |
| **Q3** | B |
| **Q4** | B |
| **Q5** | B |
| **Q6** | A |
| **Q7** | C |
| **Q8** | C |
| **Q9** | B |
| **Q10** | B |
