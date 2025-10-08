# Usecase: "Transfer Priya Nair to Senior Data Engineer in Analytics on Nov 1 and open a backfill."
The Copilot validates details against Workday, creates the Job Change, surfaces and actions the Workday inbox approval in chat, then opens an HR Case to track the backfill and shares policy answers inline from Workday Knowledge. This hits three Workday connectors plus an HTTP call for any gap endpoints.

Video: https://www.youtube.com/watch?v=tIBP6rMB2G4
## Flow Diagram

```mermaid
flowchart TD
    A[Manager Request: Transfer Employee + Backfill] --> B{Parse & Validate Request}
    B --> C[Query Workday Knowledge for Policy]
    C --> D[Validate Employee & Target Role]
    D --> E[Create Job Change via HTTP]
    E --> F[Job Change Created Successfully]
    F --> G[Surface Approval Card in Chat]
    G --> H{Manager Approval Action}
    
    H -->|Approve| I[Call Workday Approval Endpoint]
    H -->|Deny| J[Update Status: Denied]
    
    I --> K[Job Change Approved]
    K --> L[Open HR Case for Backfill]
    L --> M[Post Case Link in Chat]
    M --> N[Process Complete]
    
    J --> O[Transfer Cancelled]
    
    %% Error Handling
    B -->|Invalid Request| P[Ask for Missing Details]
    D -->|Validation Failed| Q[Show Error & Policy Info]
    E -->|API Error| R[Retry/Manual Fallback]
    
    %% Knowledge Integration
    C --> S[Policy Questions Answered Inline]
    Q --> S
    
    %% Styling
    classDef userAction fill:#e1f5fe
    classDef workdayAPI fill:#f3e5f5
    classDef approval fill:#fff3e0
    classDef success fill:#e8f5e8
    classDef error fill:#ffebee
    
    class A,H userAction
    class C,D,E,I,L workdayAPI
    class G,K approval
    class N success
    class J,O,P,Q,R error
```

## Connectors you will usease - Internal Transfer + Backfill Copilot in chat

A manager types:
“Transfer Priya Nair to Senior Data Engineer in Analytics on Nov 1 and open a backfill.”
The Copilot validates details against Workday, creates the Job Change, surfaces and actions the Workday inbox approval in chat, then opens an HR Case to track the backfill and shares policy answers inline from Workday Knowledge. This hits three Workday connectors plus an HTTP call for any gap endpoints.

## Connectors you will use

* **Workday Knowledge** - enterprise search answers for transfer policy, eligibility rules, and forms in chat. ([Moveworks][1])
* **Workday Approvals** - show the Workday inbox approval in chat and let approvers approve or deny directly. ([Moveworks][2])
* **Workday HR Cases** - open and track a backfill or comp-change case when the transfer is approved. ([Moveworks][3])
* **HTTP connector to Workday** - call specific REST or RaaS endpoints to create the Job Change and, if allowed in your tenant, create a Job Requisition. ([Moveworks][4])
* Reference - Moveworks’ Workday integration overview for automation scope across HR and finance. ([Moveworks][5])

## Connection order - fastest path to a demo

1. **Workday Knowledge** - immediate value and a quick health check that search works and auth is correct. ([Moveworks][1])
2. **Workday Approvals** - unlocks the in-chat approval moment that sells the story. Ensure the ISU security group and business process steps are configured for approvals. ([Moveworks][6])
3. **HTTP connector to Workday** - implement Create Job Change and fetch job profiles for validation. If requisition creation is permitted in your tenant, add that call. ([Moveworks][4])
4. **Workday HR Cases** - open a “Backfill requisition tracking” or “Comp review” case post-approval so you can show lifecycle status. ([Moveworks][3])

## What the MVP does in chat

* Confirms worker, target org and job profile, effective date, and whether to open a backfill.
* Calls Workday to create the **Job Change** and returns the business process and inbox task id.
* Surfaces the **Approval** card in chat and handles Approve or Deny. ([Moveworks][2])
* If approved, opens a **Workday HR Case** to track the backfill and posts the case link. ([Moveworks][3])
* Answers “What is our internal transfer policy” or “Which orgs can I transfer into” by querying **Workday Knowledge**. ([Moveworks][1])

