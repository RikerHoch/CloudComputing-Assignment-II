# Final Project Report — [Project Title]

**Course:** BACT7602 Cloud Computing — Whitecliffe  
**Assessment:** Assessment 2 — Group Cloud Project (70%)  
**Team members:** <!-- List all names and student IDs -->  
**Submission date:** End of Week 18  
**Word count:** <!-- Update when complete — target 1,500–2,000 words -->

> **Word count guidance:**
> - Sections 1–4 form the main body: aim for 1,500–2,000 words total.
> - Section headings, tables, code blocks, captions, and references are NOT counted.
> - Appendices are NOT counted and should be used for supporting evidence (diagrams, screenshots, cost breakdowns).
> - Replace every italicised prompt with your own writing, then delete the prompt.

---

## Abstract

<!-- 150 words maximum. Not counted in word limit.
     Write this LAST — once you know exactly what your project achieved.
     Summarise: what you built, why, what technology you used, and the key outcome. -->

*This report presents the design, implementation, and evaluation of [project name], a cloud-based [describe the system] developed as part of BACT7602 Cloud Computing at Whitecliffe. The solution was deployed on Microsoft Azure using [key services] and provisioned through infrastructure as code (Bicep). The project addressed [business problem] by [brief description of solution]. Key outcomes include [2–3 measurable results, e.g. "99.9% uptime achieved during testing, deployment automated from 45 minutes to under 4 minutes, and estimated monthly infrastructure cost of NZD XX"]. Lessons learned and recommended improvements are discussed in the final section.*

---

## 1. Solution Design and Architecture

<!-- Target: 400–500 words. -->

### 1.1 Business Context and Requirements

<!-- Describe the problem your cloud solution addresses.
     Answer: Who is the customer? What do they need? What are the constraints?
     Reference the well-architected framework pillars as a frame for requirements.
     
     Example prompts:
     - What is the business or organisation this system serves?
     - What is the scale (users, data volume, transactions per day)?
     - What are the non-functional requirements (uptime, data residency, budget)?
-->

*[Your organisation or scenario] required a cloud solution that could [describe key requirements]. The primary non-functional requirements identified were: [list 3–5 requirements, e.g. availability target of 99.9%, data must remain within Australia/NZ, monthly budget not to exceed NZD X, system must support Y concurrent users].*

### 1.2 Architecture Overview

<!-- Describe your architecture in prose — the diagram in ARCHITECTURE.md provides the visual; 
     this section explains the WHY.
     
     Cover:
     - What tier/pattern does your architecture follow? (e.g. three-tier, microservices, serverless)
     - Which Azure services were chosen and why (reference your ADRs)?
     - How does data flow through the system?
     - What provider did you use and why?
-->

*The solution follows a [describe architecture pattern, e.g. "three-tier web application architecture"] deployed on Microsoft Azure in the [region] region. Azure was selected as the primary provider because [give specific reasons, e.g. "it offers the best coverage in the australiaeast region for New Zealand workloads, strong identity integration via Entra ID, and native Bicep support for infrastructure as code"]. The architecture diagram is provided in [Appendix A / ARCHITECTURE.md].*

*The key services and their roles are as follows:*

*[Azure service 1] was chosen for [role] because [specific justification — reference your ADR]. [Azure service 2] provides [role]; the alternative of [alternative service] was considered but rejected because [reason]. [Continue for each key service].*

### 1.3 Design Decisions and Trade-offs

<!-- Summarise your 2–3 most significant architecture decisions.
     For each: what did you choose, what alternative did you consider, and why did you choose what you did?
     Full detail is in ARCHITECTURE.md (ADR section) — this is a summary for the report. -->

*Three significant design decisions shaped this architecture:*

*First, [Decision 1 — e.g. "we chose Azure App Service over Azure Kubernetes Service (AKS) for the application hosting layer"]. [Explain reasoning and trade-off in 2–3 sentences.]*

*Second, [Decision 2]. [Reasoning and trade-off.]*

*Third, [Decision 3]. [Reasoning and trade-off.]*

---

## 2. Implementation

<!-- Target: 400–500 words. -->

### 2.1 Infrastructure as Code

<!-- Describe how you used Bicep (or ARM/Terraform) to provision your infrastructure.
     This section should demonstrate that you understand IaC principles, not just that you used it.
     
     Cover:
     - What does your main.bicep do?
     - How are parameters used to support multiple environments (dev/prod)?
     - How did you ensure idempotency?
     - What-if validation process?
-->

*All infrastructure was provisioned using Bicep templates, enabling repeatable, version-controlled deployments. The root template (`infra/main.bicep`) orchestrates [number] reusable modules, each responsible for a logical group of resources: [list modules, e.g. networking, compute, data, security, monitoring]. Parameter files (`dev.parameters.json`, `prod.parameters.json`) allow the same templates to deploy distinct environments with different SKUs and configurations.*

