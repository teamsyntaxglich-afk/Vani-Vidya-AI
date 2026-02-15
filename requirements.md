# Bharat-Kiro Mentor - Requirements Document

## Project Overview

Bharat-Kiro Mentor is an AI-powered learning bridge designed to democratize AWS cloud technology education for rural and non-technical Indian students by eliminating the "language tax" and complexity barriers through vernacular support, intelligent analogies, and a scaffolding-based education model.

## Problem Statement

Rural and non-technical Indian students face significant barriers when learning and building on AWS:
- Technical documentation is primarily in English with complex terminology
- Lack of culturally relevant examples and analogies
- High cognitive load when translating concepts across languages
- Limited access to mentors who can explain in native languages
- Overwhelming documentation that slows prototype development from hours to days
- Absence of progressive learning paths that adapt to user skill levels

## Target User Personas

### Primary Persona: Rural Student Developer
- Age: 18-25 years
- Location: Tier 2/3 cities and rural areas in India
- Languages: Hindi, Marathi, Hinglish (primary), English (limited)
- Technical Background: Basic programming knowledge, minimal cloud experience
- Goals: Build working prototypes, learn AWS services, participate in hackathons
- Pain Points: Language barriers, complex documentation, lack of local mentorship

### Secondary Persona: Non-Technical Founder
- Age: 25-40 years
- Location: Small towns and emerging startup hubs
- Languages: Regional languages preferred for learning
- Technical Background: Domain expertise but limited technical knowledge
- Goals: Understand AWS capabilities, communicate with technical teams, validate ideas quickly
- Pain Points: Technical jargon, inability to prototype ideas, dependency on technical co-founders

## Educational Philosophy: Scaffolding Model

Bharat-Kiro Mentor employs a scaffolding-based education approach inspired by Vygotsky's Zone of Proximal Development (ZPD):

### Scaffolding Principles
- **Progressive Complexity**: Start with simplified analogies, gradually introduce technical depth
- **Contextual Support**: Provide just-in-time help based on user's current task
- **Adaptive Fading**: Reduce assistance as user demonstrates competency
- **Cultural Anchoring**: Connect new AWS concepts to familiar local experiences
- **Immediate Feedback**: Real-time validation and correction through Amazon Q

### Learning Stages
1. **Foundation**: Analogies and visual explanations (Non-Tech Persona)
2. **Transition**: Mixed analogies with technical terms (Adaptive Mode)
3. **Mastery**: Full technical documentation and best practices (Tech Persona)
4. **Independence**: Self-guided exploration with on-demand support

## Core Requirements

### Functional Requirements

#### FR1: Multi-Modal Learning Support
- Audio explanations using Amazon Polly Neural voices (Hindi: Aditi, English-IN: Kajal)
- Visual architecture diagrams auto-generated from CDK code
- Text-based documentation in vernacular languages with code-switching support
- Interactive Q&A capability through conversational AI (Amazon Bedrock Claude 3.5)
- Real-time voice input for hands-free learning

#### FR2: Vernacular Language Support
- Primary languages: Hindi, Marathi, Hinglish (code-mixed)
- Real-time translation of AWS documentation with context preservation
- Language detection and automatic switching
- Support for bilingual code comments and documentation
- Phonetic search for technical terms in native scripts

#### FR3: Logic-to-Analogy (LTA) Engine
- Map technical AWS concepts to culturally relevant analogies
- Use local stories, festivals, and everyday scenarios (dabbawalas, village panchayat, etc.)
- Maintain technical accuracy while simplifying complexity
- Adaptive difficulty based on user comprehension metrics
- Semantic caching in DynamoDB for frequently requested analogies

#### FR4: Persona Switcher (Adaptive Learning Interface)
- **Non-Tech Mode**: Business-focused explanations with zero jargon
  - Use case: Founders, product managers, domain experts
  - Output: "Lambda ek dhaba hai jo sirf tab khulta hai jab customer aata hai"
  - Focus: What it does, why it matters, cost implications
  
- **Tech Mode**: Detailed technical specifications with best practices
  - Use case: Developers, engineers, technical students
  - Output: "Lambda supports 128MB-10GB memory, 15-min timeout, event-driven invocation"
  - Focus: How it works, configuration options, optimization techniques

