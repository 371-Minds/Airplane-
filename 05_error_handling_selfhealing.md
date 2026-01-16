# Error Handling & Self-Healing Patterns

## Secure Agent Guardrails (Policy + Self-Critique)
**Error detection methods**: The policy engine detects oversized inputs, prompt injection keywords, role confusion markers, exfiltration language, and unknown tools. It also flags PII patterns, secret tokens, and insecure URLs during postflight review. 【F:AI Agents Codes/secure_ai_agent_with_guardrails_marktechpost.py†L18-L115】
**Recovery procedures**: If preflight checks fail, requests are blocked with a reason. Postflight audits redact PII/secret tokens and re-run the audit when risk is medium/high. Unsafe tool requests are blocked by allowlist checks. 【F:AI Agents Codes/secure_ai_agent_with_guardrails_marktechpost.py†L99-L147】
**Logging/monitoring integration**: Each request produces a structured log with the chosen tool, decision reasons, and a hashed request ID; audit results are printed in the test loop. 【F:AI Agents Codes/secure_ai_agent_with_guardrails_marktechpost.py†L129-L167】
**Circuit breaker patterns**: A basic rate-limit gate (`RATE_LIMIT_WINDOW`) blocks repeated calls too quickly, acting as a simple circuit-breaker-like throttle. 【F:AI Agents Codes/secure_ai_agent_with_guardrails_marktechpost.py†L22-L107】

## Advanced Multi-Tool Agent (Retry + Failure Metadata)
**Error detection methods**: Detects unknown tools, failed tool executions, and exceptions raised during tool calls. Warnings are emitted for long code outputs or missing test cases. 【F:Agentic AI Codes/advanced_multitool_agentic_ai_marktechpost.py†L194-L207】【F:Agentic AI Codes/advanced_multitool_agentic_ai_marktechpost.py†L285-L314】
**Recovery procedures**: `execute_with_recovery` retries tool execution up to a configured limit and returns structured failure metadata if retries are exhausted. 【F:Agentic AI Codes/advanced_multitool_agentic_ai_marktechpost.py†L285-L314】
**Logging/monitoring integration**: Execution history is appended with tool name, success flag, and timestamp for each run. 【F:Agentic AI Codes/advanced_multitool_agentic_ai_marktechpost.py†L300-L305】
**Circuit breaker patterns**: Not explicitly implemented beyond bounded retries. 【F:Agentic AI Codes/advanced_multitool_agentic_ai_marktechpost.py†L285-L314】

## SmartWebAgent (Fallback Paths + User-Facing Errors)
**Error detection methods**: Catches initialization failures, stream errors, direct invocation failures, and extraction errors for individual URLs. 【F:smartwebagent_tavily_gemini_webintelligence_marktechpost2.py†L77-L146】
**Recovery procedures**: Falls back from streaming to direct LLM invocation; if that fails and URLs are provided, extracts content first and retries analysis using a summary. Final failure returns a friendly error message with a connectivity hint. 【F:smartwebagent_tavily_gemini_webintelligence_marktechpost2.py†L112-L146】
**Logging/monitoring integration**: Console logs report initialization errors, streaming errors, and fallback attempts, with warnings for degraded paths. 【F:smartwebagent_tavily_gemini_webintelligence_marktechpost2.py†L77-L146】
**Circuit breaker patterns**: Not present; retries are implemented as staged fallbacks rather than a breaker. 【F:smartwebagent_tavily_gemini_webintelligence_marktechpost2.py†L112-L146】

## MCP OAuth Middleware (Auth Error Handling)
**Error detection methods**: Detects missing/invalid `Authorization` headers, invalid bearer tokens, and JSON parsing errors for request bodies. 【F:OAuth 2.1 for MCP Servers/auth.py†L34-L53】
**Recovery procedures**: Returns a 401 JSON response with `WWW-Authenticate` metadata for discovery; requests to `/.well-known/` bypass the middleware. 【F:OAuth 2.1 for MCP Servers/auth.py†L31-L62】
**Logging/monitoring integration**: Logging is configured at module load; no explicit per-request structured logging beyond exceptions. 【F:OAuth 2.1 for MCP Servers/auth.py†L11-L16】
**Circuit breaker patterns**: Not present. 【F:OAuth 2.1 for MCP Servers/auth.py†L28-L64】

## Alpha Vantage MCP Tool (Graceful Degradation)
**Error detection methods**: Treats HTTP errors and exceptions during external API calls as failures; checks response shape for expected `feed` key. 【F:OAuth 2.1 for MCP Servers/finance.py†L13-L34】
**Recovery procedures**: Returns `None` on API errors and a user-facing message when data is missing. 【F:OAuth 2.1 for MCP Servers/finance.py†L13-L34】
**Logging/monitoring integration**: Not present. 【F:OAuth 2.1 for MCP Servers/finance.py†L13-L34】
**Circuit breaker patterns**: Not present. 【F:OAuth 2.1 for MCP Servers/finance.py†L13-L34】
