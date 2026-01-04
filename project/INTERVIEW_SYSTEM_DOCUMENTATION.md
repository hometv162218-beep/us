# Interview System Documentation

## Overview

The AI Interview Assistant is an intelligent mock interview platform that helps students and candidates practice for job interviews. It uses artificial intelligence to generate personalized interview questions, record and transcribe responses, and provide detailed feedback on performance. The system supports both technical and HR/behavioral interviews.

## Complete User Journey

### 1. User Registration & Authentication

**What happens:**
- User creates a new account with personal details
- System hashes password using bcrypt for security
- User profile is stored in the Supabase database
- JWT token is generated for session management

**Data stored:**
- Full name
- Email (unique identifier)
- Hashed password
- Role (student, mentor, admin)
- Roll number (optional)
- Department (optional)
- Batch year (optional)

**Security:**
- Passwords are hashed and cannot be reversed
- JWT tokens expire after 24 hours
- Each API request requires valid authentication token

---

### 2. Dashboard

**What the student sees:**
- Personal profile with name and email
- Statistics dashboard showing:
  - Total interviews taken
  - Completed interviews
  - Total practice time in minutes
- List of all past interview sessions
- Button to start a new interview

**Functionality:**
- View all interview history
- Click on any session to see detailed results
- Delete old sessions
- Track progress over time
- One-click logout

---

### 3. Starting a New Interview

**Interview Setup Page allows student to:**

1. **Select Interview Type**
   - Technical: Focus on coding, system design, technical concepts
   - HR: Focus on behavioral, soft skills, cultural fit

2. **Enter Job Description**
   - Paste the full job description or role requirements
   - Used to generate relevant questions
   - Helps tailor feedback to the specific role

3. **Upload Resume**
   - Upload PDF resume file
   - System extracts text from PDF
   - Used for question generation and context

4. **Select Duration**
   - Choose interview length (typically 10-30 minutes)
   - Determines number of questions to generate
   - Formula: 1 question per ~1.5 minutes

5. **Start Interview Button**
   - Triggers question generation
   - System generates questions based on duration selected

---

## Core Interview Functionalities

### Question Generation

**How it works:**

1. **Duration-based Calculation**
   - Each question allocated ~90 seconds
   - Example: 15-minute interview = 10 questions
   - Minimum 1 question, no maximum

2. **AI Question Generation (OpenAI GPT-4)**

   **For Technical Interviews:**
   - Focuses on coding, algorithms, system design
   - Asks about specific technologies from resume/JD
   - Tests problem-solving approaches
   - Questions about architecture and best practices

   **For HR Interviews:**
   - Behavioral questions (Tell me about a time when...)
   - Soft skills and teamwork
   - Leadership and conflict resolution
   - Career goals and motivation
   - Cultural fit assessment

3. **Question Relevance**
   - Uses job description to ensure relevance
   - Considers candidate's resume background
   - Adapts difficulty based on experience level
   - Each question has estimated time (90 seconds)

**Example Technical Question:**
```
"Describe your approach to designing a scalable microservices architecture
for a real-time data processing application. What trade-offs would you consider?"
```

**Example HR Question:**
```
"Tell me about a time when you had to work with a difficult team member.
How did you handle the situation and what did you learn?"
```

---

### Interview Recording & Transcription

**During the Interview:**

1. **Recording Process**
   - Student sees one question at a time
   - Timer counts down (90 seconds per question)
   - Audio is recorded via browser microphone
   - Student can:
     - Record answer
     - Re-record if unsatisfied
     - Skip to next question
     - Pause interview

2. **Audio Storage**
   - Recordings saved as audio files
   - Stored on server for later analysis
   - Associated with specific question ID

3. **Transcription (OpenAI Whisper API)**
   - Audio automatically converted to text
   - Uses latest Whisper model for accuracy
   - Handles various accents and speaking speeds
   - Captures every word spoken
   - Process:
     - Upload audio file
     - Whisper API processes audio
     - Returns text transcript
     - Transcript stored with answer

**Transcription preserves:**
- Exact words spoken
- Filler words (ums, ahs) for assessment
- Speaking pace indicators
- Natural language patterns

---

### Answer Evaluation & Scoring

**After interview completes:**

#### Step 1: Generate Reference Answer
- AI creates ideal answer for each question
- Uses job description and resume context
- Tailored to interview type (technical/HR)
- Serves as comparison baseline

#### Step 2: Evaluate Student's Answer

The system scores each answer on **5 criteria** (1-10 scale):

**For Technical Interviews:**
1. **Relevance** - Does answer address the question?
2. **Accuracy** - Are technical facts correct and error-free?
3. **Depth** - Shows technical reasoning and trade-off understanding?
4. **Clarity** - Is explanation well-structured and easy to follow?
5. **Fit** - Overall technical competency for the role?

**For HR Interviews:**
1. **Relevance** - Does answer address the behavioral question?
2. **Accuracy** - Are examples authentic and believable?
3. **Depth** - Shows self-awareness and reflection?
4. **Clarity** - Well-structured using STAR method (if applicable)?
5. **Fit** - Cultural fit and soft skills alignment?

