# Design Document: AI Document Action Platform

## Overview

The AI Document Action Platform is a comprehensive full-stack application that provides intelligent document analysis and automated action recommendations across multiple domains. The system architecture consists of three main layers:

1. **Frontend Layer**: React-based single-page application providing intuitive interfaces for Study Mode, Medical Mode, Finance Mode, and Civic Mode
2. **Backend Layer**: Node.js/Express server handling API requests, business logic, and orchestration
3. **AI Layer**: Local Ollama deepseek-coder:6.7b model for content generation and analysis

The platform emphasizes privacy through local AI processing, user experience through intelligent automation, and accessibility through multi-language support.

## Architecture

### System Components

```
┌─────────────────────────────────────────────────────────────┐
│                     React Frontend (Port 3000)               │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │  Study   │  │ Medical  │  │ Finance  │  │  Civic   │   │
│  │   Mode   │  │   Mode   │  │   Mode   │  │   Mode   │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
│  ┌──────────────────────────────────────────────────────┐  │
│  │         Dashboard & History Management               │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            │
                            │ HTTP/REST API
                            ▼
┌─────────────────────────────────────────────────────────────┐
│              Node.js/Express Server (Port 5000)              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │                   API Routes                          │  │
│  │  /api/study  /api/medical  /api/actions  /api/history│  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │                Service Layer                          │  │
│  │  OCR │ Translation │ Email │ Calendar │ PDF          │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            │
                ┌───────────┴───────────┐
                │                       │
                ▼                       ▼
┌──────────────────────────┐  ┌──────────────────────┐
│  Ollama AI Model         │  │   MongoDB Database   │
│  (localhost:11434)       │  │   (Cloud/Local)      │
│  deepseek-coder:6.7b     │  │                      │
└──────────────────────────┘  └──────────────────────┘
```

### Technology Stack

- **Frontend**: React 18+, vanilla CSS, modern JavaScript (ES6+)
- **Backend**: Node.js, Express.js, Mongoose ODM
- **Database**: MongoDB (with timestamps support)
- **AI Model**: Ollama deepseek-coder:6.7b (local inference)
- **PDF Generation**: PDFKit library
- **OCR**: Tesseract.js or cloud OCR service
- **Translation**: Google Translate API or local translation model
- **Email**: Nodemailer with SMTP
- **Calendar**: Google Calendar API or iCal format

### Communication Patterns

1. **Frontend ↔ Backend**: RESTful API with JSON payloads
2. **Backend ↔ AI Model**: HTTP POST requests to Ollama API
3. **Backend ↔ Database**: Mongoose ODM with async/await
4. **Backend ↔ External Services**: HTTP clients for OCR, translation, email

## Components and Interfaces

### Frontend Components

#### 1. App Component (Root)
```javascript
// Main application component managing routing and global state
interface AppState {
  currentMode: 'welcome' | 'home' | 'study' | 'medical' | 'finance' | 'civic';
  user: UserProfile;
  language: 'en' | 'hi' | 'gu';
}
```

#### 2. Welcome Screen Component
```javascript
interface WelcomeScreenProps {
  onGetStarted: () => void;
}
// Displays logo, tagline, and entry button
```

#### 3. Home Dashboard Component
```javascript
interface HomeDashboardProps {
  user: UserProfile;
  recentActions: Action[];
  onModeSelect: (mode: string) => void;
}
// Displays mode selection cards and recent activity
```

#### 4. Study Mode Component
```javascript
interface StudyModeState {
  topic: string;
  category: string;
  learningLevel: 'Beginner' | 'Intermediate' | 'Placement Prep';
  learningGoal: 'Exam' | 'Job Interview' | 'Project Building';
  notes: string;
  codeSnippet: string;
  language: 'Python' | 'C' | 'JavaScript' | 'Java';
  activeTab: 'summary' | 'notes' | 'roadmap';
  loading: boolean;
}

interface StudyModeActions {
  generateNotes: () => Promise<void>;
  generateCode: () => Promise<void>;
  generateQuiz: () => Promise<void>;
  createFlashcards: () => Promise<void>;
  downloadPDF: () => Promise<void>;
}
```

#### 5. Medical Mode Component
```javascript
interface MedicalModeState {
  uploadedFile: File | null;
  medicalInsight: string;
  selectedLanguage: 'en' | 'hi' | 'gu';
  loading: boolean;
}

interface MedicalModeActions {
  uploadReport: (file: File) => Promise<void>;
  scanWithCamera: () => Promise<void>;
  analyzeReport: () => Promise<void>;
  translateInsight: (language: string) => Promise<void>;
  emailSpecialist: () => Promise<void>;
  bookDoctor: () => Promise<void>;
}
```

#### 6. History Component
```javascript
interface HistoryProps {
  entries: HistoryEntry[];
  searchTerm: string;
  filterCategory: string;
  onEntryClick: (entry: HistoryEntry) => void;
  onSearchChange: (term: string) => void;
  onFilterChange: (category: string) => void;
}
```

