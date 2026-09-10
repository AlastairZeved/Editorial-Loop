# Test Paragraph

## Editorial Context
- **Target reader:** A non-technical professional (e.g., marketing manager, project manager) who has never written code but is curious about AI tools and wants to understand how they might change their workflow.
- **Purpose:** Help the reader understand why "vibe coding" (using AI to generate code by describing what you want in plain English) is both more accessible and more dangerous than it appears.

## Preceding Context (what the reader just finished reading)
The previous section described how the reader's first experience with an AI coding tool felt magical — they typed a plain-English description of what they wanted, and working code appeared. It ended with: "That first moment of seeing your words turn into something functional is genuinely thrilling. But the thrill masks something important."

## Text to Edit

The real issue with vibe coding isn't the code quality — it's the impedance mismatch between your mental model and the system's execution model. When you describe what you want in natural language, the LLM tokenizes your intent into a latent representation that maps onto code patterns from its training distribution. This means your "simple request" gets filtered through layers of statistical inference before it becomes executable logic.

Obviously, this is why prompt engineering matters so much. The gap between what you meant and what the model produced is where bugs live — not in the syntax, but in the semantic distance between your specification and the model's interpretation. Think of it like writing a legal contract: every ambiguity in your language becomes a potential failure mode.

The experienced developer knows to inspect the generated code for these semantic gaps. But if you're coming to this fresh, you're essentially signing a contract you can't read. And unlike a bad legal contract, a bad piece of code won't just sit there — it'll execute. Every time. Exactly as written, not as intended.