*Idempotency was ensured by [describe your approach, e.g. "using resource existence checks and the `existing` keyword for cross-resource references"]. Before every deployment, the pipeline runs `az deployment group what-if` to preview changes and catch unintended modifications before they are applied.*

*A key challenge encountered during IaC development was [describe one real challenge, e.g. "configuring private endpoint DNS zones for Azure SQL required understanding the interaction between the virtual network, private DNS zone, and Key Vault — this was resolved by creating a dedicated private DNS zone module that links the zone to the correct VNet"]. This reinforced the importance of [lesson learned].*

### 2.2 CI/CD Pipeline

<!-- Describe your pipeline.
     Cover:
     - Which tool (Azure DevOps Pipelines or GitHub Actions)?
     - What stages does the pipeline have?
     - How does the deployment slot / approval gate work?
     - What does the pipeline protect against (linting, what-if)?
-->

*Deployment is automated via [Azure DevOps Pipelines / GitHub Actions]. The pipeline is triggered on every push to the `main` branch and consists of [number] stages:*

1. *[Stage name, e.g. Validate] — [What it does, e.g. "Runs Bicep linting and what-if validation against the development subscription. Fails the pipeline if any resources would be unexpectedly deleted."] This stage catches [type of error] before any changes are applied.*

2. *[Stage name, e.g. Deploy to Dev] — [What it does.]*

3. *[Stage name, e.g. Deploy to Production] — [What it does, e.g. "Requires manual approval from two team members before proceeding."] This gate ensures [benefit].*

*The pipeline connects to Azure using a [service principal / workload identity federation], with credentials stored as pipeline secrets — never in the repository. This follows the principle of [e.g. "least privilege: the service principal holds Contributor access only on the project resource group, not at subscription level"].*

### 2.3 Git Repository and Collaboration

<!-- Describe how your team used version control.
     Cover: branching strategy, code review, commit discipline. -->

*The team used a [feature branch / trunk-based / GitHub Flow] branching strategy. All work was done in short-lived feature branches named `feature/<member>/<description>`. Pull requests required at least one peer approval before merging to `main`. This ensured that [benefit, e.g. "no untested code reached the main branch and all team members maintained visibility of each other's changes"].*

---

## 3. Security, Reliability, and Monitoring

<!-- Target: 300–400 words. -->

### 3.1 Security Controls

<!-- Summarise your security approach. 
     Demonstrate that you understand WHY these controls matter, not just that you implemented them.
     Reference the shared responsibility model where appropriate. -->

*Security was approached using the principle of least privilege throughout. [Service] uses a [managed identity / service principal] with only the permissions required for its function — for example, the App Service managed identity holds the `Key Vault Secrets User` role, allowing it to read secrets but not modify or delete them.*

*All application secrets (connection strings, API keys) are stored in Azure Key Vault with [soft-delete / purge protection / private endpoint] enabled. The application retrieves these at runtime, meaning no secrets are present in the codebase, pipeline logs, or environment variables.*

*Network isolation is achieved through [describe: private endpoints / NSGs / subnet segmentation]. [Describe your most significant network security decision and why it matters.]*

*One security gap identified in our design is [honest acknowledgement, e.g. "we did not implement a Web Application Firewall (WAF) due to cost constraints in the student environment — in a production deployment, Azure Front Door with WAF policies would be added to protect against OWASP Top 10 threats"]. This was a deliberate trade-off accepted for this assessment context.*

### 3.2 Reliability and Availability

<!-- Summarise your reliability design.
     Cover: availability zone usage, autoscaling, SLO, backup strategy. -->

*The solution targets [X]% availability, equivalent to no more than [Y] hours of downtime per year. This is supported by [list reliability controls: e.g. "zone-redundant App Service Plan, automated SQL point-in-time backup with 7-day retention, and autoscale rules that add compute instances when CPU exceeds 70% for five minutes"].*

*The recovery time objective (RTO) is [X hours] and the recovery point objective (RPO) is [Y hours/minutes]. These were validated by [e.g. "a simulated restore test documented in the runbook (Section 4)"]. The backup restoration procedure is documented in [RUNBOOK.md, Section 6].*

### 3.3 Monitoring and Observability

<!-- Describe what you monitor and why those metrics were chosen. -->

*All resources send telemetry to a centralised Log Analytics Workspace (`law-<project>-<env>`). Application Insights provides real-time request tracking, failure rate monitoring, and dependency mapping. Three alert rules are configured: [list each alert, its threshold, and why that threshold was chosen, e.g. "CPU > 80% for 5 minutes — triggers before the autoscaler reaches maximum capacity, giving the team time to investigate"]. Alert notifications are sent to [email / Teams / SMS].*

---

## 4. Reflection and Lessons Learned

<!-- Target: 300–400 words.
     This section is read by markers for evidence of genuine learning.
     Be honest — identifying challenges and what you would do differently 
     demonstrates higher-order thinking than claiming everything went perfectly. -->