#### 7. Dashboard Component
```javascript
interface DashboardData {
  scheduledTasks: Task[];
  studyReadiness: number;
  healthAlerts: number;
  recentActions: Action[];
}
```

### Backend API Endpoints

#### Study Endpoints

```javascript
// POST /api/study
// Generate study notes for a topic
Request: {
  topic: string;
  category?: string;
  learningLevel?: string;
  learningGoal?: string;
}
Response: {
  result: string; // Generated study notes
}

// POST /api/study/code
// Generate code snippet
Request: {
  topic: string;
  language: string;
}
Response: {
  code: string;
}

// POST /api/study/quiz
// Generate quiz questions
Request: {
  topic: string;
  questionCount: number;
}
Response: {
  questions: Array<{
    question: string;
    options: string[];
    correctAnswer: number;
    explanation: string;
  }>;
}

// POST /api/study/flashcards
// Generate flashcard set
Request: {
  topic: string;
  cardCount: number;
}
Response: {
  flashcards: Array<{
    front: string;
    back: string;
  }>;
}

// POST /api/study/pdf
// Generate PDF document
Request: {
  topic: string;
  notes: string;
}
Response: Binary PDF file
```

#### Medical Endpoints

```javascript
// POST /api/medical/upload
// Upload medical report
Request: FormData with file
Response: {
  fileId: string;
  extractedText: string;
}

// POST /api/medical/analyze
// Analyze medical report
Request: {
  fileId: string;
  extractedText: string;
}
Response: {
  insight: string;
  keyIndicators: Array<{
    name: string;
    value: string;
    normal: boolean;
  }>;
}

// POST /api/medical/translate
// Translate medical insight
Request: {
  text: string;
  targetLanguage: string;
}
Response: {
  translatedText: string;
}
```

#### Action Endpoints

```javascript
// POST /api/actions/email
// Send email with document
Request: {
  to: string;
  subject: string;
  body: string;
  attachments?: string[];
}
Response: {
  success: boolean;
  messageId: string;
}

// POST /api/actions/schedule
// Create calendar event
Request: {
  title: string;
  date: string;
  time: string;
  description: string;
}
Response: {
  success: boolean;
  eventId: string;
}
```

#### History and Dashboard Endpoints

```javascript
// GET /api/history
// Retrieve all history entries
Response: Array<{
  _id: string;
  topic: string;
  notes: string;
  category: string;
  learningLevel?: string;
  learningGoal?: string;
  createdAt: string;
  updatedAt: string;
}>

// GET /api/dashboard
// Retrieve dashboard data
Response: {
  scheduledTasks: Task[];
  studyReadiness: number;
  healthAlerts: number;
  recentActions: Action[];
}
```

### Service Layer Components

#### 1. AI Service
```javascript
class AIService {
  async generateStudyNotes(topic, level, goal) {
    // Construct prompt based on learning level and goal
    // Call Ollama API with appropriate parameters
    // Return formatted study notes
  }
  
  async generateCode(topic, language) {
    // Construct code generation prompt
    // Call Ollama API
    // Return code snippet
  }
  
  async generateQuiz(topic, count) {
    // Generate quiz questions using AI
    // Parse and structure response
    // Return question array
  }
  
  async generateFlashcards(topic, count) {
    // Generate flashcard pairs using AI
    // Parse and structure response
    // Return flashcard array
  }
}
```

#### 2. OCR Service
```javascript
class OCRService {
  async extractText(imageBuffer) {
    // Process image with Tesseract.js or cloud OCR
    // Return extracted text
  }
  
  async extractFromPDF(pdfBuffer) {
    // Extract text from PDF document
    // Return extracted text
  }
}
```

#### 3. Translation Service
```javascript
class TranslationService {
  async translate(text, targetLanguage) {
    // Call translation API or use local model
    // Return translated text
  }
  
  supportedLanguages = ['en', 'hi', 'gu'];
}
```

#### 4. Email Service
```javascript
class EmailService {
  async sendEmail(to, subject, body, attachments) {
    // Configure Nodemailer transport
    // Send email with attachments
    // Return success status and message ID
  }
}
```

#### 5. Calendar Service
```javascript
class CalendarService {
  async createEvent(title, date, time, description) {
    // Create calendar event using Google Calendar API
    // Or generate iCal file
    // Return event ID
  }
}
```

#### 6. PDF Service
```javascript
class PDFService {
  async generatePDF(topic, content) {
    // Create PDF document using PDFKit
    // Format content with proper styling
    // Return PDF buffer
  }
}
```

## Data Models

### History Entry Model
```javascript
const HistorySchema = new mongoose.Schema({
  topic: {
    type: String,
    required: true,
    trim: true
  },
  notes: {
    type: String,
    required: true
  },
  category: {
    type: String,
    enum: ['Programming', 'Math', 'Science', 'Language', ''],
    default: ''
  },
  learningLevel: {
    type: String,
    enum: ['Beginner', 'Intermediate', 'Placement Prep', ''],
    default: ''
  },
  learningGoal: {
    type: String,
    enum: ['Exam', 'Job Interview', 'Project Building', ''],
    default: ''
  },
  mode: {
    type: String,
    enum: ['study', 'medical', 'finance', 'civic'],
    default: 'study'
  }
}, { 
  timestamps: true 
});
```

