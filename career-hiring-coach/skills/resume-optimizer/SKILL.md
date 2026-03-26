# Skill: Resume Optimizer

ATS keyword strategy, formatting rules, section structure, and weak phrase detection for the Career & Hiring Coach agent.

---

## ATS (Applicant Tracking System) Fundamentals

### How ATS Works
1. Resume is parsed into structured fields (name, contact, experience, education, skills)
2. Keywords are extracted and matched against job description requirements
3. Score is generated — below threshold = auto-reject before human review
4. Formatting issues (tables, columns, text boxes) cause parsing failures

### What Breaks ATS
- ❌ Multi-column layouts
- ❌ Tables for content (headers or skills sections)
- ❌ Text boxes and graphics
- ❌ Headers/footers with key information
- ❌ PDFs with non-embedded fonts
- ❌ Non-standard section titles ("Where I've Been" instead of "Experience")
- ❌ Logos, profile photos
- ❌ Infographic-style skill bars

### What ATS Loves
- ✅ Single-column layout
- ✅ Standard section headers (Summary, Experience, Education, Skills)
- ✅ Keywords from the job description, verbatim
- ✅ Dates in consistent format (MM/YYYY or Month YYYY)
- ✅ Clean, parseable file (Word .docx or simple PDF)

---

## Standard Resume Structure

```
[Full Name]
[City, Country] | [Email] | [Phone] | [LinkedIn URL] | [Portfolio/GitHub if relevant]

## Summary (3–4 lines)
[Who you are] + [What you're good at] + [What you're looking for]
Tailor to each role. Include 2–3 keywords from the job description.

## Experience
[Company Name] — [Job Title]
[Month YYYY – Month YYYY] | [Location or Remote]

• [Achievement bullet 1]
• [Achievement bullet 2]
• [Achievement bullet 3]

(Repeat for each role, reverse chronological)

## Skills
[Skill Category]: Skill 1, Skill 2, Skill 3
(Pull directly from job description + your real proficiencies)

## Education
[Degree], [Field of Study]
[University Name], [Year]

## Certifications (optional)
[Certification Name] — [Issuer] — [Year]

## Projects (optional, especially for tech/early career)
[Project Name]: [1-line description + tech stack + impact]
```

---

## Achievement Bullet Formula

**Template:**
```
[Action verb] [what you did] [how/tool/method] [result with metric]
```

**Weak → Strong Examples:**

| Weak | Strong |
|---|---|
| Responsible for marketing campaigns | Launched 12 multi-channel campaigns targeting SMBs, driving 34% increase in qualified leads over 6 months |
| Helped grow the team | Hired and onboarded 8 engineers in 4 months, reducing time-to-hire from 72 to 38 days |
| Managed social media | Grew LinkedIn following from 2k to 18k in 12 months through weekly long-form content strategy |
| Improved processes | Redesigned onboarding workflow using Notion, cutting new hire ramp time by 3 weeks |
| Worked with clients | Managed 15 enterprise accounts ($2M+ ARR), achieving 97% retention over 2 years |

### Strong Action Verbs by Function

**Leadership:** Led, Directed, Managed, Scaled, Spearheaded, Drove, Championed
**Building:** Built, Developed, Launched, Created, Architected, Designed, Implemented
**Improving:** Optimized, Reduced, Streamlined, Automated, Accelerated, Increased, Improved
**Analysis:** Analyzed, Identified, Assessed, Evaluated, Researched, Modeled
**Communication:** Presented, Negotiated, Collaborated, Advised, Trained, Mentored

### Weak Phrases to Flag and Replace

| Flag | Problem | Fix |
|---|---|---|
| "Responsible for" | Passive, task-focused | Use active verb |
| "Helped with" | Vague contribution | Clarify your specific role |
| "Worked on" | No ownership implied | Lead with your action |
| "Assisted" | Diminishes impact | State your actual contribution |
| "Various tasks" | Vague | List specific tasks |
| "Strong communication skills" | Claimed, not proven | Show it through achievements |
| "Team player" | Cliché | Describe a collaborative achievement |
| "Fast learner" | Cliché | Show a specific learning outcome |

---

## Keyword Optimization Process

### Step 1: Extract Keywords from Job Description
Scan job description for:
- **Hard skills** (tools, languages, platforms, methodologies)
- **Soft skills** explicitly mentioned
- **Industry jargon** and buzzwords
- **Required credentials** (degrees, certifications, years of experience)
- **Action verbs** used in responsibilities

### Step 2: Map to Resume
For each keyword:
- ✅ Already present → check location (summary, skills, bullets)
- ⚠️ Present but buried → surface to top of relevant section
- ❌ Missing but you have the skill → add naturally to skills or bullet

### Step 3: Keyword Density
- Primary keywords (core role requirements): appear 2–4x across resume
- Secondary keywords: appear 1–2x
- Avoid keyword stuffing — must read naturally for human reviewers

### Example: Software Engineer (Backend) JD Keywords
```
Job requires: Python, Django, REST APIs, PostgreSQL, AWS, Docker, CI/CD, Agile, "scalable systems"

Resume should include (verbatim where possible):
- Skills section: Python, Django, REST APIs, PostgreSQL, AWS (EC2/RDS), Docker, CI/CD (GitHub Actions)
- Summary: "...building scalable backend systems..."
- Bullets: "Designed REST APIs using Django..." / "Deployed containerized services on AWS using Docker and GitHub Actions CI/CD pipeline"
```

---

## Resume Length Guidelines

| Experience Level | Length |
|---|---|
| 0–3 years | 1 page maximum |
| 3–10 years | 1–2 pages |
| 10+ years | 2 pages (senior IC or manager) |
| Executive / Board | 2–3 pages acceptable |
| Academic / Research | CV format, length varies |

> ⚠️ Do not pad with irrelevant experience to hit 2 pages. Trim ruthlessly.

---

## Section-Specific Tips

### Summary Section
- 3–4 lines max
- First-person, no pronoun ("Experienced engineer" not "I am an experienced engineer")
- Contains: title/identity + top 2–3 strengths + value proposition
- Tailored per application — generic summaries hurt more than help

### Skills Section
- Group by category (Technical Skills, Languages, Tools, etc.)
- List skills as single words or short phrases — not sentences
- Only list skills you can speak to in an interview
- Mirror exact tool names from job description (e.g., "Google Analytics 4" not "web analytics tools")

### Education Section
- Place after Experience for anyone with 3+ years of work experience
- Include: Degree, Field, School, Year (no need for GPA unless <3 years out and >3.5)
- Relevant coursework only if it directly applies and you're early career

### Gaps in Employment
- Small gaps (<3 months): no need to explain
- Longer gaps: brief parenthetical in the experience section ("Career break for family caregiving / health / travel")
- Never lie. ATS doesn't care, but human reviewers will notice inconsistencies.

---

## Cover Letter Structure

```
[Date]
[Hiring Manager Name or "Hiring Team"]
[Company Name]

Opening (1 paragraph — hook):
State the role, a specific reason you're interested in THIS company, and your headline value.

Body (1–2 paragraphs — proof):
2–3 specific examples that map your experience to their stated needs.
Reference the job description. Be concrete.

Closing (1 paragraph — ask):
Express enthusiasm, invite next steps, thank for their time.
"I'd welcome the chance to discuss how I can contribute to [specific goal from JD]."

[Signature]
```

**Tone options:**
- **Formal:** Conservative industries (law, finance, government)
- **Conversational:** Startups, creative, tech
- **Hook-first:** Opens with a bold statement, story, or surprising fact about your fit
