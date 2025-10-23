# AGENTS.md for OpenAI Apps

This repository contains an open-source `AGENTS.md` file designed to guide AI coding agents in building OpenAI Apps using the Apps SDK (Find out more at: https://developers.openai.com/chatgpt).

### Technologies Covered

This list was curated by selecting more commonly used technologies. However, don't be afraid of swapping any of these out to cater to your preference!

- **Model Context Protocol (MCP):** The protocol enabling ChatGPT to call backend tools.
- **TypeScript & Node.js:** For MCP server implementations and backend logic.
- **Python & FastMCP:** Alternative MCP server framework for Python developers.
- **React 18+:** For building client-side interactive widgets/components.
- **Esbuild & Vite:** Build tools for bundling React components.
- **OAuth 2.1 & Bearer Tokens:** For secure authentication and authorization.
- **MCP Inspector & MCPJam Inspector:** Tools for local development and debugging.

This file provides setup instructions, build commands, testing workflows, error handling patterns, and deployment guidelines tailored for the first version of OpenAI Apps. It can be customized or extended to fit different development stacks or project needs.

## How to Use This AGENTS.md

1. **Add AGENTS.md to your project root**  
   Create an `AGENTS.md` file at the root of your repository. This file provides detailed build steps, tests, and conventions for coding agents.

2. **Integrate with your favorite AI coding tools**  
   Many agentic coding tools like OpenAI Codex, Aider, Gemini CLI, and others support reading instructions from `AGENTS.md` automatically or via configuration.

3. **Configure your tools to use AGENTS.md**  
   For example:

   - Aider: set `read: AGENTS.md` in `.aider.conf.yml`
   - Gemini CLI: set `{ "contextFileName": "AGENTS.md" }` in `.gemini/settings.json`

4. **Start with this file but adapt as needed**  
   Use this provided `AGENTS.md` as a starting point. Customize its sections to fit your project conventions, development workflows, or testing requirements.

5. **Use nested AGENTS.md for large projects**  
   If you work in a monorepo or multi-package repository, consider using nested `AGENTS.md` files for subprojects to keep instructions focused.

## Contributions are welcome!

Feel free to submit improvements, additions, or fixes to this `AGENTS.md` to enhance support for building MCP servers, UI components, and integrations.

❤️ Have fun building!
