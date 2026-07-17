### What this is
A lightweight agentic prototype that evaluates 5 provided sample claims (Appendix B) against the
travel reimbursement policy (Appendix A) and emits one structured JSON result per claim.

### Setup
```bash
pip install anthropic pandas matplotlib ipywidgets   # ipywidgets optional (interactive UI)
```
Python ≥ 3.10. No other services required.

### Environment variables (optional)
| Variable | Purpose |
|---|---|
| `ANTHROPIC_API_KEY` | Enables the LLM-driven agent loop (model: `claude-sonnet-4-6`). |
| `AGENT_MODE` | `llm` (default if key present), `deterministic` to force the fallback. |

If no key is set the notebook automatically uses the deterministic engine — **it still runs
end-to-end** and produces identical tool traces and decisions.

### How to run
`Kernel → Restart & Run All`. The final code cell prints the required JSON array
(one object per claim with exactly: `claim_id, decision, approved_amount, deducted_amount,
missing_docs, policy_refs, confidence, explanation, tools_used`).

### Key design choices
1. **Tools are deterministic; the LLM orchestrates.** All money math (caps, deductions,
   thresholds) lives in plain Python tools. The LLM decides *when* to call them and how to
   synthesize findings — never invents numbers. This keeps decisions auditable and reliable.
2. **Manual Review is a first-class outcome.** Missing receipts, non-economy airfare, late
   submission, and totals > $2,000 are never force-decided (POL-RCT-02, POL-AIR-01,
   POL-TIME-01, POL-APR-03).
3. **Schema-validated output with repair.** Every result passes `validate_output`; an invalid
   LLM result is repaired by falling back to the deterministic engine for that claim.
4. **Graceful degradation.** LLM → deterministic fallback means zero-dependency review.