### Medical Report Model
```javascript
const MedicalReportSchema = new mongoose.Schema({
  userId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  },
  fileName: String,
  extractedText: String,
  insight: String,
  keyIndicators: [{
    name: String,
    value: String,
    normal: Boolean
  }],
  translatedInsights: [{
    language: String,
    text: String
  }]
}, { 
  timestamps: true 
});
```

### Task Model
```javascript
const TaskSchema = new mongoose.Schema({
  userId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  },
  title: String,
  description: String,
  type: {
    type: String,
    enum: ['revision', 'appointment', 'quiz', 'flashcard_review']
  },
  scheduledDate: Date,
  completed: {
    type: Boolean,
    default: false
  },
  relatedDocumentId: mongoose.Schema.Types.ObjectId
}, { 
  timestamps: true 
});
```

### Quiz Model
```javascript
const QuizSchema = new mongoose.Schema({
  topic: String,
  questions: [{
    question: String,
    options: [String],
    correctAnswer: Number,
    explanation: String
  }],
  userAnswers: [Number],
  score: Number,
  completedAt: Date
}, { 
  timestamps: true 
});
```

### Flashcard Set Model
```javascript
const FlashcardSetSchema = new mongoose.Schema({
  topic: String,
  cards: [{
    front: String,
    back: String,
    reviewCount: {
      type: Number,
      default: 0
    },
    lastReviewed: Date
  }]
}, { 
  timestamps: true 
});
```

### User Profile Model (Future Enhancement)
```javascript
const UserSchema = new mongoose.Schema({
  name: String,
  email: String,
  preferredLanguage: {
    type: String,
    enum: ['en', 'hi', 'gu'],
    default: 'en'
  },
  studyReadiness: {
    type: Number,
    default: 0
  }
}, { 
  timestamps: true 
});
```

## Implementation Details

### AI Prompt Engineering

#### Study Notes Prompt Template
```javascript
function buildStudyNotesPrompt(topic, level, goal) {
  const levelContext = {
    'Beginner': 'Explain in very simple terms with basic examples',
    'Intermediate': 'Provide detailed explanations with practical examples',
    'Placement Prep': 'Focus on interview questions and problem-solving approaches'
  };
  
  const goalContext = {
    'Exam': 'Include key points for exam preparation',
    'Job Interview': 'Focus on practical applications and common interview questions',
    'Project Building': 'Emphasize hands-on implementation and best practices'
  };
  
  return `You are an expert teacher.

Explain ${topic} for a ${level} learner preparing for ${goal}.

${levelContext[level]}
${goalContext[goal]}

Provide:
1. Easy Explanation
2. Learning Roadmap
3. Real Life Example
4. Summary

Format the response clearly with section headers.`;
}
```

#### Code Generation Prompt Template
```javascript
function buildCodePrompt(topic, language) {
  return `Write clean, well-commented ${language} code for: ${topic}

Include:
- Clear variable names
- Comments explaining key logic
- Example usage
- Best practices for ${language}`;
}
```

#### Quiz Generation Prompt Template
```javascript
function buildQuizPrompt(topic, count) {
  return `Generate ${count} multiple-choice questions about ${topic}.

For each question provide:
- Question text
- 4 options (A, B, C, D)
- Correct answer
- Brief explanation

Format as JSON array.`;
}
```

### Frontend State Management

The application uses React hooks for state management:

```javascript
// Global state using Context API
const AppContext = React.createContext();

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [language, setLanguage] = useState('en');
  const [currentMode, setCurrentMode] = useState('welcome');
  
  return (
    <AppContext.Provider value={{
      user, setUser,
      language, setLanguage,
      currentMode, setCurrentMode
    }}>
      {children}
    </AppContext.Provider>
  );
}
```

### Error Handling Strategy

#### Frontend Error Handling
```javascript
async function generateNotes() {
  try {
    setLoading(true);
    const response = await fetch('/api/study', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ topic, category })
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const data = await response.json();
    setResult(data.result);
  } catch (error) {
    console.error('Error generating notes:', error);
    alert('Failed to generate notes. Please try again.');
  } finally {
    setLoading(false);
  }
}
```

#### Backend Error Handling
```javascript
router.post('/study', async (req, res) => {
  try {
    const { topic, category } = req.body;
    
    // Validate input
    if (!topic || topic.trim() === '') {
      return res.status(400).json({ 
        error: 'Topic is required' 
      });
    }
    
    // Call AI service
    const notes = await aiService.generateStudyNotes(topic);
    
    // Save to database
    const entry = new History({ topic, notes, category });
    await entry.save();
    
    res.json({ result: notes });
    
  } catch (error) {
    console.error('Study endpoint error:', error);
    res.status(500).json({ 
      error: 'Failed to generate study notes',
      details: error.message 
    });
  }
});
```