- **Adaptive Mode**: Dynamic switching based on query complexity
  - Analyzes user's question to determine appropriate depth
  - Gradually introduces technical terms with inline explanations
  - Tracks user progress to adjust future responses

- Features:
  - Seamless switching via UI toggle or voice command
  - Context retention across persona switches
  - Learning path recommendations based on persona usage patterns
  - Progress tracking: Non-Tech → Adaptive → Tech transition

#### FR5: Auto-Kiro Documentation Generator
- Automatically generate comprehensive project documentation from code in seconds
- **Time Reduction**: From 4+ hours of manual writing to 30 seconds of automated generation
- Create README files in vernacular languages (Hindi/Marathi/English)
- Generate architecture diagrams from AWS CDK infrastructure code
- Produce step-by-step deployment guides with troubleshooting tips
- Extract and translate code comments into documentation
- Generate API documentation with request/response examples
- Create security checklists audited by Amazon Q

#### FR6: Rapid Prototyping Acceleration
- Pre-built AWS CDK templates for common use cases (web apps, APIs, data pipelines)
- Interactive code generation with line-by-line explanations
- One-command deployment workflows with progress tracking
- Built-in testing and validation using AWS CDK assertions
- Automated rollback on deployment failures
- Cost estimation before deployment

#### FR7: Code Auditing with Amazon Q
- Real-time code review for security vulnerabilities
- AWS best practices validation (Well-Architected Framework)
- IAM policy analysis for least privilege compliance
- Cost optimization recommendations
- Performance bottleneck identification
- Automated fix suggestions with explanations

### Non-Functional Requirements

#### NFR1: Performance
- Response time < 3 seconds for text queries
- Audio generation < 5 seconds for 100-word explanations
- Support 100+ concurrent users
- 99.5% uptime during hackathon hours

#### NFR2: Usability
- Intuitive interface requiring < 5 minutes onboarding
- Mobile-responsive design for smartphone access
- Offline capability for generated documentation
- Keyboard shortcuts and voice commands

#### NFR3: Scalability
- Serverless architecture for automatic scaling
- Support for 10,000+ users during hackathon events
- Efficient caching for frequently accessed content
- CDN integration for global content delivery

#### NFR4: Security
- IAM role-based access control
- Encryption at rest and in transit
- No storage of sensitive user data
- Compliance with AWS security best practices

#### NFR5: Maintainability
- Infrastructure as Code using AWS CDK
- Comprehensive logging and monitoring
- Automated testing and deployment pipelines
- Clear documentation for contributors

## Success Metrics

### Primary Metrics

#### Learning Efficiency
- Target: 60% faster comprehension of AWS concepts
- Measurement: Time to complete learning modules (with vs without Bharat-Kiro)
- Baseline: Average 2 hours to understand basic AWS service
- Goal: Reduce to 48 minutes with vernacular explanations

#### Documentation Productivity
- Target: 99% reduction in documentation time
- Measurement: Time spent writing project documentation
- Baseline: 4 hours (240 minutes) for comprehensive project docs
- Goal: Auto-generate in 30 seconds (from hours to seconds)

#### Prototype Velocity
- Target: 10x faster from idea to working prototype
- Measurement: Time from concept to deployed AWS application
- Baseline: 2 weeks for first-time AWS users
- Goal: 1.4 days with guided CDK templates

### Secondary Metrics

#### User Engagement
- Daily active users during hackathon
- Average session duration > 20 minutes
- Return user rate > 70%
- Feature adoption rate for LTA Engine > 80%

#### Quality Metrics
- User satisfaction score > 4.5/5
- Analogy relevance rating > 4.0/5
- Documentation accuracy > 95%
- Successful deployment rate > 85%

#### Business Impact
- Number of prototypes built using the platform
- AWS service adoption among rural students
- Community contributions and feedback
- Hackathon project submissions using Bharat-Kiro

## User Stories

### Epic 1: Vernacular Learning
- As a rural student, I want to learn AWS Lambda in Hindi so that I can understand without translating mentally
- As a non-tech founder, I want to hear explanations in Marathi so that I can grasp concepts while multitasking
- As a developer, I want to switch between Hinglish and English so that I can learn at my comfort level

