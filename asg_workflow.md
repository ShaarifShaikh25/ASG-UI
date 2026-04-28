# ASG Workflows — Implementation using ASG / AAL Data Structure

This document maps the provided data structure (`asg_aal_datastructure.md`) to actionable workflows, decision rules, and example pseudocode for background workers and triggers. It focuses on Live Inputs → AI processing, AI match creation, Event registration, Ask lifecycle, News/Wall-of-Fame publishing, and repository approval flows.

---

## 1) Overview / Goals
- Use existing normalized tables (`Live_Inputs`, `Users`, `AI_Resource_Matches`, `Events`, `Event_Registrations`, `Ecosystem_Board`, `User_Repositories`) as source of truth.
- Keep heavy processing off the main Node.js thread — push to background workers.
- Make decision rules explicit (thresholds, approval steps) and auditable.

## 2) Core workflow: Live Input → Background Worker → AI → Matches

Flow:
1. Web/API: user submits an input (text/voice/video/image).
   - Insert row into `Live_Inputs` with `ai_processed = FALSE`.
2. Background worker (BullMQ / SQS worker): picks `Live_Inputs` where `ai_processed = FALSE`.
3. Worker fetches associated user, relevant context (recent posts, `Ecosystem_Board`, `User_Repositories`) and calls AI pipelines to:
   - Extract structured attributes (skills, ask_type, project_stage).
   - Produce a `digital_persona` delta.
   - Score potential matches to other users/resources.
4. Worker writes results:
   - Update `Users.digital_persona` (append/merge JSON summary).
   - Insert rows in `AI_Resource_Matches` with `status='SUGGESTED'` and `match_reason` + `score` (store score in JSON or add score column).
   - Set `Live_Inputs.ai_processed = TRUE`.
5. Notification: publish push/WS event to matched `target_user_id` and `source_user_id`.

Decision rules (examples):
- If match `score >= 0.75` → create `AI_Resource_Matches` and send push notification labeled `suggested_high`.
- If 0.5 <= score < 0.75 → create match with `priority='medium'` and surface in weekly digest only.
- If `ai_authenticity_score < 3` for submissions → mark for manual review in `Event_Submissions_AntiCheat`.

SQL examples:
- Enqueue read (worker):
```sql
SELECT * FROM Live_Inputs WHERE ai_processed = FALSE ORDER BY input_id LIMIT 20;
```
- Insert match:
```sql
INSERT INTO AI_Resource_Matches (source_user_id,target_user_id,match_reason,status,created_at)
VALUES (?, ?, ?, 'SUGGESTED', NOW());
```

## 3) Event Registration / Check-in workflow
- UI: event page reads `Events` and renders dynamic form (schema in `Event_Registrations.form_data` as JSON).
- On register: create `Event_Registrations` row with `status='REGISTERED'` and `form_data` JSON.
- If seats full: insert with `status='WAITLIST'` and notify user if promoted.
- On QR check-in: worker updates `qr_scanned_at` for registration and marks `status='ATTENDED'`.

Decision rules:
- Auto-waitlist when registration count >= event capacity.
- Auto-promote from waitlist on cancellation; notify via FCM/WebSocket.

## 4) Ask lifecycle (Ecosystem Board `post_type='ASK'`)
- Create ask → insert into `Ecosystem_Board` with `post_type='ASK'`.
- Users can `connect` (UI action) which creates a lightweight `Connections` record (or notification entry).
- AI agent periodically scans `ASK` posts and suggests matches by writing `AI_Resource_Matches` for mentors/experts.

Decision rules:
- If ask contains keywords `mentor|co-founder|designer`, tag and surface to targeted repo groups (R2–R4) using `User_Repositories`.
- If ask receives > N connects in 48 hours → escalate to `featured` tag (update `Ecosystem_Board`).

## 5) News & Wall of Fame publishing pipeline
- Posting: insert into `Ecosystem_Board` with `post_type='NEWS'` or `WALL_OF_FAME`.
- Display: feed queries read from `Ecosystem_Board` ordered by `created_at` with pagination.
- Moderation: posts by non-R5 authors go into a moderation queue (a `status` column can be added: `DRAFT|PUBLISHED|PENDING_REVIEW`).

Decision rules:
- If `post_type='WALL_OF_FAME'` and author has `repo_category = R4` or `R5`, auto-approve; else route to SPOC for approval.

## 6) Repository (R1–R5) approval flow
- When a user requests a repo mapping → create `User_Repositories` row with `approval_status = 'PENDING'`.
- Notify `approved_by` candidates (SPOC / R5). On approve, set `APPROVED` and update `approved_by`.

Decision rules:
- Auto-approve if `user` already has `R4` or `R5` in `User_Repositories` and role is `FACULTY`.
- Else manual approval required by SPOC.

## 7) Notifications & Webhooks
- Triggers: new `AI_Resource_Matches`, `Event_Registrations` status changes, `Ecosystem_Board` new important posts.
- Implementation: worker publishes events to Redis / WebSocket / FCM. Also enqueue email jobs for critical flows.

## 8) Worker pseudocode (Node.js style)
```js
// worker.js (pseudo)
async function processBatch() {
  const inputs = await db.query("SELECT * FROM Live_Inputs WHERE ai_processed=FALSE LIMIT 20");
  for (const inpt of inputs) {
    try {
      const user = await db.getUser(inpt.user_id);
      const context = await gatherContext(user.user_id);
      const aiResult = await aiClient.analyzeInput(inpt.content_url, {context});

      await db.update('Users', {digital_persona: aiResult.personaDelta}, {user_id: user.user_id});

      for (const candidate of aiResult.matches) {
        if (candidate.score >= 0.5) {
          await db.insert('AI_Resource_Matches', {
            source_user_id: user.user_id,
            target_user_id: candidate.userId,
            match_reason: candidate.reason,
            status: 'SUGGESTED'
          });
          if (candidate.score >= 0.75) notifyHighPriority(candidate.userId, user.user_id);
        }
      }
      await db.update('Live_Inputs', {ai_processed: true}, {input_id: inpt.input_id});
    } catch (err) {
      // handle & log
    }
  }
}
```

## 9) Example queries for feed & search
- Paginated Ecosystem feed:
```sql
SELECT * FROM Ecosystem_Board
WHERE post_type IN ('NEWS','ANNOUNCEMENT')
ORDER BY created_at DESC
LIMIT 20 OFFSET 0;
```
- Find suggested matches for a user:
```sql
SELECT * FROM AI_Resource_Matches WHERE target_user_id = ? AND status='SUGGESTED' ORDER BY created_at DESC;
```

## 10) Operational notes / thresholds
- Match thresholds: high >= 0.75, medium >= 0.5, low < 0.5 (do not auto-create for low).
- Worker concurrency: tune to background queue and DB connections; use optimistic locking when updating `ai_processed`.
- Audit: keep `match_reason` and raw AI payload in a separate audit table for traceability.

## 11) Next steps (I can implement)
- Generate Node.js worker stub + sample endpoints for posting `Live_Inputs`.
- Add DB migration SQL (MySQL) that includes any missing columns used above (e.g., `AI_Resource_Matches.score`, `Ecosystem_Board.status`).
- Add unit tests & a small harness to simulate an input → match flow.

---

If you want, I can now:
- Create the worker stub and sample migration files (Node.js + SQL), or
- Generate detailed sequence diagrams or an ERD PDF.

Which should I do next?