# Legal assistant (Peruvian labor law) - step-by-step plan (v5)

Personal learning project: RAG, embeddings, evaluation, tool calling, and a demo deployed on AWS.
Work on one stage at a time. Stages 0-4 already give a working, measured RAG. Stages 5-8 make it more capable, expose it and deploy it.

Priority: finish and deploy stages 0-8 first. Everything in Phase 2 comes after, as separate improvements.

## Three levels of the system

The project is built as three levels, each one measured with the same questions, so the README can show what each level adds.

- **Level 1 - Traditional RAG (stages 0-4).** Python controls the search and gives the retrieved norms to the LLM. You learn embeddings, chunking, pgvector, retrieval, prompts and evals.
- **Level 2 - RAG + tool calling (stage 5).** The LLM can request to run the CTS and gratificaciones calculators. You learn tools, schemas, argument validation and function execution.
- **Level 3 - Agentic RAG + harness (Phase 2).** The agent decides when to search the norms, rewrite the query, calculate, ask for data or finish. The harness is the code around the model: the loop, the state, the step limit, timeouts, tool execution and traces. You learn orchestration, state, memory, execution loops, security and agent evaluation.

Not to be confused with "retrieval v1 / v2" in stages 2 and 3, which compare search methods inside Level 1.

## What gets built

A web app where someone asks a question about Peruvian labor law and gets an answer that cites the exact article of the norm. If there is no source, the system says so. Salary calculations (CTS, gratificaciones) are done by tested Python functions, never by the model. Backend: Python API in Docker, deployed on AWS. Frontend: Next.js, UI only. The demo states that it is not professional legal advice.

## Stack

| Layer | Tool | Note |
|---|---|---|
| Language and API | Python 3.12+, uv, FastAPI, Pydantic | Plain code first, frameworks later |
| Database | PostgreSQL + pgvector (Docker Compose), psycopg | Vectors and data in one place |
| LLM | Claude via the Anthropic API | Behind a provider adapter |
| Embeddings | Multilingual model: Voyage, OpenAI or bge-m3 (local) | Decided in stage 2 by comparing |
| Reading norms | PyMuPDF or pdfplumber, BeautifulSoup | PDF and HTML from official sources |
| Search | pgvector (cosine) + tsvector (Spanish) | Hybrid in stage 3 |
| Tests and evals | pytest + own evaluator | Wired into CI in stage 8 |
| UI | Next.js (App Router) + TypeScript | UI only, all logic stays in Python |
| Deployment | Docker, ECR, ECS Fargate, ALB, RDS, S3, Secrets Manager, CloudWatch | Containerized backend |
| Front hosting | AWS Amplify Hosting (or Vercel) | Points to the API URL |
| CI/CD and infra | GitHub Actions; Terraform (optional) | Deploy only if the eval score does not drop beyond the tolerance |
| Traces | Own logging of tokens and cost; Langfuse (optional) | |

## Stage 0 - Norms and test questions (no code)

**Build**
- Choose the norms in scope. Candidates: the CTS law and Law 27735 (gratificaciones). Verify the current text and link on the official source (El Peruano / SPIJ).
- 50 questions, each with the article that answers it: 32 everyday ones ("when is the CTS deposited?", "how much gratificación do I get if I joined in March?"), 6 that cite an article number, 4 that need several articles, and 8 trap questions where the right answer is "no source" (for example "they fired me without notice" if no dismissal norms are loaded yet).
- Split them in two groups: 35 development questions (22 everyday, 4 with article number, 3 multi-article, 6 trap) and 15 held-out questions (10, 2, 1, 2). Held-out questions are never used to tune the prompt or the search. They are run only when closing stage 4 and before deploying.
- Save everything in `evals/questions.yaml`. Per question: expected answer, expected articles, category, group (development or held-out) and, for trap questions, the abstention criterion.
- To start fast, begin with 10 and grow to 50 before comparing versions: with few questions, a difference of one or two is noise.