### Epic 2: Scaffolding-Based Learning
- As a beginner, I want technical concepts explained through local analogies so that I can relate to familiar scenarios
- As a student, I want the system to gradually increase complexity as I learn so that I'm never overwhelmed
- As a founder, I want business-focused explanations so that I can make informed decisions
- As a learner, I want to track my progress from Non-Tech to Tech persona so that I can see my growth
- As a developer, I want code audited by Amazon Q so that I learn best practices from the start

### Epic 3: Rapid Development
- As a hackathon participant, I want pre-built CDK templates so that I can focus on my unique idea
- As a developer, I want auto-generated documentation in seconds so that I can eliminate hours of manual writing
- As a student, I want one-command deployment so that I can see my app live quickly
- As a learner, I want real-time code feedback from Amazon Q so that I can fix issues before deployment

## Constraints and Assumptions

### Constraints
- Must use AWS services exclusively for cloud infrastructure
- Must support at least 3 Indian languages at launch
- Must be deployable within AWS Free Tier limits for individual users
- Must complete MVP within hackathon timeline

### Assumptions
- Users have basic internet connectivity (3G minimum)
- Users have access to AWS account (free tier)
- Users have basic programming knowledge (any language)
- Users are comfortable with command-line interfaces

## Out of Scope (Phase 1)

- Support for languages beyond Hindi, Marathi, Hinglish
- Mobile native applications (web-first approach)
- Real-time video tutoring
- Integration with non-AWS cloud providers
- Advanced DevOps workflows (CI/CD pipelines)
- Collaborative coding features

## Dependencies

### Technical Dependencies
- Amazon Bedrock (Claude 3.5 Sonnet) for AI/ML capabilities
- Amazon Polly (Neural TTS) for text-to-speech in Indian languages
- AWS Lambda for serverless compute and microservices
- AWS Step Functions for orchestration and workflow management
- Amazon S3 for blob storage (audio files, generated documents)
- Amazon DynamoDB for metadata, caching, and project history
- AWS Amplify for frontend hosting and deployment
- Amazon API Gateway for API management with JWT authorization
- Amazon Cognito for user authentication and authorization
- AWS CDK for infrastructure as code automation
- Amazon Q for code auditing and best practices validation
- Amazon CloudWatch for logging and monitoring
- AWS X-Ray for distributed tracing and performance analysis

### External Dependencies
- AWS account approval for users
- Internet connectivity for users
- Browser compatibility (Chrome, Firefox, Safari)
- AWS service availability in user regions

## Technology Stack Summary

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Frontend | AWS Amplify + React.js | Persona-adaptive dashboard, responsive UI |
| Authentication | Amazon Cognito | User management, JWT tokens |
| API Layer | Amazon API Gateway | RESTful API with JWT authorization |
| Orchestration | AWS Step Functions | Microservice coordination, retry logic |
| Compute | AWS Lambda | Decoupled services (Bedrock, Polly, Q) |
| AI/ML | Amazon Bedrock (Claude 3.5) | Analogies, translations, code generation |
| Voice | Amazon Polly (Neural) | Text-to-speech in Indian languages |
| Code Audit | Amazon Q | Security, best practices, cost optimization |
| Database | Amazon DynamoDB | Project history, semantic caching |
| Storage | Amazon S3 | Audio files, generated documentation |
| Monitoring | CloudWatch + X-Ray | Performance metrics, distributed tracing |
| IaC | AWS CDK | Infrastructure automation |

## Acceptance Criteria

The Bharat-Kiro Mentor project will be considered successful when:
1. Users can receive AWS explanations in Hindi, Marathi, or Hinglish with <3s response time
2. LTA Engine generates culturally relevant analogies with >80% user approval rating
3. Auto-documentation reduces manual effort from hours to seconds (>99% reduction)
4. Persona Switcher adapts to user level with >90% accuracy
5. Users deploy first AWS prototype in <2 days (vs 2 weeks baseline)
6. Platform handles 100+ concurrent users without degradation
7. Security review by Amazon Q shows no critical vulnerabilities
8. Total AWS cost per user remains within Free Tier limits for first 1000 users
9. Step Functions orchestration achieves >99.5% success rate with automatic retries
10. CloudWatch dashboards provide real-time visibility into system performance and costs
