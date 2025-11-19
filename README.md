# Claude Code Permission Bypass Vulnerability

## Summary

Claude Code v2.0.42 contains a boundary-dependent access control issue where IAM deny rules are not enforced for file operations outside the project root directory. The documented IAM permission priority (Deny > Ask > Allow) is not enforced when operations target resources outside the project root directory.

## Environment

Operating System: Linux

Claude Code Versions:
- Affected: 2.0.42
- Fixed & verified: 2.0.45 (released 2025-11-18)

Execution Context: Local CLI under a standard non-privileged user account

Configuration Files:
• Global user-level config: ~/.claude/settings.json
• Project-level config: <project-root>/.claude/settings.local.json

## Vulnerability Details

CWE-284: Improper Access Control
CVE: Pending MITRE assignment

CVSS v3.1 Base Score (self-assessed): 7.8 (High)
Vector: CVSS:3.1/AV:L/AC:L/PR:N/UI:R/S:U/C:H/I:H/A:H

Attack Vector: User approval of Ask downgrade for denied Bash commands on files outside the project root.

Impact:
- Unauthorized command execution of globally denied Bash commands (rm, docker, sudo, etc.)
- Unauthorized file access outside project scope despite deny rules

## Technical Details

Claude Code IAM documentation specifies permission priority as Deny > Ask > Allow. In v2.0.42, this priority is enforced within the project root. Outside the project root, deny rules are not enforced and behavior exhibits Ask downgrade (deny rules degrading to Ask-level permissions, allowing execution upon user approval).

Project Root: The directory where Claude Code is launched.

Behavior in v2.0.42:
- Inside project root: Deny rules enforced, commands blocked
- Outside project root: Deny rules not enforced, Ask downgrade occurs, execution permitted on user approval

Behavior in v2.0.45:
- Inside project root: Deny rules enforced
- Outside project root: Deny rules enforced

## Configuration Files

This vulnerability is demonstrated using two configuration files included in the conf/ directory:
- conf/settings.json: Global configuration with numerous deny rules
- conf/settings.local.json: Project configuration with allow rules only

The global configuration denies many bash commands. The local configuration adds specific allow rules without overriding global deny or ask rules.

Note: User-specific paths in configuration files have been anonymized (e.g., /home/user/).

## Reproduction

Note: All paths shown below are example paths. Replace with your actual paths.

### 1. Environment Setup

1. Install Claude Code v2.0.42: `npm install -g @anthropic-ai/claude-code@2.0.42`
2. Create directories: /home/user/project/ and /home/user/other/
3. Place example files: /home/user/project/test.txt and /home/user/other/test.txt

### 2. Configuration

Global configuration in /home/user/.claude/settings.json:

```json
{
  "permissions": {
    "deny": ["Bash(cat:*)", "Bash(cd:*)", "Bash(ls:*)"],
    "ask": [],
    "allow": []
  }
}
```

Note: Bash(ls:*) is included for demonstration purposes. The actual global configuration (conf/settings.json) contains numerous deny rules.

Local project configuration in /home/user/project/.claude/settings.local.json: no deny rules.

(If Claude Code was already running, restart the process to ensure permission rules reload.)

### 3. Launch

```bash
cd /home/user/project/
claude
```

Project root set to /home/user/project

### 4. Expected Behavior

- Deny takes priority over Ask and Allow
- Deny rules such as Bash(cat:*) or Bash(cd:*) block all matching commands
- Target path location does not affect deny behavior

For each test case below, request Claude to execute the command. Claude will attempt to use the Bash tool.

### 5. Observed Behavior in v2.0.42

#### Test Case A: In-Project Operations

Command: `cat test.txt`, `cat /home/user/project/test.txt`

Expected: Denied
Observed: Denied

#### Test Case B: Out-of-Project Operations (Absolute Path)

Command: `cat /home/user/other/test.txt`

Expected: Denied
Observed: Ask downgrade occurred; deny rule not enforced; command executed on approval

#### Test Case B-2: Out-of-Project Operations (Relative Path)

Command: `cat ../test.txt`

Expected: Denied
Observed: Ask downgrade occurred; deny rule not enforced; command executed on approval

#### Test Case C: Directory Change

Command: `cd /tmp`

Expected: Denied
Observed: Ask downgrade occurred; deny rule not enforced; command executed on approval; working directory reset to /home/user/project/ by sandbox

#### Test Case D: Directory Listing

Command: `ls /var/log/`

Expected: Denied
Observed: Ask downgrade occurred; deny rule not enforced; command executed on approval

| Target Location      | Enforcement Result |
|---------------------|-------------------|
| Inside project root  | Deny enforced      |
| Outside project root | Ask downgrade      |

### 6. Observed Behavior in v2.0.45

All test cases (in-root and out-of-root):
- Deny rules enforced
- Commands blocked before execution
- Ask downgrade not observed
- Boundary-dependent behavior not observed

This behavior was re-tested and confirmed to occur in versions 2.0.42 and 2.0.44, and to no longer occur in version 2.0.45. The fix was verified through repeated reproduction under identical test conditions.

## References

- GitHub Issue: https://github.com/anthropics/claude-code/issues/11662
- HackerOne Report: #3426886 (closed as Informative)
- IAM Documentation: https://code.claude.com/docs/en/iam
- Release Notes: Claude Code v2.0.45 (PermissionRequest hook implementation)
- CVE: Pending MITRE assignment

## Timeline

All dates in UTC.

- 2025-11-15: GitHub Issue #11662 filed
- 2025-11-15: HackerOne Report #3426886 submitted
- 2025-11-18: HackerOne Report closed as Informative
- 2025-11-18: GitHub Issue #11662 closed
- 2025-11-18: Claude Code v2.0.45 released
- 2025-11-18: Fix verified and confirmed

## Vendor Response

Anthropic classified this issue as Informative on HackerOne, stating it does not represent a security vulnerability. The fix was implemented in v2.0.45. Release notes document implementation as PermissionRequest hook feature.

## Licensing Note

This repository is dedicated to the public domain under the Unlicense.
However, the contents of the `licensed/` directory are NOT part of the public domain dedication.
They are reproduced verbatim from Anthropic documentation and remain © Anthropic PBC.

---

All technical observations in this report are based solely on reproducible behavior observed in Claude Code v2.0.42 under the test environment described above.

This report was prepared using Claude Sonnet 4.5.