**Learn:** what an evaluation set is and why it is written before the code; why part of it is kept untouched; how a Peruvian norm is cited.
**Tools:** El Peruano, SPIJ, a YAML file.
**How it is tested:** each expected article is checked by hand against the official text. If you cannot find it, the question is rewritten.
**Done when:** the questions are labeled and split into development and held-out, and there is a list of norms with link and date.

## Stage 1 - Environment, database and loading the norms

**Build**
- Python project with uv and pytest, and PostgreSQL with pgvector running via Docker Compose.
- A script that reads each norm, splits it by "Artículo N." and stores it in an `articles` table: norm, number, text, valid from/to, source URL. It can run twice without duplicating data.
- Decide early what to do with modified norms. The first version may ignore validity dates, noted as a limitation.

**Learn:** Python from scratch (virtual env, functions, types, dataclasses, files, regular expressions), talking to PostgreSQL from code, Docker Compose. Why chunking decides quality: if an article is split in half, the system cites wrongly.
**Tools:** Python, uv, pytest, PyMuPDF or pdfplumber, BeautifulSoup, psycopg, Docker.
**How it is tested:** count articles per norm against the official index, check 10 at random by hand, confirm none is cut or mixed with the next one.
**Done when:** the norms are loaded and the script is repeatable.

## Stage 2 - Embeddings and retrieval v1

**Build**
- One embedding per article stored in a vector column, and cosine-distance search.
- A script that runs the development questions and computes recall@5: does the expected article appear in the top 5?
- The same measurement with 2 different embedding models.

**Learn:** what an embedding is ("they fired me without notice" lands near the dismissal articles even without shared words), cosine distance, vector indexes (HNSW, IVFFlat), cost per token.
**Tools:** pgvector, a multilingual embedding model, pytest.
**How it is tested:** a table with recall@5 per question and overall, saved in the repo.
**Done when:** recall@5 is measured for 2 models and one is chosen based on those numbers.

## Stage 3 - Hybrid search and reranking (v2)

**Build**
- Keyword search with tsvector (Spanish), combined with semantic search (for example rank fusion, RRF).
- A validity filter and, optionally, a reranker model that reorders the best results.
- A report comparing v1 and v2.

**Learn:** when each search wins: keywords win with "article 16" or acronyms like CTS; semantic wins with everyday language. What rank fusion and reranking are.
**Tools:** tsvector, pgvector, optional reranker (Cohere, Voyage or bge-reranker).
**How it is tested:** recall@5 of v1 and v2 over the same development questions, split by type (everyday, with article number, multi-article).
**Done when:** there is a comparison table and the chosen combination is justified with those numbers.

## Stage 4 - Answer with citations, abstention and evaluation

**Build**
- A provider adapter (a class that hides the SDK) and a versioned prompt in the repo. One interface and one adapter (Claude) for now. The interface always returns the same thing: text, list of tool calls, tokens used and cost. The simulated provider used in tests is a second implementation of the same interface, which forces a clean design. A real second provider comes in Phase 2.
- Output validated with Pydantic: answer, list of citations, status (answered or no source).
- An automatic check that every citation matches an article that was really retrieved.
- Logging of tokens, time and cost per call.
- An evaluator that runs all questions and measures 3 things: is it correct, is the citation correct, did it abstain when it should.
- The evaluator saves, per question and per run: prompt version, model, parameters and cost.

**Learn:** the Claude API (messages, tokens, cost), structured output, how to write a prompt that forces citations, and how to tell a retrieval failure (did not fetch the article) from a generation failure (fetched it and the model ignored it). Also: the context window, and what goes in the system prompt vs. the user question vs. the retrieved articles. A retrieved citation does not guarantee it supports the claim. Optional support: Anthropic Academy course "Building with the Claude API".
**Tools:** Anthropic SDK, Pydantic, pytest.
**How it is tested:** a test with a simulated provider response, a real run over all development questions, and tests for retries and timeouts.
**Done when:** a baseline score is saved on the development questions, the trap questions are rejected and, from here on, every change is compared against that score. Run the 15 held-out questions once to check that the score holds outside what you tuned on. If it drops a lot, you were tuning to the questions, not to the problem.

