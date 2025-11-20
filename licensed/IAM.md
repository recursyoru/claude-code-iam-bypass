### Configuring permissions

You can view & manage Claude Code's tool permissions with `/permissions`. This UI lists all permission rules and the settings.json file they are sourced from.

* **Allow** rules will allow Claude Code to use the specified tool without further manual approval.
* **Ask** rules will ask the user for confirmation whenever Claude Code tries to use the specified tool. Ask rules take precedence over allow rules.
* **Deny** rules will prevent Claude Code from using the specified tool. Deny rules take precedence over allow and ask rules.
* **Additional directories** extend Claude's file access to directories beyond the initial working directory.
* **Default mode** controls Claude's permission behavior when encountering new requests.

Permission rules use the format: `Tool` or `Tool(optional-specifier)`

A rule that is just the tool name matches any use of that tool. For example, adding `Bash` to the list of allow rules would allow Claude Code to use the Bash tool without requiring user approval.

---

**Note:**
Source: https://code.claude.com/docs/en/iam
This document is an unmodified excerpt of Anthropic's official IAM documentation.
Included here solely as a reference for analyzing and reproducing the described vulnerability.
© 2025 Anthropic PBC – All rights reserved.

This document is reproduced verbatim for the sole purpose of security research,
compatibility analysis, and accurate documentation of a related vulnerability.

No copyright ownership is claimed.
Original rights remain with Anthropic PBC.
