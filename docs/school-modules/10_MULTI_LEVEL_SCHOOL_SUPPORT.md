# Multi-Level School Support - U-USMS

## Overview

U-USMS supports all educational levels in Uganda from Nursery to University.

---

## 1. Nursery & Kindergarten Module

### Features

**Child Tracking:**
- Enrollment with photo
- Developmental milestones tracking
- Growth monitoring (height, weight)
- Daily activity logging

**Assessment:**
- Non-exam based evaluation
- Skills-based progress tracking
  - Motor skills
  - Social skills
  - Cognitive development
  - Language development
- Termly progress reports

**Parent Communication:**
- Daily activity updates
- Photo sharing (with privacy controls)
- Meal tracking
- Nap time logging
- Incident reporting

### Sample Report Card

```
===============================================
    HAPPY TOTS KINDERGARTEN
    PROGRESS REPORT - TERM 1 2026
===============================================

Child: Emma Johnson
Class: Baby Class
Age: 3 years, 2 months

DEVELOPMENTAL AREAS:

Physical Development:
✓ Gross Motor Skills: Excellent
✓ Fine Motor Skills: Good
✓ Self-care: Developing

Social & Emotional:
✓ Plays well with others: Excellent
✓ Follows instructions: Good
✓ Emotional regulation: Developing

Cognitive:
✓ Recognizes colors: Excellent
✓ Counts to 10: Good
✓ Shape recognition: Excellent

Language:
✓ Vocabulary: Excellent
✓ Expression: Good
✓ Listening: Excellent

Teacher's Comment:
Emma is a bright and cheerful child who enjoys
learning and playing with her classmates.

Teacher: Ms. Sarah Nakato
Date: 15-Feb-2026
===============================================
```

---

## 2. Primary Schools Module

### Features

**PLE Grading System:**
- Aggregate calculation (4 subjects)
- Division determination (D1, D2, D3, D4, U)
- UNEB result integration

**Competency-Based Curriculum:**
- Learning areas tracking
- Skill acquisition monitoring
- Project-based assessment

**Continuous Assessment:**
- BOT (Beginning of Term) tests
- MOT (Mid of Term) tests
- EOT (End of Term) exams
- Class exercises and homework

### PLE Grading Example

```
Student: John Doe
Class: Primary 7

PLE RESULTS:

English:       6  (Credit)
Mathematics:   5  (Credit)
Science:       7  (Distinction)
SST:          6  (Credit)
----------------------
Aggregate:    24
Division:     II

SUBJECT BREAKDOWN:
9 = Distinction 1
8 = Distinction 2
7 = Distinction 3
6 = Credit 4
5 = Credit 5
...
U = Ungraded (Fail)
```

---

## 3. Secondary Schools Module

### UCE (O-Level) System

**Subjects:**
- Compulsory: English, Mathematics, Science
- Optional: 5-6 subjects from various streams

**Grading:**
- D1: 1 (Distinction)
- D2: 2 (Distinction)
- C3, C4, C5, C6: Credit
- P7, P8: Pass
- F9: Fail

**Aggregate Calculation:**
Best 8 subjects (lower is better)

### UACE (A-Level) System

**Subject Combinations:**
- Sciences: Physics, Chemistry, Biology/Mathematics
- Arts: History, Literature, Geography
- Commercial: Economics, Entrepreneurship, Accounts

**Grading:**
- Principal passes: A, B, C, D, E
- Subsidiary passes: a, b, c, d, e
- Fail: F, f

### Sample UCE Report Card

```
===============================================
    KAMPALA SECONDARY SCHOOL
    UCE MOCK EXAMINATION RESULTS
===============================================

Student: Jane Smith
Class: Senior 4 East
Index Number: U1234/567

SUBJECTS:

English Language:     C5
Mathematics:          D2
Physics:              C4
Chemistry:            C3
Biology:              C5
History:              C6
Geography:            C4
Christian RE:         C3
Luganda:              D1
----------------------
AGGREGATE:            32
DIVISION:             II

Target for UNEB: Aggregate < 30 (Division I)

Class Teacher: Mr. Patrick Okello
Date: 15-Feb-2026
===============================================
```

---

## 4. Vocational & Technical Institutions

### Features

**Modular Course Structure:**
- Competency units
- Credit accumulation
- Flexible learning paths

**Practical Assessment:**
- Workshop evaluations
- Project assessments
- Industry attachments
- Skill demonstrations

**Certification:**
- Uganda National Vocational Qualifications (UNVQ)
- Certificate, Diploma, Advanced Diploma
- Industry-recognized credentials

**Internship Tracking:**
- Placement management
- Supervisor evaluations
- Logbook monitoring

### Sample Certificate Record

```
===============================================
  UGANDA TECHNICAL COLLEGE
  COMPETENCY-BASED TRAINING RECORD
===============================================

Student: Moses Muwanga
Program: Automotive Mechanics
Level: Certificate III

MODULES COMPLETED:

✓ Engine Systems (80%)
✓ Electrical Systems (75%)
✓ Brake Systems (88%)
✓ Transmission Systems (78%)
✓ Workshop Safety (95%)

PRACTICAL ASSESSMENTS:
✓ Engine Overhaul: Competent
✓ Diagnostic Testing: Competent
✓ Safety Procedures: Highly Competent

INTERNSHIP:
Company: ABC Motors Ltd
Duration: 3 months
Supervisor Rating: Excellent (92%)

OVERALL COMPETENCY: 82%
STATUS: Qualified for Certification

Assessor: Eng. David Ssemakula
Date: 15-Feb-2026
===============================================
```

