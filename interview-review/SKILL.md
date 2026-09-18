---
name: interview-review
description: Analyze interview transcripts to extract questions, evaluate answer correctness, uncover the deeper intent behind each question, and provide corrected answers for mistakes. Use when the user provides an interview transcript (text) and requests structured review, performance analysis, error correction, or deep-dive question interpretation. Supports technical interviews (coding, system design, architecture) and behavioral interviews (STAR, project deep-dive). Input is usually a conversation transcript with interviewer and candidate roles; role labels may be absent and must be inferred.
---

# Interview Review

Read `references/deep-dive.md` for the question-intent analysis framework.

## Workflow

### Step 1: Parse the transcript into Q&A pairs

Given a raw interview transcript:

1. Identify roles: determine who is the **interviewer** and who is the **candidate** (the user). If role labels are absent, infer them from context — questions come from the interviewer, answers from the candidate.
2. Extract each question-answer pair. A question is any prompt by the interviewer that expects a substantive response (including follow-ups). Group multi-part questions and their follow-ups as one question unit when they explore the same topic.
3. For each pair, capture the **question**, the **candidate's answer**, and note whether the answer was **complete**, **partial**, **incorrect**, or **missing** (the user didn't know / didn't answer).

### Step 2: Judge answer correctness

Evaluate each answer against:

- **Technical accuracy**: Is the factual content correct? Does the code/solution work? Does the reasoning follow sound engineering principles?
- **Completeness**: Did the candidate address all parts of the question? Did they cover edge cases, trade-offs, and alternatives where appropriate?
- **Communication**: Was the answer well-structured and clear? For behavioral questions, did it follow STAR (Situation, Task, Action, Result)?

### Step 3: Analyze question intent

For each question, identify the interviewer's underlying objective:

- **Knowledge depth**: Testing whether the candidate understands not just "what" but "why" and "how it works under the hood"
- **Problem-solving approach**: Testing how the candidate thinks through unfamiliar problems
- **System design trade-offs**: Testing awareness of trade-offs, scalability, constraints
- **Experience validation**: Verifying the candidate's claimed experience level
- **Cultural fit / soft skills**: Communication style, collaboration, handling ambiguity
- **Edge-case awareness**: Does the candidate think about failure modes and boundary conditions?

### Step 4: Correct errors

For questions where the answer was **incorrect**, **partial**, or **missing**, provide:

1. The **correct answer** with explanation
2. Why the candidate's answer was wrong or insufficient
3. A **model response** the candidate could have given (one that demonstrates both correctness and strong communication)

## Output format

Always produce a structured Markdown document with these sections in order:

### 1. 问题列表 (Question List)

A numbered list of all questions extracted from the interview, in order.

```
1. 问题一的内容
2. 问题二的内容
...
```

### 2. 问题深意 (Question Deep Dive)

For each question (or a representative selection of the most important ones), explain what the interviewer was really testing.

For less important questions (e.g., simple clarifications), you may skip deep-dive analysis. Prioritize: technical depth questions > system design > project deep-dives > behavioral questions > logistical questions.

Format:

#### 问题 1: [问题简述]

- **考察目的**: What the interviewer wanted to evaluate
- **深层意图**: The unspoken concern or signal behind the question
- **理想回答的方向**: What a strong candidate would cover

### 3. 面试结果分析 (Performance Summary)

- **总体评价**: Overall assessment (1-2 sentences)
- **正确率**: Percentage of correct / acceptable answers
- **分项分析**:
  - 技术深度: Score + brief note
  - 问题解决能力: Score + brief note
  - 系统设计能力: Score + brief note (if applicable)
  - 沟通表达: Score + brief note
- **优劣势总结**: Key strengths and areas to improve

### 4. 错误答案纠正 (Error Correction)

For each question answered incorrectly or inadequately:

#### 问题 [N]: [问题内容]

- **你的回答**: The candidate's original answer
- **问题所在**: What was wrong or insufficient
- **正确答案**: The correct / model answer
- **建议**: Tips for handling similar questions in the future

## Output rules (CRITICAL)

1. Output the full review directly in the chat as a Markdown document — do NOT write it to a file. The structured document below IS the final response.