### File Upload Handling

```javascript
// Using multer for file uploads
const multer = require('multer');
const upload = multer({ 
  dest: 'uploads/',
  limits: { fileSize: 10 * 1024 * 1024 }, // 10MB limit
  fileFilter: (req, file, cb) => {
    const allowedTypes = ['application/pdf', 'image/jpeg', 'image/png'];
    if (allowedTypes.includes(file.mimetype)) {
      cb(null, true);
    } else {
      cb(new Error('Invalid file type'));
    }
  }
});

router.post('/medical/upload', upload.single('report'), async (req, res) => {
  try {
    const file = req.file;
    const extractedText = await ocrService.extractText(file.path);
    
    res.json({
      fileId: file.filename,
      extractedText
    });
  } catch (error) {
    res.status(500).json({ error: 'Upload failed' });
  }
});
```

### Camera Integration

```javascript
// Frontend camera capture
function CameraCapture({ onCapture }) {
  const videoRef = useRef(null);
  const [stream, setStream] = useState(null);
  
  async function startCamera() {
    try {
      const mediaStream = await navigator.mediaDevices.getUserMedia({
        video: { facingMode: 'environment' }
      });
      videoRef.current.srcObject = mediaStream;
      setStream(mediaStream);
    } catch (error) {
      alert('Camera access denied');
    }
  }
  
  function captureImage() {
    const canvas = document.createElement('canvas');
    canvas.width = videoRef.current.videoWidth;
    canvas.height = videoRef.current.videoHeight;
    canvas.getContext('2d').drawImage(videoRef.current, 0, 0);
    
    canvas.toBlob(blob => {
      onCapture(blob);
      stopCamera();
    }, 'image/jpeg');
  }
  
  function stopCamera() {
    if (stream) {
      stream.getTracks().forEach(track => track.stop());
    }
  }
  
  return (
    <div>
      <video ref={videoRef} autoPlay />
      <button onClick={startCamera}>Start Camera</button>
      <button onClick={captureImage}>Capture</button>
    </div>
  );
}
```

### Study Readiness Calculation

```javascript
function calculateStudyReadiness(userId) {
  // Factors contributing to readiness:
  // 1. Number of topics studied
  // 2. Quizzes completed and scores
  // 3. Flashcards reviewed
  // 4. Scheduled revisions completed
  
  const weights = {
    topicsStudied: 0.3,
    quizzesCompleted: 0.4,
    flashcardsReviewed: 0.2,
    revisionsCompleted: 0.1
  };
  
  const metrics = {
    topicsStudied: await History.countDocuments({ userId, mode: 'study' }),
    quizzesCompleted: await Quiz.countDocuments({ userId, completedAt: { $exists: true } }),
    flashcardsReviewed: await FlashcardSet.aggregate([
      { $match: { userId } },
      { $unwind: '$cards' },
      { $group: { _id: null, total: { $sum: '$cards.reviewCount' } } }
    ]),
    revisionsCompleted: await Task.countDocuments({ 
      userId, 
      type: 'revision', 
      completed: true 
    })
  };
  
  // Normalize and calculate weighted score
  const normalizedScores = {
    topicsStudied: Math.min(metrics.topicsStudied / 20, 1),
    quizzesCompleted: Math.min(metrics.quizzesCompleted / 10, 1),
    flashcardsReviewed: Math.min(metrics.flashcardsReviewed[0]?.total / 100, 1),
    revisionsCompleted: Math.min(metrics.revisionsCompleted / 5, 1)
  };
  
  const readiness = Object.keys(weights).reduce((sum, key) => {
    return sum + (normalizedScores[key] * weights[key]);
  }, 0);
  
  return Math.round(readiness * 100);
}
```


## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Study Mode Properties

**Property 1: Study Notes Generation Completeness**
*For any* valid topic string, when study notes are generated, the result SHALL contain all four required sections: Easy Explanation, Learning Roadmap, Real Life Example, and Summary.
**Validates: Requirements 3.3, 3.4, 3.5, 3.6**

**Property 2: Empty Topic Validation**
*For any* string composed entirely of whitespace or empty string, attempting to generate study notes SHALL be rejected with an alert message, and no API call SHALL be made.
**Validates: Requirements 3.7**

**Property 3: Study Notes Persistence Round-Trip**
*For any* generated study notes with topic, category, learning level, and learning goal, storing the entry in the database and then querying for it SHALL return an entry with all the same field values.
**Validates: Requirements 3.10, 5.6, 24.2**

**Property 4: History Chronological Ordering**
*For any* set of history entries with different timestamps, displaying the history SHALL show entries in reverse chronological order (newest first).
**Validates: Requirements 16.2**

**Property 5: History Search Filtering**
*For any* search term and set of history entries, the filtered results SHALL contain only entries whose topic contains the search term (case-insensitive).
**Validates: Requirements 17.2**

