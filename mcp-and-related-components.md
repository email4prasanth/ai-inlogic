# MCP and Related Components

This note organizes the transcript's discussion of the Model Context Protocol (MCP), AI agents, tool servers, knowledge bases, Azure DevOps, Postman, security testing, and Langflow.

## 1. What Is MCP?

**MCP** stands for **Model Context Protocol**. It provides a standard way for AI applications to connect to tools, systems, and contextual data.

### Core Idea

> Why not deploy these tools through a server made specifically for LLMs?

An MCP server can expose a collection of tools that an AI application or agent can discover and call. FastMCP is one framework used to build MCP servers, similar in spirit to using FastAPI to build web APIs.

| FastAPI | FastMCP |
|---|---|
| Builds web APIs and services | Builds MCP servers for AI applications |
| Exposes HTTP API endpoints | Exposes tools, resources, and prompts through MCP |
| General-purpose API framework | Framework focused on MCP integrations |

### How It Works

1. Build the required tools.
2. Define their names, inputs, outputs, and behavior.
3. Expose them through an MCP server.
4. Connect an MCP-capable AI application to the server.
5. Allow the agent to select and invoke the appropriate tools.

## 2. MCP Server: The Tool-Access Layer

The MCP server acts as a standardized access layer between an AI application and its tools.

```text
LLM / Agent
     |
     v
 MCP Server
     |
     +---- Tool 1
     +---- Tool 2
     +---- Tool 3
     +---- Tool 4
```

Instead of implementing every business operation inside the agent, the application invokes the appropriate external capability through MCP.

## 3. LLM, Tools, and Agent

```text
User Input
    |
    v
  Agent
    |
    +------ LLM (reasoning and generation)
    |
    +------ MCP Tools
              |
              +---- Azure DevOps
              +---- Helpdesk
              +---- CRM
              +---- Knowledge Base
              +---- Postman
              +---- Other Services
```

- **LLM:** Interprets input, reasons over context, and generates responses.
- **Tools:** Provide controlled access to external data, services, and actions.
- **Agent:** Coordinates the LLM and tools to work toward an outcome.

> We combine the LLM and tools to create the agent.

## 4. Azure DevOps MCP

The transcript describes an Azure DevOps (ADO) MCP integration that exposes dozens of tools.

### Example ADO Tools

| Tool or capability | Purpose |
|---|---|
| `repo_get_branch_by_name` | Get a branch by name |
| `repo_list` | List repositories |
| `repo_get` | Get repository information |
| Pipeline access | Inspect or interact with pipelines |
| Deployment access | Inspect or interact with deployments |
| Repository access | Work with repositories and their contents |

Available operations depend on the permissions granted to the Azure DevOps identity or personal access token used by the integration.

```text
AI Agent
   |
   v
Azure DevOps MCP
   |
   +-- Repositories
   +-- Branches
   +-- Pipelines
   +-- Deployments
   +-- Other ADO Operations
```

## 5. Security Considerations with MCP

An MCP connection does not automatically make data safe. Security depends on the model provider, MCP server, agent implementation, authentication method, permissions, logging, and data-handling policies.

### Data Exposure Considerations

| Item | Guidance |
|---|---|
| Source code | Share only when required and permitted for the task |
| API keys | Do not send to the model |
| Credentials | Do not send to the model |
| Hard-coded secrets | Remove and rotate them immediately |
| Personal or confidential data | Minimize, redact, and handle according to policy |

An agent analyzing a repository may encounter any information stored in that repository. If a developer has hard-coded an API key, the repository already contains a security vulnerability, and an agent with repository access may be able to read it.

Recommended controls include:

- Grant least-privilege access.
- Keep secrets in a secret manager rather than source control.
- Add secret scanning and redaction before model calls.
- Restrict which tools an agent can invoke.
- Require confirmation for sensitive or destructive actions.
- Audit tool calls and access logs without recording secrets.
- Review the data-retention policy of every involved service.

## 6. Knowledge Base and Vector Context

### Why a Knowledge Base Is Needed

The current checklist-generation process uses:

- Functional Design Specifications (FDS)
- Figma designs
- User stories or work-item details

These sources may not fully capture impact analysis, regression scenarios, or backward-compatibility concerns.

### Proposed Solution

The planned knowledge base will contain information such as:

- Previously finalized FDS documents
- Platform details
- Earlier system knowledge
- Historical changes

A vector-retrieval integration can connect this knowledge base to the checklist-generation agent so that it can retrieve relevant historical context.

```text
FDS       Figma       User Story
 |          |             |
 +----------+-------------+
            |
            v
      Checklist Agent
            |
            +---- Vector Retrieval / MCP Tool
                          |
                          v
                   Knowledge Base
                          |
                          v
               Previous FDS Documents
               and Platform Information
                          |
                          v
                Impact and Regression
                       Scenarios
```

> With vector context connected to the checklist-generation agent, it can generate impact scenarios using relevant historical platform information.

## 7. Code Base vs. Knowledge Base

The transcript notes that source code alone may not clearly represent every business or regression impact.

