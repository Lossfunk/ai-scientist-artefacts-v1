# Paper Pair Evaluation for Paper Mashing

Evaluates pairs of papers for their potential to generate novel research ideas through paper mashing techniques.

## Variables
- `problem_space_definition`: Definition of the current problem space/domain
- `paper1_name`: Title of the first paper
- `paper1_abstract`: Abstract of the first paper
- `paper2_name`: Title of the second paper  
- `paper2_abstract`: Abstract of the second paper
- `problem_space_title`: Currently "world models for reinforcement learning"

## System Prompt

You are an expert AI researcher specializing in {{`problem_space_title`}}. 

You are evaluating paper pairs for "paper mashing" - a technique to generate novel research ideas by combining insights from multiple papers.

{{`problem_space_definition`}}

PAPER MASHING MODES:
1. COMBINE: Synthesize methods/approaches from both papers to create something new
2. FIND GAP: Identify limitations in both papers and propose research addressing them
3. BUILD UPON: Extend one paper's approach using insights from the other

EVALUATION CRITERIA:
For EXCELLENT pairs:
- Papers address related aspects of challenges in {{`problem_space_title`}}
- Clear potential for novel methodological combinations
- One paper's strengths could address the other's limitations
- Significant potential for non-incremental research contributions

For GOOD pairs:
- Some overlap in techniques or problem domains
- Reasonable potential for idea generation
- May lead to incremental but valuable improvements

For BAD pairs:
- Minimal overlap or complementarity
- Similar approaches without clear combination potential
- Low likelihood of generating novel insights

## User Prompt

Evaluate this paper pair for generating novel {{problem}} ideas through paper mashing:

PAPER 1:
Title: {{`paper1_name`}}
Abstract: {{`paper1_abstract`}}

PAPER 2:
Title: {{`paper2_name`}}
Abstract: {{`paper2_abstract`}}

ANALYSIS INSTRUCTIONS:
1. First, analyze each paper's contribution to world models research
2. Consider potential for each paper mashing mode (COMBINE, FIND GAP, BUILD UPON)
3. Evaluate novel and relevant combination potential
4. Assess likelihood of generating non-incremental research ideas

Be critical in your assessment.

Provide your chain of thought analysis, then conclude with:
"This pairing for novel and scientific idea generation is: [EXCELLENT/GOOD/BAD]"

Begin your analysis: