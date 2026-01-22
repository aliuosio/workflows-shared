Engineering a Deterministic Matchmaker in a Probabilistic World

I have this particular dread when I think about job matching algorithms. They're usually either complete black boxes that say "trust me, I know what I'm doing" or fragile setups of text patterns that break as soon as someone writes "Python Wizard" instead of "Software Engineer (Python)."

Today I'm walking through a workflow that aims for something reasonable in the middle. It's an automated system that runs every morning at 11 AM to score job listings against candidate profiles. The tech stack includes n8n, PostgreSQL, Qdrant, and three different Mistral models.

Here's how we put together a system that doesn't just estimate relevance but actually calculates it with the same careful precision a senior developer uses when reviewing code changes.

The Setup: A Multi-Step Assembly Line

The workflow follows a standard ETL approach, but with extra help from retrieval-augmented generation and agent-based thinking. Rather than one huge prompt trying to handle everything, we split the work across specialized steps.

1. The Starting Point and First Filter

At 11:00 AM, the workflow checks PostgreSQL for jobs that haven't been scored yet—where match_score_percentage equals zero.

We add some basic protection right away: the null check. If a job listing has only a title and no description, we skip the expensive AI processing. We give it a score of 1 and move on.

2. Creating Vectors and Finding Matches

For jobs that have actual content, we use Google Gemini to create vector embeddings from the descriptions. These get sent to Qdrant.

The goal goes beyond simple keyword searches. We want to find deep similarities between what the job really needs and what candidates have done in their careers. Qdrant gives us the most relevant candidate profiles, which then feed into the AI processing pipeline.

3. The Three-Model Approach

We use three different Mistral models through Langchain for the main work. Why three? Because using the largest model just to format data is wasteful and expensive.

The Scout (Mistral): Looks at job requirements and the candidate data we found. It figures out what the job actually needs and whether we have people who match.

The Analyst (Mistral): This handles the core logic. It uses a fixed, weighted formula to turn qualitative assessments into numbers.

The Matchmaker (Mistral): The final step. It takes the analysis and pushes it through a structured output parser. This prevents crashes when an AI model adds extra conversational text before the actual data.

The Scoring: Making "Good Fit" Measurable

We've all seen "80% match" on LinkedIn and wondered what that even means. To avoid that confusion, this system uses a fixed weighting approach. It's a deliberate choice applied to an otherwise unpredictable model:

Category | Weight | Reasoning
Skills | 45% | Can you actually do the technical work? This is the baseline requirement.
Role | 25% | Are you a frontend developer applying for a backend role? Not quite right.
Seniority | 20% | Senior engineers shouldn't match junior positions and vice versa.
Tooling | 10% | Do you know the specific tools? Helpful but not essential.

The calculation: By requiring the Analyst to use this exact math, we get scores that make sense. If someone gets 70, we can see exactly where they lost those 30 points.

Challenges and Practical Issues

No approach is perfect, and this one has some quirks that need attention:

The empty database issue: If Qdrant has no candidate data, the models return no results found. The workflow handles this, but it shows that AI is only as good as the information you give it.

Cost considerations: Running three Mistral models plus Gemini embeddings for each job posting adds up. We trade speed and API costs for better accuracy and reliable results.

Scale limits: Processing jobs one by one works for small batches, but if someone adds 5,000 jobs right before the 11 AM run, it will hit API limits quickly.