#### Step 3: Generate Feedback

For each answer, AI provides:
- **Strengths** - What the student did well
- **Areas for Improvement** - Specific weaknesses
- **Comparison to Ideal** - How it compares to reference answer
- **Overall Score** - Average of 5 criteria (1-10)

---

### Results & Analytics

**On Results Page, student sees:**

1. **Overall Interview Score**
   - Average of all question scores
   - Visual progress indicator
   - Performance rating (Excellent/Good/Average/Needs Improvement)

2. **Question-by-Question Breakdown**
   ```
   Q1: Tell me about yourself
   - Score: 8/10
   - Your answer: [Full transcript]
   - Ideal answer: [Model answer]
   - Feedback:
     * Strength: Good structure and specific examples
     * Improvement: Could elaborate more on achievements
   ```

3. **Performance Metrics**
   - Score distribution chart
   - Strengths vs weaknesses
   - Category-wise performance (relevance, accuracy, depth, clarity, fit)

4. **Export Options**
   - Download detailed PDF report
   - Print-friendly format
   - Share results (optional)

---

## Session Data Storage

### What Gets Saved

Each interview session creates the following records:

**1. Interview Session Record**
```
- Session ID (unique identifier)
- User ID (linked to student)
- Job Description (what they applied for)
- Resume Text (extracted from PDF)
- Duration (in seconds)
- Interview Type (technical or HR)
- Status (created → in_progress → analyzed)
- Created/Updated timestamps
```

**2. Interview Answer Records (one per question)**
```
- Answer ID (unique)
- Session ID (links to session)
- Question ID (q1, q2, q3, etc.)
- Question Text (the question asked)
- Audio Path (where recording is stored)
- Transcript (text of answer)
- Score (1-10)
- Feedback (array of improvement points)
- Model Answer (ideal answer for comparison)
- Timestamps
```

### Database Structure

```
Users Table
├── id, name, email, password, role
├── roll_number, department, batch
└── created_at, updated_at

Interview Sessions Table
├── id, user_id (foreign key to Users)
├── job_description, resume_text
├── duration_seconds, interview_type
├── status, created_at
└── updated_at

Interview Answers Table
├── id, session_id (foreign key to Sessions)
├── question_id, question_text
├── audio_path, transcript
├── score, feedback (JSON), model_answer
└── created_at, updated_at
```

---

## Technical Architecture

### Frontend Components

**1. LoginPage & RegisterPage**
- User authentication interface
- Form validation
- Error handling
- Secure token storage

**2. DashboardPage**
- User profile display
- Session history listing
- Statistics dashboard
- Session management

**3. SetupPage**
- Interview configuration
- File upload (resume)
- Form validation
- Interview type selection

**4. InterviewPage**
- Question display
- Audio recording interface
- Timer management
- Recording controls (play, delete, re-record)
- Next/Previous navigation

**5. ResultsPage**
- Results display
- Score visualization
- Detailed feedback
- PDF export
- Session comparison

### Backend Services

**1. Authentication Service** (`auth_service.py`)
- User registration with validation
- Password hashing and verification
- JWT token generation and verification
- User profile retrieval

**2. LLM Service** (`llm_service.py`)
- Question generation (OpenAI GPT-4)
- Reference answer generation
- Answer evaluation with detailed scoring
- Interview-type specific prompting

**3. Transcription Service** (`transcription_service.py`)
- Audio file transcription (OpenAI Whisper)
- Format conversion
- Error handling

**4. PDF Service** (`pdf_service.py`)
- Extract text from PDF resumes
- Generate PDF reports of results

**5. Export Service** (`export_service.py`)
- Create downloadable PDF reports
- Format interview results
- Include scores, feedback, transcripts

### API Endpoints

**Authentication**
```
POST   /api/auth/register          - Create new user account
POST   /api/auth/login             - Login and get JWT token
GET    /api/auth/me                - Get current user profile
```

**User Sessions**
```
GET    /api/user/my-sessions              - List all user's sessions
GET    /api/user/my-session/{id}          - Get specific session details
DELETE /api/user/my-session/{id}          - Delete a session
```

**Interview Flow**
```
POST   /api/create-session                - Create new interview
GET    /api/session/{id}                  - Get session questions & answers
POST   /api/upload-answer/{session}/{qid} - Upload answer audio
POST   /api/analyze/{id}                  - Analyze all answers (start evaluation)
GET    /api/export-pdf/{id}               - Download PDF report
```

---

## AI Integration Details

### Question Generation Flow

```
User Input (JD + Resume + Duration + Type)
         ↓
Calculate Question Count (duration ÷ 90 sec)
         ↓
Build System Prompt (with interview type rules)
         ↓
Call OpenAI GPT-4 API
         ↓
Parse JSON Response
         ↓
Return: [{"id": "q1", "text": "...", "estimated_seconds": 90}]
```

### Evaluation Flow