**Property 6: History Category Filtering**
*For any* category selection and set of history entries, the filtered results SHALL contain only entries matching that category.
**Validates: Requirements 17.4**

**Property 7: Combined Filter Logic**
*For any* search term and category selection, the filtered results SHALL contain only entries that match both the search term AND the category.
**Validates: Requirements 17.5**

**Property 8: Tab Switching Without Regeneration**
*For any* displayed study notes, switching between Summary, Notes, and Roadmap tabs SHALL not trigger new API calls to the AI model.
**Validates: Requirements 6.5**

### Code Generation Properties

**Property 9: Code Generation for All Supported Languages**
*For any* valid topic and any supported programming language (Python, C, JavaScript, Java), the system SHALL generate a code snippet.
**Validates: Requirements 10.3**

**Property 10: Code Topic Consistency**
*For any* code generation request, the generated code SHALL be based on the current topic value from the study notes input field.
**Validates: Requirements 10.4**

**Property 11: Empty Topic Code Validation**
*For any* empty or whitespace-only topic, attempting to generate code SHALL be rejected with an alert message.
**Validates: Requirements 10.5**

### Quiz and Flashcard Properties

**Property 12: Quiz Question Structure**
*For any* generated quiz, each question SHALL have a question text, exactly 4 options, a correct answer index, and an explanation.
**Validates: Requirements 7.3, 7.6**

**Property 13: Quiz Score Calculation**
*For any* set of quiz answers submitted, the system SHALL calculate and display a score based on the number of correct answers.
**Validates: Requirements 7.5**

**Property 14: Flashcard Structure**
*For any* generated flashcard set, each flashcard SHALL have a front (question) and back (answer) field.
**Validates: Requirements 8.3**

**Property 15: Flashcard Persistence Round-Trip**
*For any* generated flashcard set, storing it in the database and then querying for it SHALL return the same set of flashcards.
**Validates: Requirements 8.6**

### Medical Mode Properties

**Property 16: Medical Report OCR Processing**
*For any* uploaded medical report image, the system SHALL extract text using the OCR service.
**Validates: Requirements 11.2**

**Property 17: Medical Insight Generation**
*For any* analyzed medical report, the system SHALL generate a medical insight containing key health indicators with their values and normal/abnormal flags.
**Validates: Requirements 12.1, 12.2, 12.3**

**Property 18: Medical Insight Translation**
*For any* medical insight and any supported language (English, Hindi, Gujarati), the system SHALL translate the insight to the selected language.
**Validates: Requirements 12.6**

**Property 19: Medical Report Persistence**
*For any* analyzed medical report, storing the analysis results in the database and then querying for them SHALL return the same insight and key indicators.
**Validates: Requirements 24.4**

### Document Upload Properties

**Property 20: Document Upload OCR Extraction**
*For any* uploaded document in Study Mode, if the document is an image format, the system SHALL extract text using OCR before generating study notes.
**Validates: Requirements 4.2**

**Property 21: Camera Capture OCR Processing**
*For any* image captured via camera, the system SHALL process it using the OCR service to extract text.
**Validates: Requirements 19.4, 19.5**

### Action Automation Properties

**Property 22: Email Composition with Attachments**
*For any* "Email Specialist" action with a medical report, the composed email SHALL include the report as an attachment.
**Validates: Requirements 20.2**

**Property 23: Calendar Event Creation**
*For any* schedule action (revision or doctor appointment) with date, time, and description, the system SHALL create a calendar event with all provided details.
**Validates: Requirements 21.3**

**Property 24: Task Completion Updates Dashboard**
*For any* task marked as complete, the dashboard SHALL update to reflect the new completion status.
**Validates: Requirements 18.7**

### Dashboard and Readiness Properties

**Property 25: Study Readiness Calculation Factors**
*For any* user, the study readiness percentage SHALL be calculated based on completed quizzes, reviewed flashcards, and completed scheduled revisions.
**Validates: Requirements 22.1, 22.2, 22.3, 22.4**

**Property 26: Study Readiness Dynamic Updates**
*For any* user activity (completing quiz, reviewing flashcard, finishing revision), the study readiness percentage SHALL be recalculated and updated.
**Validates: Requirements 22.6**

**Property 27: Dashboard Task Display**
*For any* user with scheduled tasks, the dashboard SHALL display all tasks including doctor appointments and study revisions.
**Validates: Requirements 18.1, 18.2, 18.3**

### Multi-Language Properties

**Property 28: Language Preference Persistence**
*For any* language selection (English, Hindi, Gujarati), storing the preference and then reloading the application SHALL maintain the same language selection.
**Validates: Requirements 23.5**

**Property 29: UI Translation on Language Change**
*For any* language selection, all interface text SHALL be translated to the selected language.
**Validates: Requirements 23.3**

**Property 30: AI Content Language Consistency**
*For any* content generation request (study notes, code, quiz), the AI SHALL generate content in the user's selected language.
**Validates: Requirements 23.4**

