# Risk Register

| ID | Risk / uncertainty | Likelihood / impact | Mitigation/evidence | Owner | State |
|---|---|---|---|---|---|
| R-01 | Product vision is defined, but MVP boundary, outcomes and measures are not approved; architecture may optimize for the wrong first release | High / High | CINE-001 approval before implementation backlog | Product | Open |
| R-02 | Metadata source terms may not permit intended use | Medium / High | CINE-003 source/terms verification | Research / Legal | Open |
| R-03 | Rights for individual titles may be absent, limited or revoked, preventing lawful playback | Unknown / Critical | Decide delivery model and policy; fail closed for unverified rights; maintain an authorized playable catalog | Product / Legal | Open |
| R-04 | Regions/users may trigger unmodeled legal/privacy/age requirements | Unknown / High | CINE-002 and CINE-005 review | Product / Legal | Open |
| R-05 | Committed identity, profiles and analytics capabilities have unresolved provider, privacy, retention and integration details that may alter architecture | Unknown / High | Define identity/provider and analytics data constraints in CINE-005; payments are outside current core vision | Product / Research | Open |
| R-06 | Scale, SLO, budget and operations capacity unknown | High / High | Quantify/defer through CINE-004 | Product / DevOps | Open |
| R-07 | No accepted architecture artifacts/ADRs | High / High | CINE-006 after verified inputs | Architect | Open |
| R-08 | No Git repository metadata/remote or GitHub issue access; application code, tests, dependency manifests/lockfiles and CI are still absent | High / Medium | Establish repo/remote and issue access; approve implementation baseline; then add manifests, application, tests and CI with evidence | Director / Planner / DevOps | Open |

Evidence is insufficient for more precise scoring. Update with sources, owner and date rather than intuition.
