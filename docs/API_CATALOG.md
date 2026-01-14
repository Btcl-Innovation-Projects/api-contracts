# API Catalog

Base URL: `https://ztouch-api.btc.bw:2896`
Auth: Bearer JWT unless noted.
Public proxy (customer SMS web): `https://ztouch-proxy.btc.bw` (no `/v1` or `/v2` prefix).

## Accounts (`/v2/accounts`)
- POST `/login` - user login (no auth)
- POST `/logout` - logout
- POST `/create-user/` - create user
- POST `/change-password/` - change password
- POST `/reset-password/` - reset password
- POST `/forgot-password/` - forgot password (no auth)
- POST `/expo/update-token/` - update Expo push token
- GET `/groups/{group}` - list users in group
- GET `/companies/` - list companies
- POST `/companies/` - create company
- PATCH `/companies/{company_id}/` - update company
- DELETE `/companies/{company_id}/` - delete company
- GET `/teamleaders/technicians/` - list technicians under teamleader
- GET `/teamleaders/technicians/` search: `search=<text>` matches username, first_name, last_name, email, cone_username
- POST `/teamleaders/technicians/assign/` - assign technician
- POST `/teamleaders/technicians/unassign/` - unassign technician
- GET `/teamleaders/reports/technicians/` - technician performance report
- GET `/teamleaders/reports/technicians/summary/` - technician performance summary + trends
- GET `/teamleaders/reports/technicians/summary/` supports `technician` to scope to one technician
- GET `/teamleaders/reports/technicians/lowest/` - lowest performing technician tile
- GET `/teamleaders/reports/external/technicians/` - external teamleader technicians report
- GET `/teamleaders/reports/external/technicians/summary/` - external teamleader technicians summary + trends
- GET `/teamleaders/reports/external/technicians/lowest/` - external teamleader lowest technician summary
- GET `/leadops/reports/teamleaders/` - LeadOps teamleader technicians report
- GET `/leadops/reports/teamleaders/summary/` - LeadOps teamleader technicians summary + trends
- GET `/leadops/reports/overall/summary/` - LeadOps overall technicians summary + trends
  - `leadops/reports/teamleaders/*` requires `supervisor`

Reports summary additions:
- `rating_trends`: monthly team/technician rating trend for last 12 months (only months with data)
- `turnaround_trends`: monthly turnaround trend for last 12 months (only months with data)
- `avg_turnaround_hours`: aggregate turnaround for current scope
- `pending_counts`:
  - `pending_no_date`: open tasks with no appointment
  - `pending_not_confirmed`: open tasks with appointment_confirmed=false
  - `pending_confirmed`: open tasks with appointment_confirmed=true
  - `created_today`: tasks with date_assigned=today
  - `completed_today`: tasks with date_completed=today
  - `closed_today`: same as completed_today (task completion)
All counts use DB Task + Appointment only (no Redis).
- PATCH `/users/{username}/` - update technician profile fields (teamleader/admin)

## Cone (`/v2/cone`)
- GET `/cases/info/` - get case info from C1
- POST `/cases/orders/{pk}/close/` - close service order
- POST `/cases/tasks/{pk}/complete/` - complete task
- POST `/cases/tasks/{pk}/v2/complete/` - complete task v2
- POST `/cases/tasks/{pk}/v3/complete/` - complete task v3 (appointment required)
- POST `/cases/tasks/{pk}/signoff/` - signoff task
- GET `/cases/` - list cases
- GET `/cases/tasks/` - list user task cases (Redis)
- POST `/cases/assign/` - assign cases to user
- GET `/cases/workload/` - workload summary per technician (teamleader/admin)
- PATCH `/cases/{case_number}/update/` - update case fields
- POST `/cases/upload/` - upload locality cases
- GET `/offers/{pk}/get` - get offer info
- GET `/cases/customer/name/` - get customer name
- GET `/cases/token/` - get access token (no auth)
- PATCH `/cases/tasks/{pk}/assign/` - assign task to user
- GET `/cases/tasks/completed/` - list completed tasks
- POST `/cases/tasks/refresh/` - refresh user task cases
- POST `/cases/tasks/test-upload/` - admin test upload of cases to a user
- GET `/buckets/` - list buckets for user
- POST `/buckets/` - create bucket
- POST `/buckets/refresh/` - refresh bucket task cases (C1)
- PATCH `/buckets/{name}/` - rename bucket
- DELETE `/buckets/{name}/delete/` - delete bucket
- POST `/cases/assign-by-supervisor/` - assign cases by supervisor
- GET `/crm/groups/users/` - get CRM users in group
- GET `/report/task-report/` - monthly task report
- POST `/cases/assign-one/` - assign a single case