### 4.1 What Worked Well

*[Describe 2–3 things that went well and why. Be specific — e.g. "The decision to write Bicep modules early in the project saved significant time: once the networking module was proven correct, it was reused across both dev and prod environments without modification."] *

### 4.2 Challenges Encountered

*[Describe 2–3 genuine challenges your team faced. Explain how you resolved them or why you could not.]*

*Challenge 1: [Description]. We resolved this by [resolution]. This taught us [lesson].*

*Challenge 2: [Description]. [Resolution / what we would do differently].*

### 4.3 What We Would Do Differently

*If we were to rebuild this project, we would [describe 2–3 improvements]. Specifically:*

- *[Improvement 1, e.g. "We would implement a Web Application Firewall from day one rather than leaving it as a future improvement, as security controls are significantly easier to add before application code is written."]*
- *[Improvement 2]*
- *[Improvement 3]*

### 4.4 Alignment with the Azure Well-Architected Framework

*Evaluating our solution against the five pillars of the Azure Well-Architected Framework:*

- **Reliability:** [One sentence on what you achieved and one honest gap.]
- **Security:** [One sentence on what you achieved and one honest gap.]
- **Cost Optimisation:** [One sentence. Include your actual monthly cost estimate.]
- **Operational Excellence:** [One sentence.]
- **Performance Efficiency:** [One sentence.]

*Overall, our solution demonstrates [summarise maturity level]. The most significant gap is [X], which we would address by [Y] in a production-grade implementation.*

---

## References

<!-- Use IEEE or APA format — be consistent. 
     Include all sources cited in the report AND all Microsoft Learn / Azure documentation used. 
     Minimum 5 references expected; 8–12 is typical for this report length.
     References are NOT counted in the word limit. -->

[1] <!-- Author(s), "Title," Publisher, Year. URL -->  
[2] M. Beyer, B. Murphy, N. Renella, K. Kawahara, and B. Treynor, *Site Reliability Engineering: How Google Runs Production Systems*. Sebastopol, CA: O'Reilly Media, 2016.  
[3] Microsoft, "Azure Well-Architected Framework," Microsoft Learn, 2024. [Online]. Available: https://learn.microsoft.com/en-us/azure/well-architected/  
[4] Microsoft, "Bicep documentation," Microsoft Learn, 2024. [Online]. Available: https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/  
[5] <!-- Add your sources -->  

---

## Appendices

*Appendices are not counted in the word limit. Include supporting evidence here.*

### Appendix A — Architecture Diagram

<!-- Embed your architecture diagram here OR reference ARCHITECTURE.md. -->

### Appendix B — Azure Pricing Calculator Estimate

<!-- Screenshot or table from your cost estimate. Include the shareable link. -->

### Appendix C — Pipeline Screenshot

<!-- Screenshot of a successful pipeline run showing all stages completed. -->

### Appendix D — Monitoring Dashboard

<!-- Screenshot of your Azure Monitor / Log Analytics dashboard showing active metrics. -->

### Appendix E — Smoke Test Results

<!-- Screenshot or pasted output of a successful smoke test run (from the Runbook). -->

---

## Individual Contribution Statements

<!-- Each team member writes 100–150 words describing THEIR personal contributions.
     This is separate from the group Contribution Log — this is your personal account.
     Written in first person. -->

### [Member 1 Full Name] — Student ID: [W123456]

*[Write in first person. Describe: which parts of the project you personally designed, built, or led. What decisions did you make? What did you learn? 100–150 words.]*

---

### [Member 2 Full Name] — Student ID: [W123457]

*[First person, 100–150 words.]*

---

### [Member 3 Full Name] — Student ID: [W123458]

*[First person, 100–150 words.]*

---

### [Member 4 Full Name] — Student ID: [W123459]

*[First person, 100–150 words.]*

---

## AI Tool Declaration

<!-- Required. Complete honestly. Markers are aware of AI tool capabilities. -->

This group declares the following AI tool usage in this assessment:

| Tool | Used by | How it was used | How output was verified |
|------|---------|----------------|------------------------|
| <!-- e.g. GitHub Copilot --> | <!-- All members --> | <!-- Code suggestions in VS Code for Bicep --> | <!-- Each suggestion reviewed, tested by running what-if and deployment --> |
| <!-- e.g. Claude --> | <!-- Member 2 --> | <!-- Reviewed architecture decision reasoning --> | <!-- Cross-checked against Microsoft documentation and course materials --> |
| <!-- e.g. ChatGPT --> | <!-- None --> | <!-- Not used --> | <!-- — --> |

All AI-generated or AI-assisted content was reviewed and validated by the team. We take responsibility for all content in this submission.

**Signatures (type full name):**

| Name | Date |
|------|------|
| | |
| | |
| | |
| | |