```
Transcript + Reference Answer + Question
         ↓
Build Evaluation Prompt
         ↓
Call OpenAI GPT-4 API
         ↓
Parse JSON Scores
         ↓
Return:
{
  "scores": {
    "relevance": 8,
    "accuracy": 7,
    "depth": 8,
    "clarity": 9,
    "fit": 8
  },
  "total_score": 8.0,
  "feedback": ["...", "..."],
  "comparison_summary": "..."
}
```

---

## Key Features

### For Students

1. **Multiple Interview Types**
   - Technical interviews (coding, system design)
   - HR interviews (behavioral, soft skills)
   - Personalized questions based on role

2. **Realistic Interview Experience**
   - Timed questions (mirrors real interviews)
   - Time pressure handling
   - Professional feedback
   - Speaking under observation

3. **Detailed Performance Feedback**
   - Score for each question
   - Specific strengths and weaknesses
   - Comparison to ideal answers
   - Actionable improvement suggestions

4. **Practice History**
   - All sessions saved and retrievable
   - Track improvement over time
   - Compare performances across sessions
   - Review past answers anytime

5. **PDF Reports**
   - Download detailed results
   - Share with mentors
   - Print for study
   - Archive for records

### For Mentors/Admins

1. **Student Progress Tracking**
   - View student interview history
   - Analyze performance trends
   - Identify strong/weak areas

2. **Content Management**
   - Add custom job descriptions
   - Create interview templates
   - Configure interview difficulty

---

## Security & Privacy

### Data Protection

1. **Password Security**
   - Bcrypt hashing (salt + rounds)
   - Passwords never stored in plain text
   - Cannot be reversed or decrypted

2. **Authentication**
   - JWT tokens for API authentication
   - Tokens expire after 24 hours
   - Fresh login required for security

3. **Row Level Security (RLS)**
   - Users can only access their own data
   - Database-level enforcement
   - No cross-user data leakage

4. **HTTPS Communication**
   - All API calls encrypted
   - Secure data transmission
   - Protected against interception

### Data Retention

- Interview sessions permanently stored
- Audio files kept indefinitely
- User can delete sessions manually
- No automatic deletion of user data

---

## Limitations & Considerations

### Current Version

1. **Audio Quality**
   - Depends on user's microphone
   - Background noise affects transcription
   - Quiet speakers may not be fully captured

2. **AI Evaluation**
   - Based on transcribed text (not tone, confidence)
   - May not catch non-verbal communication
   - Some context-dependent nuances missed

3. **Interview Types**
   - Currently: Technical and HR
   - Future: Industry-specific (finance, design, etc.)

4. **Languages**
   - Primarily English support
   - Multi-language support can be added

---

## Usage Statistics & Tracking

### Available Metrics

**User Dashboard Shows:**
- Total interviews taken
- Interviews completed (analyzed)
- Total practice time
- Average score trend

**Session-Level Analytics:**
- Score breakdown by category
- Time per question
- Transcription availability
- Feedback provided

### Exporting Data

Students can:
- Download PDF reports
- View all session transcripts
- Access feedback anytime
- Share results with mentors

---

## Troubleshooting

### Common Issues

**1. Audio not recording**
- Check microphone permissions
- Test microphone in browser settings
- Try different browser (Chrome recommended)
- Ensure HTTPS connection

**2. Questions not generating**
- Verify OpenAI API key is valid
- Check API quota/credits
- Resume PDF may have extraction issues
- Try shorter job description

**3. Transcription incomplete**
- Background noise too high
- Speaking too fast or too soft
- Try re-recording answer
- Check audio file format

**4. Low evaluation scores**
- Transcription may have errors
- Answer not addressing question fully
- Lack of specific examples
- Review feedback for improvements

**5. Can't access session**
- Session may be deleted
- Wrong user account
- Authentication token expired (re-login)
- Browser cache issues (clear cache)

---

## Future Enhancements

1. **Real-time Feedback**
   - Instant feedback while answering
   - Pause and retry mid-answer

2. **Video Recording**
   - Record video along with audio
   - Assess body language and presentation
   - Eye contact and confidence evaluation

3. **Multi-language Support**
   - Support interviews in multiple languages
   - Multilingual transcription
   - Localized feedback

4. **Interview Difficulty Levels**
   - Junior, Mid, Senior level questions
   - Company-specific interview templates
   - Industry-specific focus areas

5. **Peer Comparison**
   - Benchmark against other candidates
   - Anonymous performance comparison
   - Industry salary data

6. **Mobile App**
   - iOS/Android native apps
   - Better mobile recording
   - Offline functionality

7. **Integration with Job Platforms**
   - Connect with LinkedIn
   - Pull real job descriptions
   - Direct job matching

8. **Mentor Review**
   - Mentors can review answers
   - Leave custom feedback
   - Schedule follow-ups

---

## Conclusion

The Interview System is a comprehensive platform that combines AI, speech-to-text technology, and educational best practices to help candidates prepare for real job interviews. It provides personalized, scalable, and data-driven interview training.

The system stores all user data securely, provides detailed analytics on performance, and uses machine learning to evaluate answers fairly and consistently. Students can practice multiple times, track progress, and get better with each attempt.