| Aspect | Code base | Knowledge base |
|---|---|---|
| Primary purpose | Implementation and security analysis | Historical, platform, and impact context |
| Useful for | Code understanding, vulnerability analysis, and VAPT | Impact analysis and regression-test planning |
| Limitation | May not explain business behavior or historical decisions | Quality depends on coverage, freshness, and retrieval accuracy |

### Code-Base Scope Described in the Transcript

- Repository branch: `DevMaster` for MMHMC
- Scope: Complete frontend and backend
- Repositories: One frontend repository and one backend API repository
- Initial phase: Web application only, not the mobile application
- Primary use case: VAPT and security analysis rather than checklist generation

## 8. VAPT and Security-Testing Agent

The security agent inspects code repositories to identify possible weaknesses and assess whether they may be exploitable.

```text
Code Repositories
       |
       v
Security / VAPT Agent
       |
       +---- Understand the code
       +---- Identify possible weaknesses
       +---- Analyze exploitability
       |
       v
Security Findings and Evidence
```

> If the agent identifies a potential weakness, it should determine whether that weakness is exploitable.

### Status Described in the Transcript

- The code repositories have been provided.
- The agent can access the two repositories for code-base understanding.
- This portion of the initial setup is complete.

## 9. Postman MCP and Tools

The Postman integration can provide capabilities such as:

- Creating or accessing workspaces
- Working with API collections and requests
- Supporting API testing workflows
- Performing other permitted Postman operations

```text
AI Agent
   |
   v
Postman MCP / Tools
   |
   +---- Workspaces
   +---- API Requests
   +---- API Testing
   +---- Other Postman Operations
```

The central idea is that API interactions normally performed manually can be exposed as controlled, agent-driven tool operations.

## 10. Langflow

Langflow is a low-code platform for building and orchestrating AI workflows and agents. The transcript indicates that it was selected partly because of time constraints, while acknowledging that low-code platforms can limit customization.

```text
             Langflow
                |
        +-------+-------+
        |               |
       LLM          MCP Tools
                        |
              +---------+---------+
              |         |         |
             ADO       KB      Postman
```

Langflow operates at the workflow or application-orchestration layer; it is not MCP itself.

## 11. MCP-Focused Architecture

```text
                         USER
                          |
                          v
                    AI AGENT / LLM
                          |
        +-----------------+------------------+
        |                 |                  |
        v                 v                  v
   MCP Tools        Knowledge Base      Other Tools
        |                 |                  |
   +----+----+            |             +----+----+
   |         |            |             |         |
  ADO     Postman         |         Helpdesk     CRM
                          |
                          v
                 Vector Retrieval Tool
                          |
                          v
                   Historical Data,
               FDS, and Platform Context
```

## 12. Summary of MCP-Related Components

| Component | Role |
|---|---|
| MCP | Standard protocol connecting AI applications to tools and contextual data |
| FastMCP | Framework for building MCP servers |
| MCP server | Standardized access layer that exposes capabilities to AI applications |
| ADO MCP | Azure DevOps tools for repositories, branches, pipelines, and deployments |
| Postman MCP | Postman capabilities for workspaces and API workflows |
| Vector retrieval tool | Retrieves relevant knowledge for an agent |
| Knowledge base | Stores historical FDS documents, platform details, and previous changes |
| Code base | Provides implementation context for code and security analysis |
| VAPT agent | Supports vulnerability analysis and exploitability assessment |
| Langflow | Low-code workflow and agent-orchestration platform |
| LLM | Reasoning and generation component |
| Agent | Coordinates reasoning, instructions, and tool use toward an outcome |
| Model provider | Runs the model used by the application, subject to its deployment and data policies |

## 13. Main Idea in Simple Terms

| Component | Purpose |
|---|---|
| LLM | Interprets information and generates a response |
| Agent | Coordinates the work needed to achieve the goal |
| MCP | Standardizes access to tools and context |
| Knowledge base | Provides historical and domain knowledge |
| Vector retrieval | Finds relevant knowledge-base content |
| Langflow | Orchestrates the agent or workflow |
| External systems | Provide capabilities such as ADO, Postman, Helpdesk, and CRM |
| Final output | May be an investigation report, checklist, impact scenario, security finding, or test result |

## 14. Current and Planned Status

### Current or Working

- Langflow and agent setup
- MCP tools for ADO, Postman, and Helpdesk
- CCD case-number investigation
- Checklist generation using FDS, Figma, and user stories
- Code-base access for VAPT and security analysis

### Being Developed or Planned

- A more complete knowledge base
- Vector-context integration
- Impact and regression checklist generation
- Broader automated web and mobile test execution

## 15. Key Transcript Quotes

> They released what we call MCP: Model Context Protocol—a very powerful tool.

> Why not deploy these tools through a server made specifically for LLMs?

> You create the tools, expose them as a collection, and deploy the server.

> With vector context, we will connect the knowledge base to the checklist-generation agent.

> The code base is not always a good representation of the business impact that might occur.

> API keys should not be exposed; if one is hard-coded in a repository, that is already a security concern.

> Interactions performed manually can also be exposed as agent-driven tool operations.
