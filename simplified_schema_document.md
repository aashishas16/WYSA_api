# Simplified Wysa Sleep App Database Schema & API Design

Based on the actual app flow, here's a simplified database schema and API design that focuses on the specific interactions needed for the sleep assessment.

## Database Schema

### SQL Schema

```sql
-- Users table to store basic user information
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    nickname VARCHAR(50) UNIQUE NOT NULL,
    terms_accepted BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Sleep assessments table to store user responses
CREATE TABLE sleep_assessments (
    assessment_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(user_id),
    sleep_goal VARCHAR(50), -- "FALL_ASLEEP_EASILY", "SLEEP_THROUGH_NIGHT", "WAKE_REFRESHED"
    sleep_issue_duration VARCHAR(50), -- "LESS_THAN_2_WEEKS", "2_TO_8_WEEKS", "MORE_THAN_8_WEEKS"
    bedtime TIME,
    wake_time TIME,
    sleep_hours FLOAT,
    sleep_efficiency INTEGER, -- calculated percentage
    completed BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Assessment progress tracking
CREATE TABLE assessment_progress (
    progress_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(user_id),
    assessment_id INTEGER REFERENCES sleep_assessments(assessment_id),
    current_step INTEGER DEFAULT 1,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### NoSQL Schema (MongoDB)

#### Users Collection
```json
{
  "_id": "ObjectId",
  "nickname": "string",
  "terms_accepted": "boolean",
  "created_at": "Date"
}
```

#### Sleep Assessments Collection
```json
{
  "_id": "ObjectId",
  "user_id": "ObjectId",
  "sleep_goal": "string", // "FALL_ASLEEP_EASILY", "SLEEP_THROUGH_NIGHT", "WAKE_REFRESHED"
  "sleep_issue_duration": "string", // "LESS_THAN_2_WEEKS", "2_TO_8_WEEKS", "MORE_THAN_8_WEEKS"
  "bedtime": "string", // "HH:MM" format
  "wake_time": "string", // "HH:MM" format
  "sleep_hours": "number",
  "sleep_efficiency": "number", // calculated percentage
  "completed": "boolean",
  "created_at": "Date"
}
```

#### Assessment Progress Collection
```json
{
  "_id": "ObjectId",
  "user_id": "ObjectId",
  "assessment_id": "ObjectId",
  "current_step": "number",
  "updated_at": "Date"
}
```

## API Design

### 1. Get Initial App Information

```
GET /api/app-info
Response:
{
  "status": "success",
  "data": {
    "app_name": "Wysa Sleep",
    "welcome_message": "Welcome to Wysa Sleep, your AI sleep coach",
    "terms_url": "https://wysa.io/terms",
    "privacy_url": "https://wysa.io/privacy"
  }
}
```

### 2. Create User / Submit Nickname

```
POST /api/users
Request:
{
  "nickname": "string"
}
Response:
{
  "status": "success",
  "data": {
    "user_id": "string",
    "nickname": "string",
    "next_step": "terms_and_conditions"
  }
}
```

### 3. Accept Terms and Conditions

```
PUT /api/users/{user_id}/accept-terms
Response:
{
  "status": "success",
  "data": {
    "user_id": "string",
    "terms_accepted": true,
    "next_step": "intro_info"
  }
}
```

### 4. Start Sleep Assessment

```
POST /api/users/{user_id}/assessments
Response:
{
  "status": "success",
  "data": {
    "assessment_id": "string",
    "current_step": "sleep_goal",
    "message": "Let's begin your sleep assessment"
  }
}
```

### 5. Update Sleep Goal

```
PUT /api/assessments/{assessment_id}/sleep-goal
Request:
{
  "sleep_goal": "string" // "FALL_ASLEEP_EASILY", "SLEEP_THROUGH_NIGHT", "WAKE_REFRESHED"
}
Response:
{
  "status": "success",
  "data": {
    "assessment_id": "string",
    "sleep_goal": "string",
    "next_step": "sleep_issue_duration"
  }
}
```

### 6. Update Sleep Issue Duration

```
PUT /api/assessments/{assessment_id}/issue-duration
Request:
{
  "sleep_issue_duration": "string" // "LESS_THAN_2_WEEKS", "2_TO_8_WEEKS", "MORE_THAN_8_WEEKS"
}
Response:
{
  "status": "success",
  "data": {
    "assessment_id": "string",
    "sleep_issue_duration": "string",
    "next_step": "bedtime"
  }
}
```

### 7. Update Bedtime

```
PUT /api/assessments/{assessment_id}/bedtime
Request:
{
  "bedtime": "string" // "HH:MM" format
}
Response:
{
  "status": "success",
  "data": {
    "assessment_id": "string",
    "bedtime": "string",
    "next_step": "wake_time"
  }
}
```

### 8. Update Wake Time

```
PUT /api/assessments/{assessment_id}/wake-time
Request:
{
  "wake_time": "string" // "HH:MM" format
}
Response:
{
  "status": "success",
  "data": {
    "assessment_id": "string",
    "wake_time": "string",
    "next_step": "sleep_hours"
  }
}
```

### 9. Update Sleep Hours

```
PUT /api/assessments/{assessment_id}/sleep-hours
Request:
{
  "sleep_hours": "number"
}
Response:
{
  "status": "success",
  "data": {
    "assessment_id": "string",
    "sleep_hours": "number",
    "next_step": "results"
  }
}
```

### 10. Complete Assessment & Get Results

```
PUT /api/assessments/{assessment_id}/complete
Response:
{
  "status": "success",
  "data": {
    "assessment_id": "string",
    "completed": true,
    "sleep_efficiency": "number", // calculated percentage
    "message": "Based on your responses, your sleep efficiency is 75%",
    "recommendations": [
      "Try to maintain a consistent sleep schedule",
      "Avoid screens before bedtime",
      "Create a relaxing bedtime routine"
    ]
  }
}
```

## API Flow Sequence

1. User opens the app → `GET /api/app-info`
2. User enters nickname → `POST /api/users`
3. User accepts T&C → `PUT /api/users/{user_id}/accept-terms`
4. User starts assessment → `POST /api/users/{user_id}/assessments`
5. User selects sleep goal → `PUT /api/assessments/{assessment_id}/sleep-goal`
6. User selects issue duration → `PUT /api/assessments/{assessment_id}/issue-duration`
7. User selects bedtime → `PUT /api/assessments/{assessment_id}/bedtime`
8. User selects wake time → `PUT /api/assessments/{assessment_id}/wake-time`
9. User selects sleep hours → `PUT /api/assessments/{assessment_id}/sleep-hours`
10. Assessment completed → `PUT /api/assessments/{assessment_id}/complete`

## Sleep Efficiency Calculation

The sleep efficiency is calculated using the formula:
```
sleep_efficiency = (sleep_hours / time_in_bed) * 100
```
Where:
- `sleep_hours` is the self-reported hours of sleep
- `time_in_bed` is calculated as the difference between wake time and bedtime

For example, if a user reports:
- Bedtime: 23:00
- Wake time: 07:00
- Sleep hours: 6.5

The time in bed is 8 hours, and the sleep efficiency would be:
```
sleep_efficiency = (6.5 / 8) * 100 = 81.25%
```
