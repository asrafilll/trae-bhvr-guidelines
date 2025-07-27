# AI Agent System for BHVR Development

A comprehensive AI agent system designed for building modern web applications using the BHVR stack (Bun + Hono + Vite + React) with documentation-driven development and single feature focus.

## 🎯 Overview

This repository contains three specialized AI agents that work together to streamline the development process from product planning to code implementation:

- **🔍 Product Agent** - Strategic planning and product requirements
- **⚙️ Development Agent** - Task management and development coordination
- **👨‍💻 Engineer Agent** - Code implementation and technical execution

## 🏗️ Tech Stack (BHVR)

- **Runtime**: Bun (package manager + JavaScript runtime)
- **Backend**: Hono + SQLite + Prisma
- **Frontend**: React 19 + Vite + TanStack React Query + React Router v7 + Tailwind CSS
- **Architecture**: Monorepo with client/server/shared workspaces
- **Testing**: Comprehensive backend testing, optional frontend testing

## 📁 Project Structure

```
project/
├── client/              # React + Vite frontend
│   ├── src/            # React components and logic
│   ├── public/         # Static assets
│   └── dist/           # Built frontend assets
├── server/             # Hono API + static server
│   ├── src/            # Hono routes and business logic
│   ├── prisma/         # Database schema and migrations
│   └── dist/           # Built server code
├── shared/             # TypeScript types used by both
│   └── types/          # Shared type definitions
├── docs/               # Project documentation
│   ├── bhvr-*.md       # BHVR technical documentation
│   ├── prd.md          # Product Requirements Document
│   ├── user-stories.md # User Stories
│   ├── implementation.md # Current development tasks
│   └── ...             # Other project docs
└── agents/             # AI Agent prompts
    ├── product-agent.md
    ├── development-agent.md
    └── engineer-agent.md
```

## 🤖 Agent System

### 🔍 Product Agent

**Purpose**: Strategic product planning and requirements creation

**Key Features**:

- Creates comprehensive PRDs and user stories
- Analyzes features for impact and feasibility
- Prioritizes development roadmap
- Bridges business objectives with technical implementation

**When to Use**: Start here for new features, product planning, or requirement clarification

### ⚙️ Development Agent

**Purpose**: Development coordination and task management

**Key Features**:

- Breaks features into 30-60 minute implementable steps
- Manages development workflow
- Ensures single feature focus
- Coordinates between product requirements and engineering

**When to Use**: After product planning, before code implementation

### 👨‍💻 Engineer Agent

**Purpose**: Code implementation and technical execution

**Key Features**:

- Implements features following BHVR patterns
- Creates comprehensive test files for backend functionality
- Provides step-by-step implementation guidance
- Maintains code quality and architectural compliance

**When to Use**: For actual code implementation after planning is complete

## 🚀 Getting Started

### 1. Setup Documentation Structure

Create the `/docs/` folder with required documentation:

```bash
mkdir docs
# Add your BHVR technical documentation
# Add prd.md and user-stories.md if you have them
# Other project documentation files
```

### 2. Choose Your Agent

**For Product Planning**:

```bash
# Use Product Agent prompt for:
- Creating product requirements
- Analyzing features
- Strategic planning
- Requirements clarification
```

**For Development Coordination**:

```bash
# Use Development Agent prompt for:
- Breaking down features into tasks
- Managing development workflow
- Task prioritization
```

**For Code Implementation**:

```bash
# Use Engineer Agent prompt for:
- Writing actual code
- Creating test files
- Technical implementation
- Following BHVR patterns
```

### 3. Workflow Process

1. **Product Agent** → Creates `implementation.md` and requirements
2. **Development Agent** → Breaks down into granular tasks
3. **Engineer Agent** → Implements code following specifications

## 📋 Key Principles

### Documentation-Driven Development

- All agents prioritize existing documentation
- PRD and User Stories take precedence when available
- BHVR technical documentation guides implementation
- Always update documentation after changes

### Single Feature Focus

- Work on ONE feature at a time until completion
- Break features into 30-60 minute implementable steps
- Complete each step before moving to the next
- Comprehensive testing for backend functionality

### Quality Standards

- Zero syntax errors or warnings
- Comprehensive backend test coverage
- BHVR architecture compliance
- Clean, maintainable code

## 🔧 Configuration

Each agent requires access to:

1. **Documentation Files** (`/docs/` folder)
2. **Context7** (for latest framework documentation)
3. **BHVR Technical Guides** (setup, backend, frontend, shared, deployment)

## 📚 Documentation Priority

1. **Critical**: `implementation.md`, `prd.md`, `user-stories.md`
2. **BHVR Technical**: All `bhvr-*.md` files
3. **Specification**: Architecture, API, UI/UX documentation
4. **Reference**: Bug tracking, coding standards

## 🎯 Success Metrics

- **Complete Feature Implementation**: No partial work
- **Test Coverage**: Comprehensive backend testing
- **Documentation Alignment**: Follows all specifications
- **BHVR Compliance**: Maintains monorepo structure
- **Quality Assurance**: Zero errors/warnings

## 🤝 Contributing

This agent system is designed to be:

- **Modular**: Each agent serves a specific purpose
- **Scalable**: Can be extended with additional specialized agents
- **Flexible**: Adapts to different project requirements
- **Maintainable**: Clear separation of concerns

## 📄 License

MIT License - Feel free to use and modify for your projects.

## 🔗 Links

- [BHVR Framework](https://bhvr.dev/)
- [Bun](https://bun.sh/)
- [Hono](https://hono.dev/)
- [Vite](https://vitejs.dev/)
- [React](https://react.dev/)

---

**Built for modern web development with AI-assisted workflows** 🚀
