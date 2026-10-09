\# Board



Status: `todo` · `doing` · `review` · `done` · `blocked`

Current milestone: \*\*M1\*\*. Only the current milestone is planned in detail. The Leader expands the next one when this one is accepted.



\## M1 — Structured project data



| ID | Owner | Task | Refs | Depends | Status |

|---|---|---|---|---|---|

| T-001 | user | Decide the open decisions in `project-map/product-brief.md` (at least #1, #3, #8) and fill in the Repos table | brief | – | todo |

| T-002 | tester | Define the demo scenario and the ground-truth set of truly affected shots, before any implementation → `contracts/test-scenario.md` | PRD §14, §16 | – | todo |

| T-003 | designer | UI spec for Project Overview and Shot Board (fields per PRD §9.2, status not by color alone) → `contracts/ui-shot-board.md` | PRD §9.1–9.2, FR-01, FR-04, NFR-07 | – | todo |

| T-004 | backend-ai | Data model contract, simplified from PRD §11, with stable IDs and versioning → `contracts/data-model.md` | PRD §11 | T-001 | todo |

| T-005 | backend-ai | API contract for Project and Shot CRUD → `contracts/api-projects-shots.md` | FR-01, FR-04 | T-004 | todo |

| T-006 | backend-ai | Implement Project and Shot CRUD, seed a 6-shot sample project | FR-01, FR-04 | T-005 | todo |

| T-007 | frontend | Implement Shot Board against the UI spec and API contract | FR-04, FR-11 | T-003, T-005 | todo |

| T-008 | tester | Verify M1 against contracts and report | contracts | T-006, T-007 | todo |



\## M2 — Brief understanding



| ID | Owner | Task | Status |

|---|---|---|---|

| – | – | To be planned by the Leader after M1 is accepted (FR-02, FR-03) | todo |



\## M3 — Versioned plan



| ID | Owner | Task | Status |

|---|---|---|---|

| – | – | To be planned after M2 is accepted (FR-05, FR-06, FR-10) | todo |



\## M4 — Change analysis



| ID | Owner | Task | Status |

|---|---|---|---|

| – | – | To be planned after M3 is accepted (FR-07, FR-08) | todo |



\## M5 — End-to-end demo



| ID | Owner | Task | Status |

|---|---|---|---|

| – | – | To be planned after M4 is accepted (FR-09, FR-11, PRD §16) | todo |