---

## 5. Universities & Tertiary Module

### Features

**Credit-Hour System:**
- Course unit registration
- Credit tracking per semester
- GPA calculation (4.0 scale)

**Semester Management:**
- Course scheduling
- Enrollment management
- Drop/Add periods
- Examination timetables

**Academic Transcript Engine:**
- Cumulative GPA
- Semester-by-semester breakdown
- Course grades
- Academic standing (probation, dean's list)

### GPA Calculation

```javascript
function calculateGPA(courses) {
  const gradePoints = {
    'A+': 4.0, 'A': 4.0, 'A-': 3.7,
    'B+': 3.3, 'B': 3.0, 'B-': 2.7,
    'C+': 2.3, 'C': 2.0, 'C-': 1.7,
    'D+': 1.3, 'D': 1.0,
    'F': 0.0
  };
  
  let totalPoints = 0;
  let totalCredits = 0;
  
  courses.forEach(course => {
    const points = gradePoints[course.grade] * course.credits;
    totalPoints += points;
    totalCredits += course.credits;
  });
  
  const gpa = totalPoints / totalCredits;
  return gpa.toFixed(2);
}

// Example:
// Course 1: A (4.0) × 3 credits = 12.0
// Course 2: B+ (3.3) × 3 credits = 9.9
// Course 3: A- (3.7) × 4 credits = 14.8
// Total: 36.7 / 10 credits = 3.67 GPA
```

### Sample University Transcript

```
===============================================
    MAKERERE UNIVERSITY
    OFFICIAL ACADEMIC TRANSCRIPT
===============================================

Name: Sarah Nalubega
Student Number: 2023/HD07/12345
Program: Bachelor of Science in Computer Science
Faculty: Computing and Information Technology

SEMESTER 1 - 2023/2024
Course Code  Course Name              Credits  Grade
----------------------------------------------------------------
CSC 1101     Intro to Programming       4      A
MAT 1101     Calculus I                 4      B+
PHY 1101     Physics I                  3      A-
ENG 1101     Communication Skills       3      B
CSC 1102     Computer Architecture      3      A-
                                       ---    ------
                           Semester GPA: 3.65

SEMESTER 2 - 2023/2024
Course Code  Course Name              Credits  Grade
----------------------------------------------------------------
CSC 1201     Data Structures            4      A
MAT 1201     Calculus II                4      B
PHY 1201     Physics II                 3      B+
CSC 1202     Database Systems           3      A
CSC 1203     Web Programming            3      A-
                                       ---    ------
                           Semester GPA: 3.70

CUMULATIVE:
Total Credits Earned: 34
Cumulative GPA: 3.68
Academic Standing: Good Standing

Classification: First Class (GPA ≥ 3.60)

Issued: 15-February-2026
Registrar: Dr. John Mukasa

===============================================
TRANSCRIPT VERIFICATION:
QR Code: [QR CODE IMAGE]
Verification URL: verify.mak.ac.ug/T123456
Blockchain Hash: 0x1a2b3c4d5e6f...
===============================================
```

---

## Unified Features Across All Levels

### Common Modules

1. **Student Management**
   - Enrollment
   - Demographics
   - Medical records
   - Guardian information

2. **Attendance**
   - Daily marking
   - Absence tracking
   - Attendance reports

3. **Financial**
   - Fee structures (level-specific)
   - Payment tracking
   - Invoicing

4. **Communication**
   - SMS notifications
   - Parent portal
   - Email updates

5. **Reporting**
   - Academic reports
   - Financial reports
   - Attendance reports

### Level-Specific Customization

```javascript
// Configuration per school type
const schoolConfig = {
  nursery: {
    assessment_type: 'developmental',
    grading_system: 'skills_based',
    features: ['growth_tracking', 'daily_activities']
  },
  
  primary: {
    assessment_type: 'continuous',
    grading_system: 'ple',
    features: ['ple_aggregate', 'competency_tracking']
  },
  
  secondary: {
    assessment_type: 'examination',
    grading_system: 'uce_uace',
    features: ['stream_management', 'subject_combinations']
  },
  
  vocational: {
    assessment_type: 'competency',
    grading_system: 'modular',
    features: ['practical_assessment', 'internship_tracking']
  },
  
  university: {
    assessment_type: 'credit_hours',
    grading_system: 'gpa',
    features: ['course_registration', 'transcript_generation']
  }
};
```

---

## Implementation Priority

**Phase 1 (Months 1-3):**
- ✓ Primary schools
- ✓ Secondary schools (UCE/UACE)

**Phase 2 (Months 4-6):**
- ✓ Nursery/Kindergarten
- ✓ University module

**Phase 3 (Months 7-9):**
- ✓ Vocational/Technical
- ✓ International curricula (Cambridge, IB)

---

**Next:** See `docs/business/09_BUSINESS_MODEL.md`
