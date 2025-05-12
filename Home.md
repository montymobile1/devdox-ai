# DevDox AI - System Architecture

## 1. Overview

**DevDox AI** is an open-source, AI-powered platform designed to help developers automate their software development lifecycle (SDLC). It interacts directly with Git repositories to analyze, document, and contribute code with minimal manual effort, helping developers to code, document, and ship faster—without burnout.

The system follows a microservices architecture with modular components that handle specific aspects of the application:
- User interfaces
- API services
- Context building and management
- Code automation agents

![DevDox AI System Overview](https://mermaid.ink/img/pako:eNp1ksFuwjAMhl_FyqlIewBpHDiFsY0DU6Vdpp5CStqitiFRkmoT4t1nQkdBbLmk_r78_nFSeSkaFZQo9LaTo6ZfI1urdK_UhjZw0GCW8UaOJsU6gUbGxnxfJ8T9E_oZ3QVRGW_EB0zxqxr7xsQHrsdNz5v8CW3EO8xvTj-3bYWF2z04xTnVy1yp0Xhq_K07tFrUHkqPjaSVNFm5WfgcSmCrRjqaRhivn06n5PgSF38u0o1PPMU4irlcNSY5Zxl-0DzLDzLwkh8zNWRG9a6Uh9k8kZ6j3Zb6C0oJHW2MtMjnHbwFz8OUe-jlIptPfOy31-7WuQxkbE-vUqgmxI0Jlc3jOUNZGXd_6sGtEjtFjUuJ0OW7NMU9m1QgvXKmwXn5HHtbYZCdUlgcRTPurcmJiA5nAZ-zL22BVaGwP_HEtfYY_0y8oi6k3QbjkCt_Jm-WS84hcTN8UFx-ATzTlvE?type=png)

## 2. Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | Vite + React + TypeScript |
| API Backend | Python + FastAPI |
| Agent Service | Python + FastAPI |
| Database | Supabase (PostgreSQL) |
| Queue System | Supabase (initial), Redis (future) |
| Vector Store | Supabase (pgvector) |
| Additional DB | MongoDB (planned), In-memory (planned) |
| Hosting | Open-source / Self-hosted |

## 3. Component Architecture

### 3.1 devdox-ai-portal

**Purpose**: Landing page and management portal for end users

**Functionality**:
- User account creation and login (via Git OAuth)
- Git project access management
- Git token management
- Repository connection setup
- Token usage monitoring and dashboards
- Administrative tasks
- User preference settings (code style, documentation format, etc.)

**Technology**: React + Vite + TypeScript

### 3.2 devdox-ai-portal-api

**Purpose**: Backend API service for the portal

**Functionality**:
- Handles all API requests from the portal UI
- Authentication and authorization
- Git token secure storage and management
- Repository listing and selection
- User preference storage
- Token usage tracking and analytics
- Data persistence and retrieval

**Technology**: FastAPI (Python)

### 3.3 devdox-ai-agent

**Purpose**: Core product facing customers for code automation

**Functionality**:
- Code creation and maintenance automation
- Git operations management (commits, pull requests, etc.)
- Reporting on code contributors, commits, PRs, and MRs
- Integration with deployment tools
- Documentation and report generation
- Code quality analysis
- Can be accessed via API or MCP (from client applications)

**Technology**: FastAPI (Python)

### 3.4 devdox-ai-context

**Purpose**: Context management service for code repositories

**Functionality**:
- Creates and maintains code context for repositories
- Collects user preferences and project requirements
- Manages code quality standards and practices
- Stores context from previous operations for continuous improvement
- Uses a queue system to process context creation requests
- Producers include the Agent or Portal API (when users request analysis)
- Maintains vector embeddings of code for semantic understanding

**Technology**: FastAPI (Python)

## 4. System Communication Flow

![DevDox AI Communication Flow](https://mermaid.ink/img/pako:eNp1ksFuwjAMhl_FyqlI5QHTOHRiY1yQJu0y9RRSaItag6MkYhPi3edAR0FsuaT-vvz-cTJaSmvYMAzVttOjpF8je6N0r8WGNnCQoJbRRo4qwTqGRkbafF8nxP0z-hndBVEZbdgHTOGrGnlj4gOu1abn9f6ELuQd5ncnP7dtjlm63Qkn2ad6mSszGk-Nv3OHVrPaQemwkbQSJsg3C59BCdyoka7KkMavp9MpOr6E2Z-LdOMTT3EcRlwuGpOcs4w-aJ7l-xh4yZeZGjKjeleKY3qeSJdod1v1F5QCOtoYaeHMO_gIHocp99DLRTaf-MRvr91tcBnI2J5ehVBNiBsTKpvHc4GyMu7-1INbJXaKGpcSoctPaYqPbFKB9MqZBuflc-xthUH2SmBxFM24tzYnIjqcBXyevrQZVoXC_sSJ19rj_DPRirrQdhuMQ678mbxZLniKiZvhg-LyC09Nlmw?type=png)

The communication flow between components works as follows:

1. **User Interaction**:
   - Users interact with the devdox-ai-portal web interface
   - The portal communicates with devdox-ai-portal-api for all operations

2. **Data Storage**:
   - The portal API interacts with Supabase (PostgreSQL) for data persistence
   - User preferences, tokens, and repository information are stored securely

3. **Context Generation**:
   - When a repository analysis is requested, the portal API or agent creates a task in the queue
   - The devdox-ai-context service consumes these tasks and processes repository analysis

4. **Code Automation**:
   - Based on contexts and user requests, the devdox-ai-agent interacts with:
     - Git repositories (via API)
     - CI/CD systems
     - Documentation generators

## 5. Queue & Context Management

- **Initial Queue Implementation**: Supabase (using `realtime` or `cron` jobs + table triggers)
- **Planned Queue Implementation**: Redis with Celery/RQ for high-volume asynchronous operations
- **Producers**: 
  - Portal API (when users request repository analysis)
  - Agent (when automation triggers context updates)
- **Consumer**: Context service (continuously listens to queue and processes analysis jobs)

## 6. Security & Access Control

- Git token management follows OAuth best practices with secure storage
- Role-based access control for the portal (admin vs. regular user)
- Secure data sharing between components with authentication
- Comprehensive Git operation logging for audit trails
- Encrypted storage of sensitive data

## 7. Future Enhancements

- Add MongoDB for flexible, document-based storage
- Implement Redis for faster in-memory caching and job queuing
- Add support for multiple SCM tools (GitHub, GitLab, Bitbucket)
- Integration with issue tracking systems (Jira, GitHub Issues) for automated ticket creation
- Enhanced AI models for better code understanding and generation
- Self-healing and scaling capabilities
- Support for custom plugins and extensions

---

*This documentation is maintained as part of [DEV-1](https://montyholding.atlassian.net/browse/DEV-1) and will be updated as the architecture evolves.*