## Stage 5 - Tool calling: calculators

**Build**
- Two Python functions, one for gratificación and one for CTS, using Decimal. The exact rules come from the norm and the official MTPE examples, not from memory.
- The functions exposed to the model as tools. If data is missing, the model asks for it: "how much is my CTS if I earn 3000?" must ask for the start date instead of inventing it.
- 20 test phrases with the expected tool and arguments.
- The backend validates the arguments the model sends (types, ranges, dates) before running the function. Never trust what the model generates.

**Learn:** how tool calling works (define the tool with a schema, run it, return the result), why money uses Decimal (0.1 + 0.2 is not 0.3), when a function is better than the model.
**Tools:** Claude API with tools, Pydantic, pytest.
**How it is tested:** at least 15 tests per calculator (incomplete month, variable salary, unjustified absences, start mid-semester) and the 20 tool phrases. Extra cases: invalid values (negative salary, future date), missing data, tool errors, and a question that needs no calculator (the model must not call it).
**Done when:** the model picks the right tool, asks for missing data, and the stage 4 score does not drop.

## Stage 6 - API with streaming and security

**Build**
- FastAPI with a streaming chat endpoint (Server-Sent Events), one to fetch an article and one health endpoint.
- A usage limit per IP or user and a spending cap in the model provider's console, because every question costs money.
- A Dockerfile for the API and a Docker Compose that starts API and database with one command.
- Basic traces in the logs: what was retrieved, tokens, cost and time. Langfuse is optional.
- Security: the norm text and what the user types are untrusted data, never instructions. Limits on input size, on model calls per question and on spend per session.
- Logs never store personal data (a user may type their salary or employer) or secrets.
- A visible notice: the tool gives guidance based on the loaded norms and is not professional legal advice.

**Learn:** FastAPI, dependency injection, SSE streaming, config and secrets via environment variables, rate limiting, containers. What direct prompt injection is (the user writes "ignore your instructions") and indirect (the instruction comes inside a retrieved document).
**Tools:** FastAPI, httpx, pytest, Docker, Docker Compose.
**How it is tested:** pytest + httpx per endpoint, an integration test against the real database in Docker, and a check that the rate limit triggers. Adversarial cases saved as tests: a malicious instruction inside a test article, an attempt to make the model reveal the key or the prompt, invalid arguments to the calculators, a huge input, and an exhausted session budget.
**Done when:** one command starts everything, a test request returns an answer with citations, and the adversarial cases do not leak secrets or bypass the rules.

## Stage 7 - Next.js interface

**Build**
- A streaming chat, tappable citations that show the article text, loading and error states.
- A home page with example questions and a "How it works" page with your evaluation results (recall@5 and the v1 vs v2 comparison).
- The "not legal advice" notice is visible in the chat, not only in the README.

**Learn:** Next.js App Router in a real project, Server vs Client Components, consuming streaming, CORS. The UI contains no business or AI logic.
**Tools:** Next.js, TypeScript, Playwright (optional).
**How it is tested:** three flows: a question with a cited answer, one that is rejected, and a calculation that asks for missing data.
**Done when:** it works from a phone with a link.

## Stage 8 - Deployment on AWS

Put the dockerized backend on the internet, with the database and norms on AWS, and make every change deploy only if tests and evaluation do not get worse. AWS AI services are not used; the model is still called through its API.

