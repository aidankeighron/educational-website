---
name: Educational Content Guidelines
description: Rules and pedagogical guidelines for writing technical tutorials and educational content.
---

# Educational Content Guidelines

When creating or refactoring tutorials and educational materials, strictly adhere to the following pedagogical principles. The goal is to provide a seamless, encouraging, and highly digestible learning experience.

## 1. Guided Instruction over "Challenge Tasks"
- **Avoid Intentional Errors:** Never provide code snippets with intentional bugs, syntax errors, or missing logic meant for the reader to "figure out." Provide clean, working code.
- **Remove Hidden Content:** Do not hide answers behind blur effects or toggles. Present the solution clearly and explain it step-by-step.
- **Step-by-Step Framing:** Replace "Challenge Tasks" with guided walkthroughs. Build the feature incrementally with the reader.

## 2. Modularize Code Examples
- **Break Down Large Blocks:** Never dump massive, multi-hundred-line code blocks all at once. 
- **Partitioning:** Split large files into smaller, logical parts (e.g., *Part 1: Setup and State*, *Part 2: UI Rendering*, *Part 3: Navigation*).
- **In-Line Explanations:** Provide a clear, concise explanation immediately preceding each partitioned code block so the reader understands what that specific chunk accomplishes.

## 3. Interactive Pedagogy (Open-Ended Questions)
- Keep the reader engaged by asking critical thinking questions about the "why" and "how" behind the architecture.
- **Formatting:** Use blockquote styling with a tip callout for these questions:
  ```markdown
  > **QUESTION:** Why do we use approach X instead of approach Y? What happens behind the scenes?
  {: .prompt-tip }
  ```
- Place these questions naturally when introducing new concepts, hooks, or standard practices.

## 4. Enforce Modern Best Practices
- Always teach the most modern, accepted best practices for the framework being used.
- Explicitly point out why a framework-specific feature is better than a legacy approach (e.g., using framework routing components instead of standard HTML anchor tags for Single Page Applications).

## 5. Tone and Encouragement
- Maintain an encouraging, positive, and authoritative tone.
- Acknowledge when a concept is difficult, but assure the reader that breaking it down makes it manageable.