### Data Persistence Properties

**Property 31: Timestamp Auto-Generation**
*For any* history entry created, the database SHALL automatically generate and store createdAt and updatedAt timestamp fields.
**Validates: Requirements 24.3**

**Property 32: Quiz Results Persistence**
*For any* completed quiz with answers and score, storing the results in the database and then querying for them SHALL return the same quiz data.
**Validates: Requirements 24.5**

**Property 33: Task Persistence Round-Trip**
*For any* scheduled task with title, description, type, and date, storing it in the database and then querying for it SHALL return a task with all the same field values.
**Validates: Requirements 24.6**

### Error Handling Properties

**Property 34: Invalid Input Error Messages**
*For any* invalid user input (empty fields, wrong format, unsupported file type), the system SHALL display a descriptive error message.
**Validates: Requirements 27.1**

**Property 35: AI Model Failure Handling**
*For any* AI model request that fails, the system SHALL display a user-friendly error message without exposing technical details.
**Validates: Requirements 27.2**

**Property 36: File Upload Error Handling**
*For any* file upload that fails, the system SHALL display the reason for failure to the user.
**Validates: Requirements 27.3**

**Property 37: OCR Failure Recovery**
*For any* OCR operation that fails, the system SHALL display an error message and provide an option to retry.
**Validates: Requirements 27.4, 19.6**

**Property 38: Network Error Handling**
*For any* network request that fails due to connection loss, the system SHALL display a connection error message.
**Validates: Requirements 27.5**

**Property 39: Input Validation Before Processing**
*For any* user input, the system SHALL validate it before making API calls or processing data.
**Validates: Requirements 27.6**

**Property 40: Error Logging**
*For any* error that occurs (API failure, database error, validation error), the system SHALL log the error with timestamp and details.
**Validates: Requirements 27.7, 24.8**

### API Endpoint Properties

**Property 41: Study Notes API Response Structure**
*For any* POST request to /api/study with valid topic, the response SHALL contain a "result" field with the generated study notes.
**Validates: Requirements 26.1**

**Property 42: PDF Generation API Response**
*For any* POST request to /api/study/pdf with topic and notes, the response SHALL be a binary PDF file with correct content-type header.
**Validates: Requirements 26.2**

**Property 43: History API Response Structure**
*For any* GET request to /api/history, the response SHALL be a JSON array of history entries sorted by createdAt in descending order.
**Validates: Requirements 26.3**

**Property 44: Code Generation API Response Structure**
*For any* POST request to /api/study/code with topic and language, the response SHALL contain a "code" field with the generated code snippet.
**Validates: Requirements 26.4**

**Property 45: Quiz Generation API Response Structure**
*For any* POST request to /api/study/quiz with topic, the response SHALL contain a "questions" array with properly structured quiz questions.
**Validates: Requirements 26.5**

**Property 46: Flashcard Generation API Response Structure**
*For any* POST request to /api/study/flashcards with topic, the response SHALL contain a "flashcards" array with front/back pairs.
**Validates: Requirements 26.6**

**Property 47: Medical Upload API Response Structure**
*For any* POST request to /api/medical/upload with valid file, the response SHALL contain "fileId" and "extractedText" fields.
**Validates: Requirements 26.7**

**Property 48: Medical Analysis API Response Structure**
*For any* POST request to /api/medical/analyze with fileId, the response SHALL contain "insight" and "keyIndicators" fields.
**Validates: Requirements 26.8**

**Property 49: Translation API Response Structure**
*For any* POST request to /api/medical/translate with text and target language, the response SHALL contain "translatedText" field.
**Validates: Requirements 26.9**

**Property 50: Email API Response Structure**
*For any* POST request to /api/actions/email with valid email data, the response SHALL contain "success" boolean and "messageId" fields.
**Validates: Requirements 26.10**

**Property 51: Calendar API Response Structure**
*For any* POST request to /api/actions/schedule with event details, the response SHALL contain "success" boolean and "eventId" fields.
**Validates: Requirements 26.11**

**Property 52: Dashboard API Response Structure**
*For any* GET request to /api/dashboard, the response SHALL contain "scheduledTasks", "studyReadiness", "healthAlerts", and "recentActions" fields.
**Validates: Requirements 26.12**

**Property 53: API Error Response Format**
*For any* API request that results in an error, the response SHALL have an appropriate HTTP status code (400 for bad requests, 500 for server errors) and a JSON body with an "error" field.
**Validates: Requirements 26.15, 26.16**

### UI State Properties

**Property 54: Loading Indicator Display**
*For any* asynchronous operation (AI generation, file upload, API call), the system SHALL display a loading indicator while the operation is in progress.
**Validates: Requirements 3.8, 10.6**

**Property 55: Button State During Loading**
*For any* button that triggers an asynchronous operation, the button SHALL be disabled and its text SHALL change to indicate loading state while the operation is in progress.
**Validates: Requirements 25.6**

