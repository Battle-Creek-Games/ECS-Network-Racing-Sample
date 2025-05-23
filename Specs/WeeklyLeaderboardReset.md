# Weekly Leaderboard Reset – Feature Brief

> **Purpose**
> Keep competition fresh, boost weekly engagement, and provide regular reward cycles by resetting all global leaderboards every Monday at **00:00 UTC**.

---

## 1 · Scope

* Applies to **all leaderboard categories** (e.g., Fastest Trail, Most XP, Longest Jump).
* Targets **top 100** entries per category.
* Archive + reward logic runs **server‑side**; client simply queries current and archived boards.

---

## 2 · Functional Requirements

| ID   | Requirement                                                                                                                          |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------ |
| F‑01 | **Reset schedule** – Clear current leaderboards at 00:00 UTC each Monday. Schedule driven by server cron; no client trust.           |
| F‑02 | **Archiving** – Snapshot pre‑reset rankings into `Leaderboards_Archive` table with week number (ISO‑8601). Retain 12 weeks.          |
| F‑03 | **Reward payout** – After snapshot, award gold based on rank tiers (1–10, 11–50, 51–100). Rewards piped to existing mailbox service. |
| F‑04 | **Client refresh** – Clients receive `LeaderboardReset` event; UI auto‑refreshes without relog.                                      |
| F‑05 | **Countdown banner** – Show time‑to‑reset banner when < 24 h remain.                                                                 |
| F‑06 | **Archived view** – “Past Weeks” dropdown lets players browse last 12 snapshots (read‑only).                                         |

---

## 3 · Non‑Functional Requirements

* **Latency** – Reset + archive must finish < 30 s to avoid long blank state.
* **Scalability** – Handle up to 2 M entries across all categories.
* **Reliability** – Retry archive + payout up to 3× on failure; emit alert on final fail.
* **Security** – Reward amounts calculated server‑side only; no writable client fields.

---

## 4 · Analytics & Telemetry

| Metric                          | Detail                                                 |
| ------------------------------- | ------------------------------------------------------ |
| `leaderboard_reset_duration_ms` | Time taken for full reset pipeline.                    |
| `weekly_leaderboard_players`    | Distinct player IDs appearing in top 100 any category. |
| `reward_payout_failures`        | Count of failed payout attempts per week.              |

---

## 5 · Edge Cases & Constraints

* **Clock drift** – Server authoritative; clients show local countdown derived from server time.
* **Partial visibility** – If reset in progress, client shows “Updating…” placeholder.
* **Rollback** – If archive fails after clearing, restore from pre‑snapshot backup.
* **Offline players** – Reward mailbox persists 30 days; claimable on next login.

---

## 6 · Out of Scope (V1)

* Regional or per‑platform leaderboards.
* Dynamic reward tiers beyond top 100.
* Season‑long leaderboards (handled by separate feature).

---

## 7 · Assumptions

1. Existing leaderboard service exposes clear read/write endpoints.
2. Reward currency values are predefined by Design (see `RewardsConfigSO`).
3. Ops team has access to Cron/Scheduler to adjust reset time if needed.

---

## 8 · Acceptance Criteria (QA)

1. **Schedule** – At Monday 00:00 UTC, leaderboards clear; new entries accepted immediately.
2. **Archive Integrity** – Snapshot row count equals pre‑reset row count; top ranks preserved.
3. **Rewards** – Player in rank tier receives correct gold amount within 60 s.
4. **Banner** – Countdown matches server time ±2 s; disappears post‑reset.
5. **Resilience** – Simulated archive failure triggers rollback and alert.

---

*One‑page spec owner: Product ✕ Backend Engineering.*