## Service Quality (`/v2/servicequality`)
- POST `/cases/appointments/sendlink/` - send appointment link
- GET `/cases/appointments/validate-token/` - validate appointment token (no auth)
- GET `/cases/appointments/availability/` - availability slots for a token (no auth)
- POST `/cases/appointments/schedule/` - customer schedules appointment (token)
- POST `/cases/appointments/reschedule/` - customer reschedules appointment (token)
- PATCH `/cases/appointments/location/` - update appointment location (token)
- POST `/cases/appointments/technician/schedule/` - technician schedules
- POST `/cases/appointments/technician/reschedule/` - technician reschedules
- POST `/cases/appointments/technician/depart/` - technician departs
- POST `/cases/appointments/technician/arrive/` - technician arrives
- POST `/cases/appointments/technician/confirm/` - technician confirms/adjusts
- POST `/cases/appointments/technician/cancel/` - technician cancels
- POST `/cases/appointments/task/complete/` - complete task (appointment flow)
- POST `/cases/appointments/status/` - set appointment ongoing
- POST `/cases/appointments/feedback/submit/` - submit feedback (token)
- GET `/cases/appointments/` - list appointments with feedbacks
- GET `/cases/appointments/` filters: `supervisor`, `technician_username`, `date_from`, `date_to`
- Appointments feed includes `technician_username`, `rating`, `comment`, `created_at`, `case_number`, `task_number`
- Feedback filters: `feedback_type=technician|service`, `has_technician_feedback=true|false`, `has_service_feedback=true|false`
  - Precedence: `feedback_type` is applied first; `has_*` filters further narrow the result.

Examples:
1) Technician feedback only (all time)
`/v2/servicequality/cases/appointments/?feedback_type=technician&has_technician_feedback=true&page=1&page_size=10`

2) Service feedback only (all time)
`/v2/servicequality/cases/appointments/?feedback_type=service&has_service_feedback=true&page=1&page_size=10`

3) Technician feedback for a technician + date range
`/v2/servicequality/cases/appointments/?feedback_type=technician&has_technician_feedback=true&technician_username=<technician_username>&date_from=YYYY-MM-DD&date_to=YYYY-MM-DD&page=1&page_size=50`
- GET `/cases/appointments/summary/` - appointment workload summary
- GET `/cases/ratings/` - list ratings
- GET `/reports/summary/` - service quality summary cards + rolling 12-month series
- GET `/reports/lowest-rated/` - lowest-rated listing (technician or service)

Service Quality reports:
- `/v2/servicequality/reports/summary?feedback_type=technician|service&window=rolling_12_months`
  - cards: totals + averages + weekly response counts + WoW trends
  - series: monthly buckets for last 12 months (only months with data)
  - timezone: uses backend TIME_ZONE (Africa/Gaborone). Weeks are Mon 00:00 to Sun 23:59:59.
  - averages rounded to 1 decimal
  - date_from/date_to override window; response window becomes `date_range`
- `/v2/servicequality/reports/lowest-rated?feedback_type=technician|service&window=rolling_12_months&page=&page_size=`
  - optional: `include_preview=true` returns `feedback_preview` (last 3–5)
  - technician: `technician_username`, `technician_full_name`, `avg_rating`, `rating_count`
  - service: `problem_area`, `avg_nps`, `rating_count`
  - series `rating` meaning: technician = avg CSAT, service = avg NPS

Summary fields:
- cards: `total_feedbacks`, `avg_csat`, `avg_ces`, `avg_nps`, `avg_rating`, `rating_count`
- cards.responses: `technicianWeek`, `technicianLastWeek`, `serviceWeek`, `serviceLastWeek`, `sentWeekTotal`, `sentLastWeek`
- cards.trends: `total`, `csat`, `ces`, `nps`, `rating` (WoW % when computable)
- series[]: `period`, `avg_rating`, `csat`, `ces`, `nps`, `rating_count`
- response_rate is omitted unless a reliable sent-denominator exists

Report examples:
- `/v2/servicequality/reports/summary/?feedback_type=technician&window=rolling_12_months`
- `/v2/servicequality/reports/lowest-rated/?feedback_type=technician&window=rolling_12_months&page=1&page_size=20`

## Network Audit (`/v2/networkaudit`)
- GET `/copper-theft/reports/` - copper theft reports
- GET/POST `/copper-theft/incidents/` - list/create incidents
- GET/PATCH/DELETE `/copper-theft/incidents/{pk}/` - incident detail
- GET/POST `/copper-theft/incident/{incident_id}/affected-customers/` - affected customers
- GET/PATCH/DELETE `/copper-theft/affected-customers/{pk}/` - affected customer detail
- DELETE `/copper-theft/affected-customers/delete-all/{incident_id}/` - delete all affected customers
- GET/POST `/copper-theft/incident/{incident_id}/comments/` - incident comments
- GET/PATCH/DELETE `/copper-theft/comments/{pk}/` - comment detail
- GET `/copper-theft/{content_type}/{object_id}/comments/` - filtered comments
- GET `/copper-theft/prices/` - list prices
- GET/PATCH `/copper-theft/prices/{pk}/` - price detail
- POST `/subscriber/upload/` - upload subscriber data
- GET `/localities/` - list localities
- GET `/regions-with-locations/` - regions with locations
- POST `/locations/report-technician-location/` - report technician location
- GET `/locations/technician-locations/latest/` - latest technician locations