**Property 56: Content Display After Generation**
*For any* successfully generated content (study notes, code, quiz, flashcards), the system SHALL display the content to the user in the appropriate format.
**Validates: Requirements 3.9, 10.7**

**Property 57: Formatted Code Display**
*For any* generated code snippet, the system SHALL display it in a formatted code block element with appropriate styling.
**Validates: Requirements 25.5**

**Property 58: Formatted Notes Display**
*For any* generated study notes, the system SHALL display them in a pre-formatted text block with readable styling.
**Validates: Requirements 25.4**

### Navigation and Mode Properties

**Property 59: Mode Navigation**
*For any* mode selection (Study, Medical, Finance, Civic), clicking the mode SHALL navigate to the corresponding interface.
**Validates: Requirements 2.6**

**Property 60: History Entry Regeneration**
*For any* history entry clicked, the system SHALL regenerate study notes for that topic and display them.
**Validates: Requirements 16.3, 16.4**

**Property 61: History Refresh After Generation**
*For any* new topic generation, the history display SHALL refresh to include the newly created entry.
**Validates: Requirements 16.5**

### Confirmation and User Review Properties

**Property 62: Action Confirmation Display**
*For any* action recommendation selected (email, schedule, etc.), the system SHALL display a confirmation screen before executing the action.
**Validates: Requirements 9.7, 13.7**

**Property 63: Email Review Before Sending**
*For any* email composition action, the system SHALL allow the user to review the email content and attachments before sending.
**Validates: Requirements 20.3**

**Property 64: Email Delivery Confirmation**
*For any* successfully sent email, the system SHALL display a confirmation message to the user.
**Validates: Requirements 20.5, 13.8**

### Performance Properties

**Property 65: Image Compression Before Upload**
*For any* image uploaded or captured via camera, the system SHALL compress the image before sending it to the server.
**Validates: Requirements 28.5**

**Property 66: Progress Indicator for Large Documents**
*For any* document processing operation that takes significant time, the system SHALL display a progress indicator.
**Validates: Requirements 28.6**

## Error Handling

### Frontend Error Handling

1. **Input Validation**: All user inputs are validated before API calls
   - Empty/whitespace-only strings are rejected
   - File types are checked before upload
   - Required fields are enforced

2. **Network Errors**: Graceful handling of connection failures
   - Display user-friendly error messages
   - Provide retry options
   - Maintain application state

3. **API Errors**: Handle various HTTP status codes
   - 400: Display validation error messages
   - 500: Display generic server error message
   - Timeout: Display timeout message with retry option

4. **UI Error States**: Visual feedback for errors
   - Error messages displayed near relevant inputs
   - Error styling (red borders, error icons)
   - Clear error dismissal mechanisms

### Backend Error Handling

1. **Request Validation**: Validate all incoming requests
   - Check required fields
   - Validate data types and formats
   - Sanitize inputs to prevent injection attacks

2. **AI Model Errors**: Handle AI service failures
   - Retry logic for transient failures
   - Fallback responses when AI is unavailable
   - Timeout handling for long-running requests

3. **Database Errors**: Handle database operation failures
   - Connection pool management
   - Transaction rollback on errors
   - Graceful degradation when database is unavailable

4. **External Service Errors**: Handle third-party service failures
   - OCR service failures
   - Translation service failures
   - Email service failures
   - Calendar service failures

5. **Error Logging**: Comprehensive error logging
   - Log all errors with timestamps
   - Include stack traces for debugging
   - Log request context (user, endpoint, parameters)

### Error Response Format

```javascript
{
  "error": "Human-readable error message",
  "code": "ERROR_CODE",
  "details": "Additional technical details (optional)",
  "timestamp": "2024-01-15T10:30:00Z"
}
```

## Testing Strategy

### Dual Testing Approach

The system requires both unit testing and property-based testing for comprehensive coverage:

- **Unit tests**: Verify specific examples, edge cases, and error conditions
- **Property tests**: Verify universal properties across all inputs
- Both approaches are complementary and necessary

### Unit Testing

Unit tests focus on:
- Specific examples that demonstrate correct behavior
- Integration points between components
- Edge cases (empty inputs, boundary values, special characters)
- Error conditions (network failures, invalid data, service unavailable)

**Example Unit Tests**:
```javascript
// Test specific welcome screen rendering
test('Welcome screen displays correct tagline', () => {
  render(<WelcomeScreen />);
  expect(screen.getByText('Understand. Decide. Act.')).toBeInTheDocument();
});

// Test specific error case
test('Empty topic shows alert', () => {
  const alertSpy = jest.spyOn(window, 'alert');
  generateNotes('');
  expect(alertSpy).toHaveBeenCalledWith('Enter a topic first!');
});

// Test API endpoint exists
test('POST /api/study endpoint exists', async () => {
  const response = await request(app)
    .post('/api/study')
    .send({ topic: 'React Hooks' });
  expect(response.status).not.toBe(404);
});
```

### Property-Based Testing