**Build**
- The API Docker image, pushed to ECR.
- The API running on ECS Fargate, behind an Application Load Balancer.
- PostgreSQL with pgvector on RDS (the `vector` extension), in a private subnet.
- The original norms in S3.
- The model key in Secrets Manager and logs in CloudWatch.
- The front on AWS Amplify Hosting (or Vercel), pointing to the API URL.
- A GitHub Actions flow: on every change run tests and evaluation and, if the score does not drop beyond the tolerance, build the image and deploy.
- Two test levels: deterministic tests (pytest, no model calls) always run; the evaluation that calls the API runs with a threshold and a tolerance. A drop of 1 question out of 35 does not block; the doubtful case is repeated before deciding. Security cases tolerate zero failures.
- A budget alert (AWS Budgets) from day one. Terraform is optional, to create and destroy everything with commands.

**Learn:** containers in the cloud, basic networking and permissions (IAM, security groups), secrets management, CI/CD with evaluations, infrastructure as code.
**How it is tested:** the public URL answers with a citation. Push a change that lowers the score and check that the deployment stops. If using Terraform, destroy and recreate everything.
**Costs:** ALB, Fargate and RDS cost money even with no traffic. Check prices on the official AWS page and destroy the infrastructure when not demoing. A cheaper variant is a single EC2 machine with the same local Docker Compose, though you manage the server. AWS Budgets only alerts, it does not cut spending: the real limits are the spending cap in the model provider's console, the rate limit in the API, and destroying what you do not use. Every evaluation run in CI calls the model and costs money, so run it on changes that touch the prompt, the search or the tools, not on every commit.
**Done when:** the public URL works and you can explain the path of one question end to end: what came in, what was retrieved, what the model answered, what it cost.

## Phase 2 (optional, once the above works)

Each item is an independent improvement: one commit and one decision note more to talk about in interviews. You do not need all of them.

- **Agent on top of the same RAG.** An agent that chooses between searching the norms, rewriting the query and searching again, using the calculators, asking for missing data, or abstaining. First a hand-written tool loop with a step limit and a timeout; then LangGraph. Compare both against the fixed flow from stage 4 with the same questions (correctness, citations, steps, latency, cost) and note when the framework is worth it.
- **Second model provider.** A second adapter (for example OpenAI) and the same evaluation with both, with fixed versions and configuration. Compare quality, abstention, latency and cost, and document the choice.
- **Finer evaluation.** Repeat each case 3 times in the final comparisons and look at the mean and the worst result. Also measure whether each claim is supported by the cited article. If a model is used as a judge, check it by hand on a sample.
- **MCP server** (read-only) with norm search and the two calculators.
- **Bedrock:** call Claude through AWS, changing only the provider adapter.
- **Second channel** on the same API: WhatsApp (Meta Cloud API via a webhook in the backend) or a React Native client.
- **Payslip photos** with a vision model to feed the CTS calculator, with explicit validation of the extracted data and care with personal data.

## Out of the plan on purpose

LangChain (the main stages use plain Python; LangGraph appears only in Level 3 as a comparison against the hand-written loop), fine-tuning, training models, GPUs, Transformer and quantization theory, local inference, and multi-agent systems. They add nothing for this goal beyond finishing and deploying the stages above well. If they interest you, they are separate reading.

## What to show when finished

README with the problem, architecture and how to run it; the evaluation table (retrieval v1 vs v2) and the held-out result; what each level (1, 2 and, if done, 3) adds on the same questions; the security cases that pass; a short demo with question, sources, calculation and trace; the decision notes; and the public URL.

## Change log

- v5 (2026-11-21): three levels (traditional RAG, RAG + tool calling, agentic RAG + harness) named at the top; stage 4 fixes what the provider interface returns; LangChain declared out of scope; stage 0 grows from 30 to 50 questions with a held-out group; security and the "not legal advice" notice added to stages 6 and 7; argument validation in stage 5; CI tolerance and cost notes in stage 8; Phase 2 reorganized (agent first, second provider and finer evals as options). Stages 1-3 unchanged apart from using the development questions.
- v3: original plan.