---

## 6-hour build plan

**Prereqs before 10:15 am**

* ISU credentials and OAuth client ready for Workday. Approvals require a User-Based Security Group added to the approval step. ([Moveworks][6])
* One test worker, one target org, two job profiles, and a simple approval flow.

**10:15 - 11:00 - Scaffold the conversation**

* Intent: “transfer X to role Y on date Z and open a backfill.”
* Short form for any missing fields.
* Load a small static map of {label -> Workday id} to avoid building selectors.

**11:00 - 12:00 - Wire Workday Knowledge**

* Connect Workday Knowledge and verify Q&A in chat for transfer policy and benefits. This proves your bot is live and useful while you wire writes. ([Moveworks][1])

**12:00 - 1:00 - Add Approvals**

* Connect Workday Approvals and confirm your test approver can see and act on inbox items in chat. Configure approval mirroring settings. ([Moveworks][2])

**1:00 - 2:30 - Job Change via HTTP**

* HTTP connector call: Create Job Change for the worker with reason, effectiveDate, new org and job profile. Store businessProcessId and inboxTaskId. ([Moveworks][4])
* Post a status card with Approve and Deny buttons tied to the inboxTaskId.

**2:30 - 3:15 - Approval callback**

* On Approve, call the Workday approval endpoint and update the status card to Approved. ([Moveworks][2])

**3:15 - 4:15 - HR Case for backfill tracking**

* Connect Workday HR Cases. Create a case with the moved worker’s previous position details and link it back in chat. ([Moveworks][3])

**4:15 - 5:30 - Resilience and polish**

* Add retries, error messages, and a “Refresh status” button that re-hydrates from stored state.
* Validate Knowledge answers for top 3 policy questions.

**5:30 - 6:00 - Dry run and screenshots**

* Show the full thread: request -> confirmation -> Job Change created -> Approve in chat -> HR Case opened -> policy answers.

---

## Minimal payloads you can adapt

* **Create Job Change** - via HTTP connector to your Workday REST endpoint.
  `workerId, jobChangeReasonId, effectiveDate, supervisoryOrgId, jobProfileId, locationId, comments` - keep fields minimal for the demo. ([Moveworks][4])
* **Approve inbox task** - call the Workday approval action with `inboxTaskId` and `action=APPROVE`. ([Moveworks][2])
* **Open HR Case** - submit a case with summary, workerId, prior position, and target org so HR can post the requisition and coordinate comp. ([Moveworks][3])

---

## Why this is hackathon-worthy

* It delivers a high-impact, end-to-end HR change in one chat thread using multiple Workday connectors that Moveworks supports today. ([Moveworks][5])
* The in-chat approval is a clear wow moment for judges and requires limited UI work. ([Moveworks][2])
* The Knowledge connector ensures useful answers even if a write call fails, so your demo is resilient. ([Moveworks][1])

If you want, I can generate the exact HTTP connector request templates and a compact checklist you can paste into your runbook so your team can execute step by step during the build window.

[1]: https://help.moveworks.com/docs/content-integration-workday?utm_source=chatgpt.com "Workday"
[2]: https://help.moveworks.com/docs/enterprise-approvals?utm_source=chatgpt.com "Approvals Queue - Moveworks"
[3]: https://help.moveworks.com/docs/workday-access-requirements-cases?utm_source=chatgpt.com "Workday Access Requirements - HR Cases - help.moveworks.com"
[4]: https://help.moveworks.com/docs/http-connectors?utm_source=chatgpt.com "HTTP Connectors - help.moveworks.com"
[5]: https://www.moveworks.com/us/en/platform/integrations/workday?utm_source=chatgpt.com "Workday Integrations: Automate HR Processes with Moveworks AI"
[6]: https://help.moveworks.com/docs/workday-access-requirements?utm_source=chatgpt.com "Workday Access Requirements - Approvals - Moveworks"
