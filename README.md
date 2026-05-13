# Valkyrie

Valkyrie is a reinforcement learning environment for training models to remediate security vulnerabilities in real-world code. Each episode places the model inside an isolated codebase containing an injected CWE-class vulnerability. The model receives a natural-language problem statement, interacts with the codebase through tool calls, and submits a patch. The environment returns a binary reward: +1 when tests pass and no regressions are introduced, 0 otherwise.

Pipeline extends [SWE-smith](https://arxiv.org/abs/2504.21798) (NeurIPS 2025 Spotlight) with CWE-targeted mutation and human curation. Targeting 10,000 task instances across 8 languages and 85+ CWE categories. Currently 20 validated instances with production pipeline active.

---

## Dataset

**Repository**: [ethara/Valkyrie](https://huggingface.co/datasets/ethara/Valkyrie) (private, license: CC-BY-NC-ND-4.0)

### Schema

| Column | Type | Description |
|--------|------|-------------|
| `instance_id` | string | Unique task identifier (`{repo_slug}.{commit_hash}.{mutation_id}`) |
| `patch` | string | Ground-truth fix (unified diff format) |
| `FAIL_TO_PASS` | list\<string\> | Security tests: must pass after fix (reward gate) |
| `PASS_TO_PASS` | list\<string\> | Regression tests: must remain passing (reward gate) |
| `image_name` | string | Docker image URI for isolated execution |
| `repo` | string | Source repository (org/repo format) |
| `problem_statement` | string | Natural-language description of the vulnerability |
| `vulnerability_type` | list\<string\> | CWE identifiers for the injected vulnerability |
| `category` | string | Vulnerability composition: `Atomic CWE` or `Composite CWE` |
| `evaluation` | struct | Baseline agent results (see below) |

### Evaluation Struct

```
evaluation: {
    difficulty: string           // "Easy", "Medium", "Hard"
    num_files_affected: string   // Number of files changed in ground-truth patch
    kimi_k2_5: {
        pass_at_1: string        // "Pass" or "Fail"
        time_of_completion_secs: string
        cost_usd: string
    }
    nova_2_lite: {
        pass_at_1: string        // "Pass" or "Fail"
        time_of_completion_secs: string
        cost_usd: string
    }
}
```

---



---

## Reward Signal

Binary: **+1** or **0**. No partial credit.

```
reward = 1 if (FAIL_TO_PASS all pass) AND (PASS_TO_PASS all pass) else 0
```

- `FAIL_TO_PASS`: Security-specific tests that exercise the injected vulnerability. These must pass after the agent's fix.
- `PASS_TO_PASS`: Pre-existing tests that passed before the vulnerability was injected. These must remain passing (no regressions).

Both conditions required. A patch that fixes the vulnerability but breaks other functionality receives 0.

**Rationale**: Security fixes are pass/fail by nature. A partial fix to a buffer overflow is still exploitable. Sparse binary reward targets RL methods that learn from rare positive signal.


---

## Task Composition Pipeline

### Vulnerability Injection

LLM-guided adversarial mutation followed by human curation:

1. Extract code entities (functions, methods) from established open-source repositories
2. Apply CWE-specific mutation prompts that introduce vulnerabilities:
   - Compile without errors or warnings
   - Preserve function signatures and behavior on non-adversarial inputs
   - Appear as plausible developer mistakes
3. Human curators validate realism, difficulty calibration, and test coverage adequacy
4. Test verification confirms reward signal fires correctly (vulnerable code fails security tests, patched code passes)

### Quality Control

Automated QC pipeline validates every instance before inclusion:
- Schema validation (column types, nested struct integrity)
- Data structure validation (ID format, patch format, non-empty test lists, CWE format)
- Cross-row consistency (uniqueness, distribution checks)


---


## Links

| Resource | URL |
|----------|-----|
| Dashboard | https://projects.ethara.ai/valkyrie |
| HF Dataset | https://huggingface.co/datasets/ethara/Valkyrie |
| SWE-smith paper | https://arxiv.org/abs/2504.21798 |

---

## Technical Tags

`reinforcement-learning` `security` `software-engineering` `agents` `code` `vulnerability-repair`

---

*Project Valkyrie | Ethara AI*