## Star (`/v2/star`)
- GET `/olt/ont/link` - check ONT link to OLT
- POST `/service/provisioning` - provision service
- POST `/service/provisioning/change-cpe/` - change customer CPE
- POST `/service/provisioning_v2` - provision service v2
- POST `/provision/{service_id}` - provision service by ID
- POST `/olt/ont/delete/` - delete ONT
- GET `/olt/ont/unprovisioned/` - unprovisioned ONTs
- GET `/olt/vlans/available/{node_name}` - available VLANs
- GET `/olt/vlans/unused/` - unused VLANs
- POST `/olt/vlans/unused/delete/` - delete unused VLANs
- POST `/service/provisioning/{metro}/{node}/{type}` - provision VLANs on metro/node
- GET `/subscribers/search` - search VLANs
- GET `/subscribers/lookup` - subscriber lookup
- GET `/subscribers/health` - subscriber health

## Pagination & Filtering
- Pagination parameters: `page`, `page_size` on `/v2/accounts/groups/{group}`, `/v2/accounts/teamleaders/technicians/`, `/v2/cone/cases/`, `/v2/cone/cases/tasks/`, `/v2/servicequality/cases/appointments/`, `/v2/servicequality/cases/ratings/`
- Sorting: `ordering` on `/v2/accounts/groups/{group}`, `/v2/accounts/teamleaders/technicians/`
- Teamleader technicians filters:
  - `supervisor={username}` (admin only)
  - `unassigned=true`
  - `missing_fields=true` or `missing=cone_username,company,bucket`
- Ratings filters: `supervisor`, `technician`, `date_from`, `date_to`
- Appointments filters: `supervisor`, `technician_username`, `date_from`, `date_to`

## Status Logic Definitions
- `active/open tasks`: `Task.date_completed IS NULL`
- `pending_confirmation`: appointment exists with `appointment_confirmed=false` and `scheduled_date=filter date`
- `date_not_confirmed`: appointment exists with `appointment_confirmed=false` (any date)
- `no_date`: open tasks with no appointment record

## ID Types
- `bucket_id` is a string (Bucket.id is a CharField)
- `company_id` is an integer

## Public Proxy (Customer SMS Flow)
- SMS links point to `https://ztouch-proxy.btc.bw/service/appointments/...`
- API calls from the web flow target the `/v2/servicequality/cases/appointments/*` endpoints above.

## Availability Rules
- Working window: 08:00–17:00 (local server time)
- Slot step: 30 minutes
- Slot duration: per task estimate (default 30 minutes)
- Lead time: 60 minutes (same-day)
- Max days ahead: 30
- Service type detection (token context): from `Case.service_type` if present; falls back to task description keywords (`voice`, `internet/data`), else `other`.

## Error Cases
- Token expired: 401 with message `Token expired. Renew token.`
- Invalid token: 400 with message `Invalid token.`
- Already submitted: 409 with `Feedback has already been submitted` or `Service feedback has already been submitted`
- Slot unavailable: 409 with `Appointment clashes with a confirmed booking for this technician.`

## Token Context Example
Request:
`GET /v2/servicequality/cases/appointments/validate-token/?token=abc123`

Response:
```json
{
  "success": true,
  "message": "Token is valid",
  "token_valid": true,
  "expires_at": "2026-01-30T09:00:00Z",
  "case_number": "1805745",
  "task_number": "3039479",
  "reference_code": "3039479",
  "technician": { "id": 12, "username": "matihan", "display_name": "Matihan K" },
  "service_type": "internet",
  "appointment_state": {
    "scheduled_date": null,
    "scheduled_time": null,
    "estimated_duration_minutes": 30,
    "confirmed": false,
    "status": "None"
  },
  "allow_reschedule": true,
  "allow_location_update": false
}
```

## Feedback Payloads
Technician feedback (token only):
```json
{
  "customer_satisfaction": 4,
  "customer_effort_score": 5,
  "contractor_identified": "Yes",
  "branded_clothing": "Yes",
  "speedtest_conducted": "No",
  "extended": "Resolved quickly"
}
```

Service feedback (token + fbid):
```json
{
  "net_promoter_score": 9,
  "service_rating": 5,
  "service_feedback": "Great service"
}
```

## Availability Example
Request:
`GET /v2/servicequality/cases/appointments/availability/?token=abc123&date=2026-01-30`

Response:
```json
{
  "success": true,
  "date": "2026-01-30",
  "timezone": "Africa/Gaborone",
  "slots": [
    { "start_time": "09:00:00", "end_time": "09:30:00", "is_available": true },
    { "start_time": "09:30:00", "end_time": "10:00:00", "is_available": false }
  ]
}
```
