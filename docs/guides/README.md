# 📘 Feature Guides

> In-depth guides for understanding CareCircle features and systems.

---

## Guides in This Section

| Guide | Description | Best For |
|-------|-------------|----------|
| [Project Overview](./PROJECT_OVERVIEW.md) | Complete project walkthrough | Understanding the full system |
| [Authentication](./AUTHENTICATION.md) | JWT, sessions, email verification | Understanding auth flows |

---

## Project Overview

The [Project Overview](./PROJECT_OVERVIEW.md) covers:

- **What CareCircle solves** - The problem and solution
- **Architecture** - How components fit together
- **Monorepo structure** - Apps and packages
- **Database design** - Entities and relationships
- **Event system** - Real-time updates
- **Running the project** - Development setup
- **Deployment** - Production considerations

**Start here** if you're new to the project.

---

## Authentication

The [Authentication Guide](./AUTHENTICATION.md) covers:

- **Token architecture** - Access & refresh tokens
- **Cookie security** - HTTP-only, secure, SameSite
- **Authentication flows** - Login, register, refresh
- **Family invites** - Invite and accept flow
- **Role-based access** - Admin, Caregiver, Viewer
- **Email system** - Verification, password reset
- **Testing** - Manual and automated tests

**Read this** to understand how auth works end-to-end.

---

## Quick Links by Topic

### "How do I...?"

| Question | Answer In |
|----------|-----------|
| Understand the full system? | [Project Overview](./PROJECT_OVERVIEW.md) |
| Run the project locally? | [Project Overview](./PROJECT_OVERVIEW.md#10-running-the-project) |
| Understand the database? | [Project Overview](./PROJECT_OVERVIEW.md#4-database-design) |
| Implement authentication? | [Authentication](./AUTHENTICATION.md) |
| Handle JWT tokens? | [Authentication](./AUTHENTICATION.md#1-understanding-tokens) |
| Set up family invites? | [Authentication](./AUTHENTICATION.md#34-family-invite-flow) |
| Test auth flows? | [Authentication](./AUTHENTICATION.md#7-testing-guide) |

---

## Related Documentation

| Topic | Location |
|-------|----------|
| API patterns | [Architecture → API](../architecture/API_ARCHITECTURE.md) |
| Frontend patterns | [Architecture → Frontend](../architecture/FRONTEND_ARCHITECTURE.md) |
| Event system | [Architecture → Events](../architecture/EVENT_DRIVEN.md) |
| Setup help | [Getting Started](../getting-started/) |
| Production engineering | [Engineering Mastery](../engineering-mastery/) |

---

_Back to [Documentation Index](../README.md)_
