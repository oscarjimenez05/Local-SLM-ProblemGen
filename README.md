# Personalized Coding Problem Generation Using Open-source Small Language Models (SIGCSE 2026)

**Official Source Code for the SIGCSE 2026 Undergraduate Student Research Competition Submission.**

> To ensure the reproducibility of our results (specifically the comparison between CHASE, Default, and GPT-4 pipelines), the codebase has been maintained as it was during the data collection phase.

## Abstract
LLMs and their use cases within computer science education have been the subject of much discussion. However, the reliance on cloud-based services when using proprietary models like GPT-4 has barriers, such as cost and data privacy compliance. This work shows an end-to-end local microservice system to generate programming problems with open-source small language models that can run on consumer devices.

In addition to a baseline that uses a single model directly, we evaluate two generation pipelines for generating problems. One is a ChatGPT-4 benchmark, and the second is a multi-agent refinement loop inspired by the CHASE paradigm. We use five models in a feedback loop that work towards increasing the depth of a problem until a target difficulty is reached or exceeded.

We generated 150 problems total across the three methods, which were blindly scored by a computer science educator for metrics such as clarity, difficulty, and overall quality. The results show that CHASE did have better topic adherence, but was 18x slower than the default generation. Chaining small models might not fix the deficiencies of a single model, but rather that the deficiencies are added together. However, the single-model end-to-end method from the open-source models was reasonably fast and outperformed GPT-4 on clarity metrics. This work successfully shows the feasibility of using local models for creating meaningful coding problems, but chained-pipeline approaches may need to either have a higher degree of system robustness for storing user preferences and problem settings or simply use larger models.

---

**Key Result:** While the complex CHASE loop increased topic adherence, the simpler "Default" local model was **18x faster** and achieved a higher Clarity score (4.8/5.0), outperforming GPT-4.

## System Architecture
The system is a full-stack microservice architecture designed for offline deployment:
* **Frontend:** React-based student interface for solving problems.
* **Backend:** Java/Spring Boot application managing the "Generator," "Hider," and "Verifier" agents.
* **Inference Engine:** Local Ollama instance managing 4-bit quantized models.
* **Database:** H2 (Vector & Relational) for tracking student mastery and problem history.

![System Class Diagram](./backend/src/main/resources/Diagrams/TECMap-AI%20UML%20Class%20Diagram.png)

## Quick Start (Demo)

### Prerequisites
* Docker & Docker Compose
* (Optional) NVIDIA GPU with Container Toolkit for faster inference

### Running the Full Platform (Student View)
To launch the full Student Experience (Frontend + Backend + LLM):

```shell
# Builds and starts all services including the React frontend
docker compose --profile frontend up -d
````
> Run inside `/backend` directory

# Service types
There's two main service types: Ollama (open-source) and OpenAI (proprietary).

## Ollama
It uses the models located at ```backend/src/main/resources/Modelfiles```. The files contain the model it's pulling, system prompt, and parameters.  
The main models for (the ones the API endpoint uses) are those with the prefix _cs-_ (e.g., _cs-problemGenerator_). 
Those that do not have this prefix (e.g., _Extractor_) are used for CHASE-like problem generation, and are solely for research purposes (not called by API).

## OpenAI
It uses the same modelfiles at ```backend/src/main/resources/Modelfiles```, but since OpenAI does not admit the same format as Ollama they are being parsed inside ```backend/src/main/java/OpenAI/InterfaceOpenAI.java``` for convenience. This means you can make edits in those modelfiles for tweaking system prompts and parameters, and this will affect both Ollama and OpenAI models.  
The OpenAI key should be added as an environmental variable called ```OPENAI_API_KEY```.

# How to build?
All builds should be done from the root directory, the one that contains the two (backend, frontend) Maven projects.
## Just the backend (Ollama + API endpoints)
### From scratch / day-to-day starting
```
docker compose up -d
```
In this case if building from scratch, you have to wait for models to download, those will appear in docker logs.
Now for stop the running container but preserve modules in volume.
```
docker compose down
```
### Only backend changes
```
docker compose build backend
```
### WARNINGS:
The following deletes volumes with all models.
```
docker compose down -v
```
The following command is unsupported, ollama builds with normal builds, only backend has special container initialization:
```
docker compose build ollama
```
## Backend + frontend
Every other command remains the same, except the building. The following docker profile was created to allow for easy frontend use. This starts all three containers.
```
docker compose --profile frontend up -d
```

# Database access (h2-console)
- Driver class: ```org.h2.Driver```
- DB url: ```jdbc:h2:file:./data/db```
- Username: ```sa```

# Reviewer
Inside of ```reviews/``` run the same commands as for backend.
```
docker compose up
docker compose down
```
The problem set files should be added to ```reviewer/data```. The number does not matter.  
The resulting file will be in ```reviewer/reviews/reviews.csv```. It's important not to directly edit this file, it's already set up to use.  
This system supports container restarts and page refreshes, but for two or more graders there are two recommended options:
- For asynchronous editing, one grader does their part and then sends the ```reviews.csv``` file to the next grader, who puts it in the same directory and keeps going from there.
- Make a master dataset, scramble it, split it, and then process the two files separately (join them later in spreadsheet software).
