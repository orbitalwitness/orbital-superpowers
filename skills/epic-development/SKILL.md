---
name: epic-development
description: Pick up the next available task from a Jira epic, assign it to yourself, transition to In Progress, and start development with an isolated git worktree
allowed-tools:
  - mcp__atlassian__atlassianUserInfo
  - mcp__atlassian__getAccessibleAtlassianResources
  - mcp__atlassian__searchJiraIssuesUsingJql
  - mcp__atlassian__getJiraIssue
  - mcp__atlassian__editJiraIssue
  - mcp__atlassian__transitionJiraIssue
  - mcp__atlassian__getTransitionsForJiraIssue
  - Skill
  - Bash
  - Read
  - Grep
  - Glob
---

# Epic Development Skill

Pick up the next available task from a Jira epic and start development in an isolated environment.

## Usage

Invoke with an epic key:
```
/epic-development PROJ-123
```

## Workflow

### Step 1: Get Current User

Use `mcp__atlassian__atlassianUserInfo` to get the current user's account ID for assignment.

### Step 2: Find Next Available Task

Search for unassigned tasks in the epic, ordered by priority:

```
JQL: parent = {EPIC_KEY} AND status = Backlog AND assignee IS EMPTY ORDER BY priority DESC, created ASC
```

Priority order: Highest > High > Medium > Low > Lowest (Jira sorts priority DESC to get highest first)

If no unassigned tasks exist, search for tasks already assigned to the current user that are in Backlog status.

### Step 3: Display Task Options

Show the user the top 3 available tasks with:
- Issue key and summary
- Priority
- Brief description (first 200 chars)

Ask the user which task they want to pick up, or let them specify a different issue key.

### Step 4: Assign and Transition

Once a task is selected:

1. **Assign the task** to the current user using `mcp__atlassian__editJiraIssue`:
   ```json
   {"assignee": {"accountId": "{ACCOUNT_ID}"}}
   ```

2. **Get available transitions** using `mcp__atlassian__getTransitionsForJiraIssue` to find the "In Progress" transition ID

3. **Transition to "In Progress"** using `mcp__atlassian__transitionJiraIssue`:
   ```json
   {"transition": {"id": "{IN_PROGRESS_TRANSITION_ID}"}}
   ```

### Step 5: Confirm Task Pickup

Output a brief summary:
```
Task picked up: {ISSUE_KEY} - {SUMMARY}
Jira status: In Progress
```

### Step 6: Create Isolated Development Environment

**IMPORTANT: Do not stop here. Immediately invoke the git worktree workflow.**

Use the `Skill` tool to invoke `superpowers:using-git-worktrees`:

```
Skill(skill: "superpowers:using-git-worktrees")
```

This will:
- Create a git worktree for isolated development
- Set up the environment properly
- Allow parallel work without affecting other branches

### Step 7: PR Transition Reminder

When development is complete and a PR is created, transition the Jira ticket to "In Review" (or equivalent status):

1. Use `mcp__atlassian__getTransitionsForJiraIssue` to find the review transition
2. Use `mcp__atlassian__transitionJiraIssue` to move the ticket

## Notes

- Transition IDs vary by Jira project configuration. Always use `getTransitionsForJiraIssue` to discover available transitions.
- The skill requires the Atlassian MCP server to be configured with appropriate permissions.
- Works best when combined with `superpowers:using-git-worktrees` for isolated development.