Property tests verify universal properties across many generated inputs. Each property test should:
- Run minimum 100 iterations (due to randomization)
- Reference its design document property
- Use appropriate generators for test data

**Property Testing Library**: Use `fast-check` for JavaScript/TypeScript

**Example Property Tests**:
```javascript
// Property 1: Study Notes Generation Completeness
test('Property 1: Generated notes contain all required sections', () => {
  fc.assert(
    fc.asyncProperty(
      fc.string({ minLength: 1 }), // Generate random non-empty topics
      async (topic) => {
        const notes = await generateStudyNotes(topic);
        expect(notes).toContain('Easy Explanation');
        expect(notes).toContain('Learning Roadmap');
        expect(notes).toContain('Real Life Example');
        expect(notes).toContain('Summary');
      }
    ),
    { numRuns: 100 }
  );
});
// Feature: ai-study-notes-generator, Property 1: For any valid topic string, when study notes are generated, the result SHALL contain all four required sections

// Property 3: Study Notes Persistence Round-Trip
test('Property 3: History entry round-trip preserves all fields', () => {
  fc.assert(
    fc.asyncProperty(
      fc.record({
        topic: fc.string({ minLength: 1 }),
        notes: fc.string(),
        category: fc.constantFrom('Programming', 'Math', 'Science', 'Language', ''),
        learningLevel: fc.constantFrom('Beginner', 'Intermediate', 'Placement Prep', ''),
        learningGoal: fc.constantFrom('Exam', 'Job Interview', 'Project Building', '')
      }),
      async (entry) => {
        const saved = await History.create(entry);
        const retrieved = await History.findById(saved._id);
        expect(retrieved.topic).toBe(entry.topic);
        expect(retrieved.notes).toBe(entry.notes);
        expect(retrieved.category).toBe(entry.category);
        expect(retrieved.learningLevel).toBe(entry.learningLevel);
        expect(retrieved.learningGoal).toBe(entry.learningGoal);
      }
    ),
    { numRuns: 100 }
  );
});
// Feature: ai-study-notes-generator, Property 3: For any generated study notes with topic, category, learning level, and learning goal, storing the entry in the database and then querying for it SHALL return an entry with all the same field values

// Property 4: History Chronological Ordering
test('Property 4: History entries are sorted newest first', () => {
  fc.assert(
    fc.asyncProperty(
      fc.array(
        fc.record({
          topic: fc.string({ minLength: 1 }),
          notes: fc.string(),
          createdAt: fc.date()
        }),
        { minLength: 2, maxLength: 10 }
      ),
      async (entries) => {
        // Save entries to database
        await History.insertMany(entries);
        
        // Retrieve sorted entries
        const sorted = await History.find().sort({ createdAt: -1 });
        
        // Verify order
        for (let i = 0; i < sorted.length - 1; i++) {
          expect(sorted[i].createdAt >= sorted[i + 1].createdAt).toBe(true);
        }
      }
    ),
    { numRuns: 100 }
  );
});
// Feature: ai-study-notes-generator, Property 4: For any set of history entries with different timestamps, displaying the history SHALL show entries in reverse chronological order

// Property 25: Study Readiness Calculation Factors
test('Property 25: Readiness calculation includes all activity types', () => {
  fc.assert(
    fc.asyncProperty(
      fc.record({
        quizzesCompleted: fc.nat(20),
        flashcardsReviewed: fc.nat(200),
        revisionsCompleted: fc.nat(10)
      }),
      async (activities) => {
        const userId = new mongoose.Types.ObjectId();
        
        // Create test data
        await Quiz.insertMany(
          Array(activities.quizzesCompleted).fill({ userId, completedAt: new Date() })
        );
        await Task.insertMany(
          Array(activities.revisionsCompleted).fill({ 
            userId, 
            type: 'revision', 
            completed: true 
          })
        );
        
        const readiness = await calculateStudyReadiness(userId);
        
        // Readiness should be between 0 and 100
        expect(readiness).toBeGreaterThanOrEqual(0);
        expect(readiness).toBeLessThanOrEqual(100);
        
        // More activities should generally increase readiness
        // (This is a metamorphic property)
      }
    ),
    { numRuns: 100 }
  );
});
// Feature: ai-study-notes-generator, Property 25: For any user, the study readiness percentage SHALL be calculated based on completed quizzes, reviewed flashcards, and completed scheduled revisions
```

### Integration Testing

Integration tests verify that components work together correctly:
- Frontend-backend communication
- Database operations
- External service integrations (OCR, translation, email, calendar)
- End-to-end user workflows

### Test Coverage Goals

- Unit test coverage: 80%+ for business logic
- Property test coverage: All 66 correctness properties implemented
- Integration test coverage: All major user workflows
- API endpoint coverage: 100% of endpoints tested

### Testing Tools

- **Frontend**: Jest, React Testing Library, fast-check
- **Backend**: Jest, Supertest, fast-check, mongodb-memory-server
- **E2E**: Playwright or Cypress (optional)
- **API Testing**: Postman/Newman for manual and automated API testing
