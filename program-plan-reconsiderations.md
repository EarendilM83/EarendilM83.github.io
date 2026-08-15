# Program Plan v6 — what we must reconsider (vs the PRD)
Reviewed against: design-requirements.html v2.5 (Part R), master PRD ch. 4/5, journey map.

## The plan in one paragraph
10X Front-End: 10 months · 500 students in 20 groups × ≤25 (morning/evening streams, same lecture twice daily) ·
11 staff (3 lecturers + 8 mentors) · 2 new topics/week, lecture Mon/Wed + mentoring Tue/Thu, Friday =
booster/masterclass/reserve · 9 pre-planned booster sessions (content set 5 days ahead from signals) ·
7 blocks / 78 sessions, every session with written syllabus (content, mentoring focus, assignment, typical mistakes) ·
skill map = verifiable actions, not topics · production rule: topic author (lecturer) ships slides + 10-question quiz +
auto-checked assignment + mentoring sheet 48h before delivery · narrative frame = "Code Order" (one continuous
10-month story, NO licensed IP — content distributes to 500 students + LMS + other colleges) ·
remaining before start: production (78 quizzes, 78 auto-assignments, 78 mentoring sheets, slides), the LMS
code-auto-check line OR GitHub-Classroom fallback, group registration with stream choice, start date.

## What we must reconsider

### R1 — "Quest" means two different things (naming collision, real)
Plan: every session has ONE named story-quest due before next session (the assignment itself).
Our PRD: quests = daily/weekly gamified mini-tasks (RE chapter, quests tab content).
→ RECONSIDER: rename one. Recommendation: plan's session-assignments = **"missions"** (they're the curriculum);
our daily/weekly loop items stay **"quests"**. Sweep RM-04/RM-09 assignment copy accordingly.

### R2 — Streams (morning/evening) don't exist in our model
Plan: 20 groups split into 2 streams; lecture held twice daily; stream disbalance ladder; registration form
with stream choice is a pre-start deliverable.
→ RECONSIDER: add `stream` to cohort model (spec 01) + registration/admin flow for stream choice +
capacity balancing. Our PRD has cohorts, no streams. New flow needed (admin-side).

### R3 — Mentoring sessions + boosters are first-class schedule events
Plan: each group gets 2 mentoring sessions/week after lectures; Friday = booster/masterclass/reserve;
booster content triggered by signals 5 days ahead.
→ RECONSIDER: RM-01 calendar/session kinds must include mentoring (small-group, mentor-attached) and
booster/masterclass kinds. Wire-in opportunity: our mistake-queue + quiz-fail signals ARE the booster trigger
data — the app should surface "Friday booster: CSS Grid — 6 of your group struggled" (dashboard + calendar).

### R4 — Code auto-grading line = THEIR named critical path
Plan: "quiz engine, attendance, submissions work. Code auto-check (GitHub link → editor → commit verification)
still in design — needed by start, or GitHub Classroom fallback."
→ This VALIDATES master-PRD batch B8 (server-side grading + Classroom wrap) and makes it date-critical.
Reconsider batch order if the start date is near: B8's minimal line (repo link + autograde → gradebook) may
need to land before the full console.

### R5 — "Code Order" narrative frame vs Kata
Plan: one continuous 10-month story world, deliberately no licensed franchises (distribution to other colleges).
→ RECONSIDER: Kata must live INSIDE the Code Order universe (e.g., the Order's guild-cat mentor), not as a
parallel mascot world. Part M skins + copy framing need a Code-Order pass. (Also their no-licensed-IP rule is
exactly right for multi-tenant — keep it as a content law.)

### R6 — Their content atom ≠ our content atom
Plan atom: session (lecture + mentoring + assignment + quiz). Our atom: floor (practice unit) + LMS session.
→ The teacher-content bridge (audit gap #17, master-PRD batch B4) must map: 78 sessions → syllabus →
slides/quiz/assignment/mentoring-sheet → floor units on the roadmap. B4 is confirmed as THE critical bridge;
the plan's own "LMS critical line" section is its acceptance letter.

### R7 — Production SLA exists (48h before delivery) → builder needs a due-date pipeline
→ R-K builder + RQ-06 publishing center should carry per-session production deadlines ("materials due 48h before
session") — a back-office due-date lane we haven't spec'd.

### R8 — Group (25 + mentor) is the social unit students actually live in
Our social design centers bands (≈30 league) and cohorts (12). The plan's daily reality = the 25-person
mentored group.
→ RECONSIDER: cohort feed / class quest / attendance surfaces should key on the GROUP first; buddy/mentor
assignment should respect group boundaries.

## What the plan gets RIGHT that we should honor
- Booster sessions pre-planned in the calendar (catch-up built in, not emergency) — matches our
  "wounded ≠ zeroed" philosophy; make boosters visible in-app.
- Skill map = verifiable actions — matches our mastery model.
- 48h authoring SLA with fixed content roles — matches our reviewer≠author QA gate.
- No licensed IP — correct for multi-tenant distribution.
