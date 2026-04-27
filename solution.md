# GyanBot — AI Product Design Document

**Project:** CampusConnect / GyanBot
**Role:** AI Product Designer (1-day engagement)
**Author:** [Your Name]

---

## Part 1 — GyanBot's System Prompt

> **Identity:** You are *GyanBot*, an AI study buddy built by CampusConnect for Indian engineering college students (B.Tech / B.E., 1st to 4th year). You help with three things only: academic doubt-solving, career & internship guidance, and discovery of hackathons, workshops, and college fests happening in the next 30 days.
>
> **Tone:** Warm, calm, and encouraging — like a helpful senior in the hostel. Use simple English, short sentences, and zero jargon. Students are usually stressed before exams or placements; never make them feel dumb for asking.
>
> **Boundaries:**
> 1. Never write complete assignments, lab records, or full code submissions that the student will pass off as their own — explain the concept and give partial code only.
> 2. Never give medical, mental-health, or legal advice. If a student sounds anxious or burnt out, gently suggest they talk to their college counsellor or iCall (9152987821).
>
> **Format:** Start every answer with a one-line *TL;DR*, then give the detailed explanation in bullet points. End with a "Next Step" line.
>
> **Scope:** In-scope — engineering subjects (CSE, ECE, Mech, Civil, EEE, etc.), GATE/placements prep, internships (Internshala, Unstop, LinkedIn), Indian college events (Techfest, Mood Indigo, Smart India Hackathon, IIT/NIT/IIIT fests). Out-of-scope — politics, dating, fee disputes, anything outside academics/career/events.

**Why this boundary matters (the homework-writing rule):**
If GyanBot solves complete assignments, it becomes a cheating tool — colleges will block it and parents will distrust the product. By teaching concepts instead of submitting answers, GyanBot stays useful long-term and protects students from academic-integrity violations under AICTE/university rules.

---

## Part 2 — User Prompt for Aryan (C-I-F-C)

> **[Context]** I'm Aryan, a 3rd-year B.Tech CSE student at a Tier-2 college in Pune. I know Python well, have done two basic ML projects (Titanic dataset, house-price regression on Kaggle), and I'm a decent communicator — I anchor college events. CGPA is 7.8.
>
> **[Instruction]** I want to land a Data Science / Data Analyst summer internship by December 2026. Tell me exactly which skills I'm missing for this role, and which Indian companies and platforms I should target.
>
> **[Format]** Reply in three sections: (1) Skill Gaps — what's missing and why it matters, (2) 8-Week Learning Plan — free or low-cost resources (NPTEL, Coursera financial aid, YouTube), (3) Target Companies — 5 names with where to apply (Internshala, Unstop, LinkedIn, careers page).
>
> **[Constraints]** Budget under ₹2,000 total. Only Indian or India-hiring companies. No GRE/GATE-style suggestions. Keep the plan doable in 8–10 hours per week alongside college.

**Note on labels:**
*Context* = Aryan's background and current level. *Instruction* = the exact task. *Format* = the structure of the reply. *Constraints* = budget, geography, time, and exclusions that keep the answer realistic.

---

## Part 3 — Framework Choice: **LangChain**

**Recommendation:** LangChain.

**Justification:**
GyanBot's defining requirement is grounding answers in **CampusConnect's own data** — college-specific FAQs and syllabus PDFs — not just the LLM's general knowledge. LangChain is purpose-built for exactly this: it ships with document loaders (PDFLoader, CSVLoader), text splitters, vector-store integrations (FAISS, Chroma, Pinecone), and a Retriever → LLM chain that turns "search the syllabus + answer" into a few lines of config. Since GyanBot is a **single assistant** giving **one fast, clean response**, LangChain's straightforward chain pattern (`Retriever → Prompt → LLM → Output`) fits perfectly.

**Why CrewAI is less suitable:** CrewAI is designed for multi-agent *teams* with role-played agents (Researcher, Writer, Reviewer) collaborating on a task. GyanBot has no such division of labour — spinning up multiple agents for a doubt-solving query adds latency and cost with no benefit.

**Why AutoGen is less suitable:** AutoGen specialises in multi-agent *conversations and debate* between agents (e.g., a Coder agent and Critic agent iterating). GyanBot must reply in one shot to a stressed student before a viva — back-and-forth agent debate would slow the response and break the "fast, single clean reply" requirement.

---

## Part 4 — Agent Flow: Career Guidance Feature

**Building blocks used:** Input (Prompt) · Brain (LLM) · Hands (Tools) · Output (Result)

**Step 1 — Input (Prompt):**
Aryan types his C-I-F-C prompt into the GyanBot chat: *"I'm a 3rd-year CSE student… want a Data Science internship by December… reply in 3 sections…"*

**Step 2 — Brain (System Prompt loads):**
The GyanBot system prompt from Part 1 is prepended. The LLM now knows it is GyanBot, the tone, boundaries, and that this query falls under the *career guidance* scope.

**Step 3 — Brain (Intent + Query Planning):**
The LLM parses the message, identifies intent = *career guidance* and entities = {branch: CSE, year: 3, target role: Data Science Intern, current skills: Python + basic ML, deadline: Dec 2026}. It decides a tool call is needed because skill-gap and company data live in CampusConnect's database, not in the LLM's head.

**Step 4 — Hands (Tool Call → Career Resource Database):**
GyanBot calls the `career_db_retriever` tool (a LangChain RAG retriever over a vector store of: role-skill maps, company hiring pages, NPTEL/Coursera course catalogue, past internship postings from Internshala & Unstop). Query sent: *"Data Science Intern — required skills, India-hiring companies, ₹2000 budget resources."* The retriever returns the top 5–7 matching documents.

**Step 5 — Brain (Synthesis):**
The LLM merges Aryan's profile with the retrieved docs, computes the gap (missing: SQL, Pandas-deep, Statistics, Power BI, one end-to-end project), maps each gap to an affordable resource, and shortlists companies hiring DS interns in India.

**Step 6 — Output (Result):**
Aryan sees a clean Markdown reply on his screen, opening with a TL;DR line, then three sections exactly as he asked:
1. **Skill Gaps** — bulleted list (SQL, Statistics, Pandas, Power BI, GitHub portfolio) with one-line "why it matters."
2. **8-Week Learning Plan** — week-by-week table with free resources (NPTEL "Data Analytics with Python," Krish Naik's YouTube playlist, Kaggle Learn, one ₹449 Udemy SQL course).
3. **Target Companies** — 5 names (Razorpay, Swiggy, ZS Associates, Fractal Analytics, Mu Sigma) with the exact platform to apply on (Unstop, Internshala, LinkedIn, careers page).
The reply ends with a **Next Step:** *"Start with the SQL course this week — ping me after Day 7 for a checkpoint."*

---

*End of design document.*
