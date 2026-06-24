# HP Demo Runbook — MUnit + CI/CD Quality Gate

**Goal of the demo:** show that the pipeline *automatically blocks* code that
doesn't have enough test coverage, and that legacy apps can be backfilled with
AI-generated MUnit. This is the "automatic gatekeeping, not manual" story HP asked for.

**Project location:** `~/HP-MUnit-Demo/orders-api`
**GitHub repo:** https://github.com/kavishworks/orders-api
**Live app:** `hp-orders-api-demo` on CloudHub 2.0 (space: agent-fabric-demo-network)

---

## BEFORE THE SESSION (do this once, ~10 min before)

1. Open **Anypoint Code Builder (ACB)** on the folder `~/HP-MUnit-Demo/orders-api`
   (open THIS folder as the workspace root, not the parent).
2. Open a **terminal** in that folder:
   `cd ~/HP-MUnit-Demo/orders-api`
3. Open these browser tabs and log in:
   - GitHub Actions: https://github.com/kavishworks/orders-api/actions
   - SonarCloud: https://sonarcloud.io/project/overview?id=kavishworks_orders-api
   - Anypoint Runtime Manager (to show the running app)
4. Do one warm-up build so the live run is fast (downloads cached):
   `mvn clean test`  → should end BUILD SUCCESS, 100% coverage.

---

## THE LIVE DEMO — 3 acts

### ACT 1 — "Here's a quality gate, and it BLOCKS bad code" (local, in ACB)

Talking point: *"A quality gate means the build fails automatically if coverage
is too low. No human has to remember to check."*

1. In ACB, open the **MUnit** panel.
2. Run ONLY the happy-path test:
   - Right-click `src/test/munit/happy-path-suite.xml` → Run, OR in terminal:
     `mvn clean test -Dmunit.test=happy-path-suite.xml`
3. **Point at the output:** coverage = **50%**, BUILD FAILURE:
   `Application coverage is below defined limit. Required: 80.0% - Current: 50.00%`
   Say: *"The developer thinks they're done — one test passes. But the gate
   stops it: only 50% of the app is covered. It cannot be deployed."*

### ACT 2 — "Fix it by adding the missing tests → gate goes green"

Talking point: *"Now we cover the error paths — validation, the downstream
outage, the discount logic. Watch the gate flip."*

1. Run ALL tests:
   - MUnit panel → **Run all**, OR in terminal:
     `mvn clean test`
2. **Point at the output:** coverage = **100%**, BUILD SUCCESS, 7 tests pass.
3. (Optional visual) open the coverage report:
   `open target/site/munit/coverage/summary.html`
   Show green vs red lines.

### ACT 2.5 (optional but high-impact) — "Backfill legacy with AI"

Talking point: *"You have 900 APIs, many with no tests. You don't hand-write
those — you generate them."*
1. In ACB, open a flow with no test, use **Code Builder AI → generate MUnit test**.
2. Show the generated test, run it, watch coverage rise.
   (Rehearse this once beforehand — AI output varies.)

### ACT 3 — "The same gate runs automatically in the pipeline" (GitHub)

Talking point: *"Everything you just saw runs automatically on every push —
no one runs it by hand."*

1. Make a tiny visible change (e.g. edit a comment in `orders.xml`), then in terminal:
   ```
   git add -A
   git commit -m "demo: trigger pipeline"
   git push https://kavishworks:<TOKEN>@github.com/kavishworks/orders-api.git main
   ```
   (Keep your push token handy — see note below.)
2. Switch to the **GitHub Actions** tab, open the running workflow.
3. Walk the 4 stages as they go green:
   - **Build, Test & Scan** — runs MUnit + the 80% gate + SonarCloud
   - **Publish to Exchange** — versioned asset published
   - **Deploy to CloudHub** — app deployed to CloudHub 2.0
4. Open **SonarCloud** tab — show the code-quality dashboard + coverage.
5. Open **Runtime Manager** — show `hp-orders-api-demo` running.

Optional kicker: *"And if we wanted, we'd require a human approval before the
prod deploy."* (That's the `production` environment gate in the workflow.)

---

## THE "MONEY SHOT" if you only have 2 minutes
Run `mvn clean test -Dmunit.test=happy-path-suite.xml` → show 50% / FAIL.
Run `mvn clean test` → show 100% / PASS.
That single contrast IS the whole message.

---

## NOTES / GOTCHAS
- A full pipeline run takes ~9-10 min (cold). Push EARLY in the session so it's
  finishing while you talk through Acts 1-2.
- The push token is in your password manager / chat history. If it expired,
  generate a new fine-grained PAT (scopes: Contents + Workflows: read/write) at
  github.com/settings/tokens.
- To re-show the RED state after a green run: re-run only the happy-path suite
  (command in Act 1). The gate config lives in `pom.xml` (failBuild=true, 80%).
- Each pipeline run auto-bumps the version (1.0.<run-number>) so Exchange never
  rejects a re-publish — you can run the demo as many times as you like.
- If CloudHub deploy is slow/over quota during the session, it's fine to stop at
  "Build+Test+Sonar+Exchange green" — the gate story is already made.
