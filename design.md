# Vani-Vidya Mentor - Design Document

## Executive Summary

Vani-Vidya Mentor is a serverless, event-driven, AI-powered learning platform built entirely on AWS that bridges the gap between complex cloud technology and rural Indian students through vernacular language support, cultural analogies, scaffolding-based education, and automated development workflows. The architecture leverages AWS Step Functions for orchestration, Amazon Bedrock (Claude 3.5) for intelligent content generation, and Amazon Q for code auditing to deliver a comprehensive learning experience.

## System Architecture

### High-Level Architecture (Event-Driven Serverless)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    User Interface Layer                              │
│  React.js App hosted on AWS Amplify (Persona-Adaptive Dashboard)    │
│  - Non-Tech Persona UI  |  Tech Persona UI  |  Adaptive Mode        │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   Amazon Cognito (Authentication)                    │
│  - User Pools  |  JWT Token Generation  |  MFA Support              │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│              Amazon API Gateway (REST API + WebSocket)               │
│  - JWT Authorization  |  Rate Limiting  |  Request Validation       │
│  - CORS Configuration  |  API Keys  |  Usage Plans                  │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    AWS Step Functions (Orchestrator)                 │
│  ┌───────────────────────────────────────────────────────────┐      │
│  │  Workflow: Query Processing                               │      │
│  │  1. Validate Input → 2. Check Cache → 3. Route to Service │      │
│  │  4. Generate Response → 5. Cache Result → 6. Return       │      │
│  │  - Automatic Retries  |  Error Handling  |  Parallel Tasks│      │
│  └───────────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│ Lambda: Query    │  │ Lambda: LTA      │  │ Lambda: AutoDoc  │
│ Handler          │  │ Engine           │  │ Generator        │
│ (Python 3.12)    │  │ (Python 3.12)    │  │ (Python 3.12)    │
└──────────────────┘  └──────────────────┘  └──────────────────┘
        │                     │                     │
        ▼                     ▼                     ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│ Lambda: Polly    │  │ Lambda: Amazon Q │  │ Lambda: Persona  │
│ Service          │  │ Code Auditor     │  │ Switcher         │
│ (Audio Gen)      │  │ (Best Practices) │  │ (Adaptive)       │
└──────────────────┘  └──────────────────┘  └──────────────────┘
        │                     │                     │
        └─────────────────────┼─────────────────────┘
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                 Amazon Bedrock (Claude 3.5 Sonnet)                   │
│  - Analogy Generation  |  Translation  |  Code Analysis             │
│  - Documentation Generation  |  Conversational AI                   │
└─────────────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│ Amazon Polly     │  │ Amazon DynamoDB  │  │ Amazon S3        │
│ (Neural TTS)     │  │ - Semantic Cache │  │ - Audio Files    │
│ Hindi: Aditi     │  │ - User Profiles  │  │ - Documents      │
│ English: Kajal   │  │ - Project History│  │ - CDK Templates  │
└──────────────────┘  └──────────────────┘  └──────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│ CloudWatch       │  │ AWS X-Ray        │  │ Amazon Q         │
│ - Logs & Metrics │  │ - Distributed    │  │ - Code Review    │
│ - Dashboards     │  │   Tracing        │  │ - Security Audit │
│ - Alarms         │  │ - Performance    │  │ - Cost Optimize  │
└──────────────────┘  └──────────────────┘  └──────────────────┘
```

### Architecture Principles

- **Serverless-First**: Zero server management, automatic scaling, pay-per-use
- **Event-Driven**: Asynchronous processing with Step Functions orchestration
- **Microservices**: Decoupled Lambda functions for single responsibility
- **Cost-Optimized**: Free Tier friendly, semantic caching, efficient resource allocation
- **Security-First**: Least privilege IAM, encryption everywhere, Amazon Q audits
- **Observable**: CloudWatch metrics, X-Ray tracing, real-time dashboards
- **Resilient**: Automatic retries, error handling, graceful degradation
- **Multi-Region Ready**: CloudFront CDN, S3 cross-region replication

## Core Components

### 1. API Management Layer

**Purpose**: Secure, scalable API gateway with JWT authorization

**Technology Stack**:
- Amazon API Gateway (REST API + WebSocket API)
- Amazon Cognito User Pools (JWT token generation)
- AWS WAF (Web Application Firewall)

**API Gateway Configuration**:

```yaml
API Endpoints:
  /query:
    POST: Submit user query (text/voice)
    Authorization: JWT (Cognito)
    Rate Limit: 100 requests/minute per user
    
  /persona:
    PUT: Switch between Tech/Non-Tech mode
    Authorization: JWT (Cognito)
    
  /documentation:
    POST: Generate project documentation
    Authorization: JWT (Cognito)
    Timeout: 30 seconds
    
  /audit:
    POST: Request Amazon Q code review
    Authorization: JWT (Cognito)
    
  /history:
    GET: Retrieve user's project history
    Authorization: JWT (Cognito)
    Cache: 5 minutes

WebSocket API:
  /connect: Establish real-time connection
  /message: Streaming responses for long queries
  /disconnect: Clean up connection
```

**JWT Authorization Flow**:
1. User authenticates via Cognito (email/password or social login)
2. Cognito issues JWT access token (1-hour expiry) and refresh token
3. Client includes JWT in Authorization header: `Bearer <token>`
4. API Gateway validates JWT signature and expiration
5. Custom authorizer extracts user_id and persona from token claims
6. Request forwarded to Lambda with user context
7. Refresh token used to obtain new access token when expired

**Security Features**:
- CORS configuration for web app domain only
- API keys for partner integrations
- Usage plans with throttling (burst: 200, rate: 100/sec)
- Request validation against JSON schemas
- AWS WAF rules for SQL injection, XSS protection
- CloudWatch logging for all API calls

### 2. Orchestration Layer (AWS Step Functions)

**Purpose**: Coordinate microservices, handle retries, manage complex workflows

**Technology Stack**:
- AWS Step Functions (Standard Workflows)
- Amazon EventBridge (Event routing)
- AWS Lambda (Task execution)

**Workflow: Query Processing**

```json
{
  "Comment": "Bharat-Kiro Query Processing Workflow",
  "StartAt": "ValidateInput",
  "States": {
    "ValidateInput": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:REGION:ACCOUNT:function:ValidateInput",
      "Retry": [
        {
          "ErrorEquals": ["States.TaskFailed"],
          "IntervalSeconds": 2,
          "MaxAttempts": 3,
          "BackoffRate": 2.0
        }
      ],
      "Next": "CheckCache"
    },
    "CheckCache": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:REGION:ACCOUNT:function:CheckDynamoDBCache",
      "ResultPath": "$.cacheResult",
      "Next": "IsCacheHit"
    },
    "IsCacheHit": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.cacheResult.hit",
          "BooleanEquals": true,
          "Next": "ReturnCachedResponse"
        }
      ],
      "Default": "DeterminePersona"
    },
    "DeterminePersona": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:REGION:ACCOUNT:function:PersonaSwitcher",
      "ResultPath": "$.persona",
      "Next": "ParallelProcessing"
    },
    "ParallelProcessing": {
      "Type": "Parallel",
      "Branches": [
        {
          "StartAt": "GenerateAnalogy",
          "States": {
            "GenerateAnalogy": {
              "Type": "Task",
              "Resource": "arn:aws:lambda:REGION:ACCOUNT:function:LTAEngine",
              "End": true
            }
          }
        },
        {
          "StartAt": "GenerateAudio",
          "States": {
            "GenerateAudio": {
              "Type": "Task",
              "Resource": "arn:aws:lambda:REGION:ACCOUNT:function:PollyService",
              "End": true
            }
          }
        }
      ],
      "ResultPath": "$.processingResults",
      "Next": "CombineResults"
    },
    "CombineResults": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:REGION:ACCOUNT:function:CombineResults",
      "Next": "CacheResult"
    },
    "CacheResult": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:REGION:ACCOUNT:function:CacheToDynamoDB",
      "Next": "ReturnResponse"
    },
    "ReturnCachedResponse": {
      "Type": "Pass",
      "Next": "ReturnResponse"
    },
    "ReturnResponse": {
      "Type": "Succeed"
    }
  }
}
```

**Workflow Benefits**:
- Automatic retries with exponential backoff
- Parallel execution of independent tasks (analogy + audio generation)
- Visual workflow monitoring in Step Functions console
- Built-in error handling and compensation logic
- Execution history for debugging
- Cost-effective: Pay only for state transitions

**Workflow: Documentation Generation**

```
Start → Parse CDK Code → Parallel [
  Extract Architecture → Generate Diagram,
  Analyze IAM Policies → Amazon Q Audit,
  Extract Comments → Translate to Hindi/Marathi
] → Combine Sections → Generate README → Store in S3 → Return URL
```

### 3. Query Handler Service

**Purpose**: Process user queries and route to appropriate services

**Technology Stack**:
- AWS Lambda (Python 3.12, 512MB memory, 30s timeout)
- Amazon API Gateway (REST API)
- Amazon Cognito (Authentication)
- AWS X-Ray (Distributed tracing)

**Key Functions**:
- Parse user input (text/voice transcription from frontend)
- Detect language (Hindi/Marathi/Hinglish/English) using langdetect
- Route to LTA Engine or direct AWS documentation lookup
- Manage conversation context in DynamoDB
- Return formatted responses with metadata

**Data Flow**:
1. User submits query via React app (AWS Amplify)
2. API Gateway validates JWT token and authenticates user
3. Lambda function receives query with user context (user_id, persona, language_pref)
4. Language detection determines processing path
5. Query enriched with user persona (tech/non-tech) from DynamoDB profile
6. Step Functions workflow initiated for complex queries
7. Response formatted based on persona and cached in DynamoDB
8. Audio generated via Polly if user preference enabled
9. Response returned to user with cache headers

**Implementation**:
```python
import boto3
import json
from aws_xray_sdk.core import xray_recorder

@xray_recorder.capture('query_handler')
def lambda_handler(event, context):
    # Extract user context from JWT claims
    user_id = event['requestContext']['authorizer']['claims']['sub']
    persona = get_user_persona(user_id)
    
    # Parse query
    query = json.loads(event['body'])['query']
    language = detect_language(query)
    
    # Check semantic cache
    cache_key = generate_semantic_hash(query, persona, language)
    cached_response = check_cache(cache_key)
    
    if cached_response:
        return {
            'statusCode': 200,
            'body': json.dumps(cached_response),
            'headers': {'X-Cache': 'HIT'}
        }
    
    # Initiate Step Functions workflow
    sfn_client = boto3.client('stepfunctions')
    execution = sfn_client.start_execution(
        stateMachineArn=os.environ['STATE_MACHINE_ARN'],
        input=json.dumps({
            'user_id': user_id,
            'query': query,
            'persona': persona,
            'language': language
        })
    )
    
    # Return execution ARN for status polling
    return {
        'statusCode': 202,
        'body': json.dumps({
            'execution_arn': execution['executionArn'],
            'message': 'Processing query...'
        })
    }
```

### 4. Logic-to-Analogy (LTA) Engine

**Purpose**: Transform complex AWS concepts into culturally relevant analogies

**Technology Stack**:
- AWS Lambda (Python 3.12, 1GB memory)
- Amazon Bedrock (Claude 3.5 Sonnet)
- DynamoDB (Semantic cache with TTL)
- S3 (Cultural context database)
- AWS X-Ray (Performance tracing)

**Analogy Mapping Framework**:

#### Technical Concept → Cultural Analogy Examples

**AWS Lambda**:
- Hindi: "Lambda ek dhaba hai jo sirf tab khulta hai jab customer aata hai" (Lambda is a roadside eatery that only opens when customers arrive)
- Concept: Pay only when function executes, automatic scaling

**Amazon S3**:
- Marathi: "S3 ek motha ghodaam aahe jithun tumhi kaahi pan thevun shakta" (S3 is a large warehouse where you can store anything)
- Concept: Object storage, unlimited capacity, durability

**AWS IAM**:
- Hinglish: "IAM ek security guard hai jo check karta hai ki kaun kya kar sakta hai" (IAM is a security guard who checks who can do what)
- Concept: Access control, permissions, roles

**Amazon EC2**:
- Hindi: "EC2 ek kiraye ka computer hai jo aap apni zarurat ke hisaab se chun sakte hain" (EC2 is a rental computer you can choose based on your needs)
- Concept: Virtual servers, instance types, flexibility

**AWS CDK**:
- Marathi: "CDK ek blueprint aahe jyamule tumhi ek command madhe purn building ubhi karu shakta" (CDK is a blueprint that lets you construct an entire building with one command)
- Concept: Infrastructure as Code, automation

**Implementation Details**:

```python
# LTA Engine Core Logic with Semantic Caching
import boto3
import json
import hashlib
from aws_xray_sdk.core import xray_recorder

class LTAEngine:
    def __init__(self):
        self.bedrock_client = boto3.client('bedrock-runtime')
        self.dynamodb = boto3.resource('dynamodb')
        self.cache_table = self.dynamodb.Table('BharatKiro-AnalogyCache')
        self.cultural_context = self.load_cultural_database()
    
    @xray_recorder.capture('generate_analogy')
    def generate_analogy(self, aws_concept, language, user_level, persona):
        # Generate semantic cache key
        cache_key = self.generate_semantic_key(aws_concept, language, persona)
        
        # Check DynamoDB cache
        cached = self.get_from_cache(cache_key)
        if cached:
            xray_recorder.put_annotation('cache', 'hit')
            return cached
        
        # Retrieve cultural context
        context = self.get_cultural_context(language)
        
        # Build prompt for Bedrock Claude 3.5
        prompt = self.build_analogy_prompt(
            concept=aws_concept,
            language=language,
            context=context,
            user_level=user_level,
            persona=persona
        )
        
        # Generate analogy using Claude 3.5 Sonnet
        response = self.bedrock_client.invoke_model(
            modelId='anthropic.claude-3-5-sonnet-20241022-v2:0',
            body=json.dumps({
                'anthropic_version': 'bedrock-2023-05-31',
                'messages': [{'role': 'user', 'content': prompt}],
                'max_tokens': 800,
                'temperature': 0.7,
                'system': 'You are a patient teacher explaining AWS to Indian students using local cultural references.'
            })
        )
        
        analogy = json.loads(response['body'].read())['content'][0]['text']
        
        # Cache for future use with 7-day TTL
        self.cache_analogy(cache_key, analogy, ttl_days=7)
        
        xray_recorder.put_annotation('cache', 'miss')
        xray_recorder.put_metadata('analogy_length', len(analogy))
        
        return analogy
    
    def generate_semantic_key(self, concept, language, persona):
        # Create semantic hash for similar queries
        normalized = f"{concept.lower()}:{language}:{persona}"
        return hashlib.sha256(normalized.encode()).hexdigest()
    
    def get_from_cache(self, cache_key):
        try:
            response = self.cache_table.get_item(Key={'cache_key': cache_key})
            if 'Item' in response:
                return response['Item']['analogy']
        except Exception as e:
            print(f"Cache read error: {e}")
        return None
    
    def cache_analogy(self, cache_key, analogy, ttl_days=7):
        import time
        ttl = int(time.time()) + (ttl_days * 86400)
        
        self.cache_table.put_item(
            Item={
                'cache_key': cache_key,
                'analogy': analogy,
                'ttl': ttl,
                'created_at': int(time.time())
            }
        )
```

**Cultural Context Database Structure**:
- Festivals: Diwali, Holi, Ganesh Chaturthi
- Daily Life: Dabbawalas, Local trains, Street vendors
- Traditional Systems: Village panchayat, Joint family
- Popular Culture: Cricket, Bollywood, Regional stories

### 5. Persona Switcher (Adaptive Learning Engine)

**Purpose**: Adapt explanations based on user technical proficiency with scaffolding

**Technology Stack**:
- AWS Lambda (Python 3.12)
- Amazon DynamoDB (User profiles and learning progress)
- Amazon Bedrock (Claude 3.5 for adaptive content)

**Scaffolding Implementation**:
- Track user's learning journey from Non-Tech → Adaptive → Tech
- Gradually introduce technical terminology with inline definitions
- Adjust complexity based on user's query history and comprehension signals
- Provide "Learn More" links for deeper dives

**Modes**:

#### Non-Tech Mode
- Focus: Business value and outcomes
- Language: Simple, jargon-free
- Examples: Real-world use cases
- Depth: High-level overview
- Target: Founders, product managers, beginners

**Example Output**:
```
Q: What is AWS Lambda?
A (Non-Tech): Lambda aapko code run karne deta hai bina server ki chinta kiye. 
Jaise ek auto-rickshaw - aap sirf jitna chalate hain utna hi paisa dete hain. 
Server hamesha on nahi rehta, sirf jab zarurat ho tab chalta hai.
```

#### Tech Mode
- Focus: Technical specifications and implementation
- Language: Technical terms with explanations
- Examples: Code snippets, architecture patterns
- Depth: Detailed with best practices
- Target: Developers, engineers, technical students

**Example Output**:
```
Q: What is AWS Lambda?
A (Tech): Lambda ek serverless compute service hai jo event-driven architecture 
support karta hai. Aap apna code upload karte hain aur Lambda automatically 
scale karta hai. Billing per-millisecond hoti hai. Memory: 128MB-10GB, 
Timeout: max 15 minutes. Supported runtimes: Python, Node.js, Java, etc.
```

**Implementation**:
```python
class PersonaSwitcher:
    def __init__(self):
        self.dynamodb = boto3.resource('dynamodb')
        self.profile_table = self.dynamodb.Table('BharatKiro-UserProfiles')
    
    def determine_persona(self, user_id, query):
        # Get user's current persona and learning progress
        profile = self.get_user_profile(user_id)
        current_persona = profile.get('persona', 'non-tech')
        learning_stage = profile.get('learning_stage', 0)  # 0-100
        
        # Analyze query complexity
        query_complexity = self.analyze_query_complexity(query)
        
        # Adaptive logic: Gradually increase complexity
        if current_persona == 'non-tech' and learning_stage > 30:
            # User showing progress, introduce adaptive mode
            suggested_persona = 'adaptive'
        elif current_persona == 'adaptive' and learning_stage > 70:
            # User ready for full technical mode
            suggested_persona = 'tech'
        else:
            suggested_persona = current_persona
        
        # Override if query explicitly requires technical depth
        if query_complexity > 0.8:
            suggested_persona = 'tech'
        
        return suggested_persona
    
    def update_learning_progress(self, user_id, query_success, time_spent):
        # Update learning stage based on successful interactions
        profile = self.get_user_profile(user_id)
        current_stage = profile.get('learning_stage', 0)
        
        if query_success:
            new_stage = min(100, current_stage + 2)  # Increment by 2%
        else:
            new_stage = max(0, current_stage - 1)  # Slight decrement
        
        self.profile_table.update_item(
            Key={'user_id': user_id},
            UpdateExpression='SET learning_stage = :stage, last_updated = :time',
            ExpressionAttributeValues={
                ':stage': new_stage,
                ':time': int(time.time())
            }
        )
```

### 6. Auto-Kiro Documentation Generator

**Purpose**: Automatically generate comprehensive project documentation from code in seconds

**Technology Stack**:
- AWS Lambda (Python 3.12, 3GB memory, 5-minute timeout)
- Amazon Bedrock (Claude 3.5 for code analysis and documentation)
- AWS CDK Parser (Infrastructure analysis)
- Amazon S3 (Document storage)
- Amazon Q (Code auditing integration)

**Time Reduction**: From 4+ hours of manual writing to 30 seconds of automated generation

**Generated Artifacts**:

1. **README.md** (Vernacular + English)
   - Project overview
   - Setup instructions
   - Usage examples
   - Architecture explanation

2. **ARCHITECTURE.md**
   - System design diagrams
   - Component descriptions
   - Data flow explanations
   - AWS services used

3. **DEPLOYMENT.md**
   - Step-by-step deployment guide
   - Prerequisites checklist
   - CDK commands with explanations
   - Troubleshooting tips

4. **API_DOCS.md**
   - Endpoint documentation
   - Request/response examples
   - Error codes
   - Authentication details

**Processing Pipeline**:
```
Code Repository → CDK Parser → Bedrock Analysis → Template Engine → 
Multi-language Generation → S3 Storage → User Download
```

**Key Features**:
- Automatic architecture diagram generation
- Code comment extraction and translation
- Best practices recommendations
- Security checklist from Amazon Q

### 7. Audio Generation Service

**Purpose**: Convert text explanations to natural-sounding speech in Indian languages

**Technology Stack**:
- Amazon Polly (Neural TTS Engine)
- AWS Lambda (Audio processing)
- S3 (Audio file storage with lifecycle policies)
- CloudFront (Audio delivery CDN)

**Supported Voices**:
- Hindi: Aditi (Female, Neural)
- English (Indian): Kajal (Female, Neural)
- Bilingual support for Hinglish

**Features**:
- SSML support for natural pauses and emphasis
- Adjustable speech rate (0.8x - 1.2x)
- Audio caching for frequently requested content
- Streaming support for long explanations

**Implementation**:
```python
import boto3
import hashlib

def generate_audio(text, language, voice_id, user_id):
    polly_client = boto3.client('polly')
    s3_client = boto3.client('s3')
    
    # Generate cache key based on text content
    audio_hash = hashlib.md5(f"{text}:{voice_id}".encode()).hexdigest()
    s3_key = f'audio/{language}/{audio_hash}.mp3'
    
    # Check if audio already exists in S3
    try:
        s3_client.head_object(Bucket='bharat-kiro-audio', Key=s3_key)
        return f"https://cdn.bharatkiro.com/{s3_key}"  # Return cached audio
    except:
        pass  # Audio doesn't exist, generate new
    
    # Add SSML for natural speech with pauses
    ssml_text = f"""
    <speak>
        <prosody rate="medium" pitch="medium">
            {text}
        </prosody>
    </speak>
    """
    
    response = polly_client.synthesize_speech(
        Text=ssml_text,
        TextType='ssml',
        OutputFormat='mp3',
        VoiceId=voice_id,
        Engine='neural',
        LanguageCode=get_language_code(language)
    )
    
    # Store in S3 with caching headers and lifecycle policy
    s3_client.put_object(
        Bucket='bharat-kiro-audio',
        Key=s3_key,
        Body=response['AudioStream'].read(),
        ContentType='audio/mpeg',
        CacheControl='max-age=2592000',  # 30 days
        Metadata={
            'user_id': user_id,
            'language': language,
            'generated_at': str(int(time.time()))
        }
    )
    
    return f"https://cdn.bharatkiro.com/{s3_key}"
```

## Storage Strategy

### Data Architecture

**Amazon S3 (Blob Storage)**:
- **Purpose**: Store large binary objects (audio files, generated documents, CDK templates)
- **Bucket Structure**:
  ```
  bharat-kiro-audio/
    ├── hindi/
    │   └── {hash}.mp3
    ├── marathi/
    │   └── {hash}.mp3
    └── english/
        └── {hash}.mp3
  
  bharat-kiro-docs/
    ├── generated/
    │   ├── {user_id}/
    │   │   ├── {project_id}/
    │   │   │   ├── README.md
    │   │   │   ├── ARCHITECTURE.md
    │   │   │   └── DEPLOYMENT.md
    └── templates/
        ├── web-app-cdk/
        ├── api-gateway-cdk/
        └── data-pipeline-cdk/
  ```

- **Lifecycle Policies**:
  - Audio files: Transition to S3 Intelligent-Tiering after 30 days
  - Generated docs: Transition to S3 Glacier after 90 days
  - Temporary files: Delete after 7 days
  
- **Access Control**:
  - Private buckets with CloudFront OAI (Origin Access Identity)
  - Pre-signed URLs for user downloads (1-hour expiry)
  - Encryption at rest: S3-SSE (AES-256)
  - Versioning enabled for document buckets

**Amazon DynamoDB (Metadata & Caching)**:

**Table 1: UserProfiles**
```
Partition Key: user_id (String)
Attributes:
  - email (String)
  - persona (String): 'non-tech' | 'adaptive' | 'tech'
  - language_preference (String): 'hindi' | 'marathi' | 'hinglish'
  - learning_stage (Number): 0-100
  - total_queries (Number)
  - created_at (Number): Unix timestamp
  - last_active (Number): Unix timestamp
  
GSI: email-index (for login lookup)
Billing: On-Demand (PAY_PER_REQUEST)
```

**Table 2: AnalogyCache**
```
Partition Key: cache_key (String): SHA-256 hash
Attributes:
  - analogy (String): Generated analogy text
  - aws_concept (String): Original AWS service/concept
  - language (String)
  - persona (String)
  - hit_count (Number): Cache hit counter
  - created_at (Number)
  - ttl (Number): Auto-delete after 7 days
  
TTL Enabled: Yes (ttl attribute)
Billing: On-Demand
```

**Table 3: ProjectHistory**
```
Partition Key: user_id (String)
Sort Key: project_id (String): UUID
Attributes:
  - project_name (String)
  - cdk_code (String): Compressed CDK code
  - generated_docs (Map):
      - readme_url (String): S3 URL
      - architecture_url (String): S3 URL
      - deployment_url (String): S3 URL
  - aws_services_used (List): ['Lambda', 'S3', 'DynamoDB']
  - deployment_status (String): 'draft' | 'deployed' | 'failed'
  - created_at (Number)
  - updated_at (Number)
  
GSI: user_id-created_at-index (for chronological queries)
Billing: On-Demand
```

**Table 4: QueryHistory**
```
Partition Key: user_id (String)
Sort Key: timestamp (Number): Unix timestamp
Attributes:
  - query (String)
  - response (String): Compressed response
  - persona_used (String)
  - language (String)
  - response_time_ms (Number)
  - cache_hit (Boolean)
  - audio_generated (Boolean)
  
TTL: 30 days (for privacy)
Billing: On-Demand
```

**Storage Cost Optimization**:
- Semantic caching reduces Bedrock API calls by ~70%
- Audio file deduplication via content hashing
- DynamoDB on-demand pricing for unpredictable traffic
- S3 Intelligent-Tiering for automatic cost optimization
- CloudFront caching reduces S3 GET requests by ~80%

**Estimated Monthly Storage Costs** (1000 active users):
- S3 Storage (50GB): $1.15
- S3 Requests (1M GET, 100K PUT): $0.50
- DynamoDB Storage (10GB): $2.50
- DynamoDB Requests (5M reads, 1M writes): $3.75
- CloudFront Data Transfer (100GB): $8.50
- **Total Storage: ~$16.40/month**

## Code Auditing with Amazon Q

### Purpose
Ensure students learn AWS best practices from day one by providing real-time code reviews, security audits, and cost optimization recommendations.

### Integration Points

**1. Pre-Deployment Audit**
- Triggered automatically before CDK deployment
- Analyzes infrastructure code for security vulnerabilities
- Validates IAM policies for least privilege compliance
- Checks for cost optimization opportunities

**2. Real-Time Code Review**
- Integrated into Bharat-Kiro IDE/editor
- Provides inline suggestions as students write code
- Explains why certain patterns are recommended
- Links to AWS Well-Architected Framework documentation

**3. Post-Deployment Analysis**
- Reviews deployed resources for configuration issues
- Monitors for security group misconfigurations
- Identifies unused resources for cost savings
- Generates compliance reports

### Amazon Q Audit Workflow

```python
import boto3
import json

class AmazonQAuditor:
    def __init__(self):
        self.q_client = boto3.client('q')
        self.bedrock_client = boto3.client('bedrock-runtime')
    
    def audit_cdk_code(self, cdk_code, user_id, language='hindi'):
        """
        Audit CDK infrastructure code for best practices
        """
        # Step 1: Security Analysis
        security_issues = self.analyze_security(cdk_code)
        
        # Step 2: IAM Policy Review
        iam_issues = self.review_iam_policies(cdk_code)
        
        # Step 3: Cost Optimization
        cost_recommendations = self.analyze_costs(cdk_code)
        
        # Step 4: Well-Architected Review
        architecture_score = self.well_architected_review(cdk_code)
        
        # Step 5: Generate Vernacular Report
        audit_report = self.generate_report(
            security_issues,
            iam_issues,
            cost_recommendations,
            architecture_score,
            language
        )
        
        return audit_report
    
    def analyze_security(self, cdk_code):
        """
        Check for security vulnerabilities
        """
        issues = []
        
        # Check for public S3 buckets
        if 'publicReadAccess: true' in cdk_code:
            issues.append({
                'severity': 'CRITICAL',
                'issue': 'Public S3 bucket detected',
                'recommendation': 'Set publicReadAccess to false',
                'line': self.find_line_number(cdk_code, 'publicReadAccess')
            })
        
        # Check for overly permissive IAM policies
        if '"Action": "*"' in cdk_code or '"Resource": "*"' in cdk_code:
            issues.append({
                'severity': 'HIGH',
                'issue': 'Overly permissive IAM policy',
                'recommendation': 'Specify exact actions and resources',
                'line': self.find_line_number(cdk_code, '"Action": "*"')
            })
        
        # Check for missing encryption
        if 'new s3.Bucket' in cdk_code and 'encryption' not in cdk_code:
            issues.append({
                'severity': 'MEDIUM',
                'issue': 'S3 bucket without encryption',
                'recommendation': 'Add encryption: s3.BucketEncryption.S3_MANAGED',
                'line': self.find_line_number(cdk_code, 'new s3.Bucket')
            })
        
        return issues
    
    def review_iam_policies(self, cdk_code):
        """
        Validate IAM policies for least privilege
        """
        # Use Amazon Q to analyze IAM policies
        response = self.q_client.analyze_iam_policy(
            policyDocument=self.extract_iam_policies(cdk_code)
        )
        
        return response.get('findings', [])
    
    def analyze_costs(self, cdk_code):
        """
        Identify cost optimization opportunities
        """
        recommendations = []
        
        # Check for always-on resources
        if 'new ec2.Instance' in cdk_code:
            recommendations.append({
                'service': 'EC2',
                'issue': 'Always-on EC2 instance',
                'recommendation': 'Consider Lambda for event-driven workloads',
                'potential_savings': '~$50/month'
            })
        
        # Check for provisioned DynamoDB
        if 'billingMode: dynamodb.BillingMode.PROVISIONED' in cdk_code:
            recommendations.append({
                'service': 'DynamoDB',
                'issue': 'Provisioned capacity mode',
                'recommendation': 'Use on-demand for unpredictable traffic',
                'potential_savings': '~$20/month'
            })
        
        return recommendations
    
    def well_architected_review(self, cdk_code):
        """
        Score against AWS Well-Architected Framework
        """
        pillars = {
            'operational_excellence': 0,
            'security': 0,
            'reliability': 0,
            'performance_efficiency': 0,
            'cost_optimization': 0
        }
        
        # Security pillar
        if 'encryption' in cdk_code:
            pillars['security'] += 20
        if 'backup' in cdk_code or 'versioned: true' in cdk_code:
            pillars['reliability'] += 20
        if 'autoScaling' in cdk_code:
            pillars['performance_efficiency'] += 20
        
        return pillars
    
    def generate_report(self, security, iam, cost, architecture, language):
        """
        Generate audit report in vernacular language using Bedrock
        """
        report_data = {
            'security_issues': security,
            'iam_issues': iam,
            'cost_recommendations': cost,
            'architecture_score': architecture
        }
        
        # Use Bedrock to translate technical report to vernacular
        prompt = f"""
        Generate a friendly, educational code audit report in {language}.
        Explain each issue in simple terms with cultural analogies.
        
        Audit Results:
        {json.dumps(report_data, indent=2)}
        
        Format:
        1. Summary (overall score)
        2. Critical Issues (explain why they matter)
        3. Recommendations (step-by-step fixes)
        4. Learning Resources (links to AWS docs)
        """
        
        response = self.bedrock_client.invoke_model(
            modelId='anthropic.claude-3-5-sonnet-20241022-v2:0',
            body=json.dumps({
                'anthropic_version': 'bedrock-2023-05-31',
                'messages': [{'role': 'user', 'content': prompt}],
                'max_tokens': 2000,
                'temperature': 0.5
            })
        )
        
        report = json.loads(response['body'].read())['content'][0]['text']
        
        return {
            'report': report,
            'raw_data': report_data,
            'overall_score': self.calculate_overall_score(report_data)
        }
    
    def calculate_overall_score(self, report_data):
        """
        Calculate overall code quality score (0-100)
        """
        score = 100
        
        # Deduct points for issues
        for issue in report_data['security_issues']:
            if issue['severity'] == 'CRITICAL':
                score -= 20
            elif issue['severity'] == 'HIGH':
                score -= 10
            elif issue['severity'] == 'MEDIUM':
                score -= 5
        
        return max(0, score)
```

### Audit Report Example (Hindi)

```
🔒 Code Audit Report - Bharat-Kiro Mentor

Overall Score: 75/100 ✅

📊 Summary:
Aapka code mostly secure hai, lekin kuch improvements ki zarurat hai.
(Your code is mostly secure, but needs some improvements.)

🚨 Critical Issues (0):
Koi critical issue nahi mila! Bahut badhiya! 🎉

⚠️ High Priority (1):
1. IAM Policy bahut zyada permissive hai
   - Problem: Aapne "*" use kiya hai actions mein
   - Analogy: Yeh aise hai jaise aap ghar ki saari chabiyan kisi ko de dein
   - Fix: Sirf zaruri actions specify karein:
     ```typescript
     actions: ['s3:GetObject', 's3:PutObject']
     ```

💡 Recommendations (2):
1. S3 bucket mein encryption add karein
   - Cost: Free (S3-managed encryption)
   - Security boost: High
   
2. DynamoDB ko on-demand mode mein switch karein
   - Savings: ~₹1,500/month
   - Better for unpredictable traffic

📚 Learning Resources:
- IAM Best Practices (Hindi): [link]
- S3 Security Guide (Hindi): [link]
```

### Benefits of Amazon Q Integration

1. **Educational**: Students learn best practices while building
2. **Preventive**: Catch issues before deployment
3. **Cost-Aware**: Avoid expensive mistakes
4. **Security-First**: Build secure applications from day one
5. **Vernacular**: Explanations in Hindi/Marathi for better understanding

## Observability & Monitoring

### CloudWatch Integration

**Metrics Tracked**:
- Lambda invocation count, duration, errors (per function)
- API Gateway request count, latency, 4xx/5xx errors
- Step Functions execution count, success rate, duration
- Bedrock token usage, request count, latency
- DynamoDB read/write capacity, throttling events
- S3 bucket size, request count
- Polly character count, audio generation time

**Custom Metrics**:
```python
import boto3

cloudwatch = boto3.client('cloudwatch')

def log_custom_metric(metric_name, value, unit='Count'):
    cloudwatch.put_metric_data(
        Namespace='BharatKiro',
        MetricData=[
            {
                'MetricName': metric_name,
                'Value': value,
                'Unit': unit,
                'Dimensions': [
                    {'Name': 'Environment', 'Value': 'Production'},
                    {'Name': 'Service', 'Value': 'LTAEngine'}
                ]
            }
        ]
    )

# Usage
log_custom_metric('AnalogyGenerated', 1)
log_custom_metric('CacheHitRate', 0.75, 'Percent')
log_custom_metric('UserSatisfaction', 4.5, 'None')
```

**CloudWatch Dashboards**:

**Dashboard 1: System Health**
- API Gateway request rate (requests/minute)
- Lambda error rate (%)
- Step Functions success rate (%)
- Average response time (ms)
- Active users (real-time)

**Dashboard 2: Cost Tracking**
- Bedrock token usage (tokens/hour)
- Lambda invocations (count/hour)
- DynamoDB read/write units consumed
- S3 storage size (GB)
- Estimated daily cost ($)

**Dashboard 3: User Engagement**
- Queries per user (average)
- Persona distribution (Non-Tech vs Tech vs Adaptive)
- Language preference breakdown
- Documentation generation count
- Audio generation count

**Alarms**:
```yaml
Alarms:
  HighErrorRate:
    Metric: Lambda Errors
    Threshold: > 5% error rate
    Action: SNS notification to dev team
    
  HighLatency:
    Metric: API Gateway Latency
    Threshold: > 3 seconds (p99)
    Action: SNS notification + auto-scale Lambda
    
  CostOverrun:
    Metric: Estimated Daily Cost
    Threshold: > $50/day
    Action: SNS notification + email to admin
    
  DynamoDBThrottling:
    Metric: ThrottledRequests
    Threshold: > 10 requests/minute
    Action: Auto-switch to provisioned capacity
```

### AWS X-Ray Integration

**Distributed Tracing**:
- Trace requests from API Gateway → Lambda → Bedrock → DynamoDB
- Identify performance bottlenecks
- Visualize service dependencies
- Analyze error patterns

**X-Ray Implementation**:
```python
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch_all

# Patch all supported libraries
patch_all()

@xray_recorder.capture('process_query')
def process_query(query, user_id):
    # Add annotations for filtering
    xray_recorder.put_annotation('user_id', user_id)
    xray_recorder.put_annotation('query_type', 'analogy')
    
    # Add metadata for debugging
    xray_recorder.put_metadata('query_text', query)
    xray_recorder.put_metadata('timestamp', time.time())
    
    # Your processing logic
    result = generate_analogy(query)
    
    return result
```

**X-Ray Service Map**:
```
User → API Gateway → Step Functions → [
  Lambda (Query Handler) → DynamoDB (Cache Check),
  Lambda (LTA Engine) → Bedrock (Claude 3.5),
  Lambda (Polly Service) → Polly (Audio Gen) → S3
] → API Gateway → User
```

**Performance Insights**:
- Average trace duration: 2.5 seconds
- Bedrock call: 1.8 seconds (72% of total time)
- DynamoDB cache check: 50ms (2%)
- Lambda cold start: 300ms (12%)
- Polly audio generation: 400ms (16%)

**Optimization Opportunities** (identified via X-Ray):
1. Implement Lambda provisioned concurrency to reduce cold starts
2. Parallelize Bedrock and Polly calls using Step Functions
3. Increase DynamoDB cache TTL to reduce Bedrock calls
4. Use Bedrock streaming for faster perceived response time

### Logging Strategy

**Structured Logging**:
```python
import json
import logging
from datetime import datetime

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def log_event(event_type, user_id, details):
    log_entry = {
        'timestamp': datetime.utcnow().isoformat(),
        'event_type': event_type,
        'user_id': user_id,
        'details': details,
        'environment': os.environ.get('ENVIRONMENT', 'dev')
    }
    logger.info(json.dumps(log_entry))

# Usage
log_event('query_processed', 'user123', {
    'query': 'What is Lambda?',
    'persona': 'non-tech',
    'language': 'hindi',
    'response_time_ms': 2500,
    'cache_hit': False
})
```

**Log Retention**:
- Production logs: 30 days in CloudWatch
- Archive to S3 Glacier: After 30 days (for compliance)
- Development logs: 7 days

**Log Insights Queries**:
```sql
-- Average response time by persona
fields @timestamp, details.persona, details.response_time_ms
| filter event_type = "query_processed"
| stats avg(details.response_time_ms) by details.persona

-- Cache hit rate
fields @timestamp
| filter event_type = "query_processed"
| stats sum(details.cache_hit) / count(*) * 100 as cache_hit_rate

-- Most common queries
fields details.query
| filter event_type = "query_processed"
| stats count() by details.query
| sort count() desc
| limit 10
```

## Infrastructure as Code (AWS CDK)

### CDK Stack Structure

```typescript
// Main Stack with nested constructs
export class BharatKiroStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // ========== Frontend (AWS Amplify) ==========
    const amplifyApp = new amplify.App(this, 'BharatKiroApp', {
      sourceCodeProvider: new amplify.GitHubSourceCodeProvider({
        owner: 'your-org',
        repository: 'bharat-kiro-frontend',
        oauthToken: cdk.SecretValue.secretsManager('github-token')
      }),
      buildSpec: codebuild.BuildSpec.fromObjectToYaml({
        version: '1.0',
        frontend: {
          phases: {
            preBuild: { commands: ['npm install'] },
            build: { commands: ['npm run build'] }
          },
          artifacts: {
            baseDirectory: 'build',
            files: ['**/*']
          }
        }
      }),
      environmentVariables: {
        'REACT_APP_API_URL': cdk.Fn.importValue('ApiGatewayUrl'),
        'REACT_APP_USER_POOL_ID': cdk.Fn.importValue('UserPoolId')
      }
    });

    // Add main branch
    const mainBranch = amplifyApp.addBranch('main', {
      autoBuild: true,
      stage: 'PRODUCTION'
    });

    // ========== Authentication (Cognito) ==========
    const userPool = new cognito.UserPool(this, 'UserPool', {
      userPoolName: 'BharatKiroUsers',
      selfSignUpEnabled: true,
      signInAliases: { email: true },
      autoVerify: { email: true },
      passwordPolicy: {
        minLength: 8,
        requireLowercase: true,
        requireUppercase: true,
        requireDigits: true,
        requireSymbols: false
      },
      accountRecovery: cognito.AccountRecovery.EMAIL_ONLY,
      mfa: cognito.Mfa.OPTIONAL,
      mfaSecondFactor: { sms: true, otp: true }
    });

    const userPoolClient = userPool.addClient('WebClient', {
      authFlows: {
        userPassword: true,
        userSrp: true
      },
      oAuth: {
        flows: { authorizationCodeGrant: true },
        scopes: [cognito.OAuthScope.EMAIL, cognito.OAuthScope.OPENID, cognito.OAuthScope.PROFILE],
        callbackUrls: ['https://bharatkiro.com/callback'],
        logoutUrls: ['https://bharatkiro.com/logout']
      }
    });

    // ========== Storage Layer ==========
    // S3 Buckets
    const audioBucket = new s3.Bucket(this, 'AudioBucket', {
      bucketName: 'bharat-kiro-audio',
      encryption: s3.BucketEncryption.S3_MANAGED,
      blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL,
      versioned: false,
      lifecycleRules: [
        {
          id: 'TransitionToIntelligentTiering',
          transitions: [{
            storageClass: s3.StorageClass.INTELLIGENT_TIERING,
            transitionAfter: cdk.Duration.days(30)
          }]
        }
      ],
      cors: [{
        allowedMethods: [s3.HttpMethods.GET],
        allowedOrigins: ['https://bharatkiro.com'],
        allowedHeaders: ['*']
      }]
    });

    const docsBucket = new s3.Bucket(this, 'DocsBucket', {
      bucketName: 'bharat-kiro-docs',
      encryption: s3.BucketEncryption.S3_MANAGED,
      blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL,
      versioned: true,
      lifecycleRules: [
        {
          id: 'ArchiveOldDocs',
          transitions: [{
            storageClass: s3.StorageClass.GLACIER,
            transitionAfter: cdk.Duration.days(90)
          }]
        }
      ]
    });

    // DynamoDB Tables
    const userProfilesTable = new dynamodb.Table(this, 'UserProfiles', {
      tableName: 'BharatKiro-UserProfiles',
      partitionKey: { name: 'user_id', type: dynamodb.AttributeType.STRING },
      billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
      encryption: dynamodb.TableEncryption.AWS_MANAGED,
      pointInTimeRecovery: true,
      stream: dynamodb.StreamViewType.NEW_AND_OLD_IMAGES
    });

    userProfilesTable.addGlobalSecondaryIndex({
      indexName: 'email-index',
      partitionKey: { name: 'email', type: dynamodb.AttributeType.STRING },
      projectionType: dynamodb.ProjectionType.ALL
    });

    const analogyCacheTable = new dynamodb.Table(this, 'AnalogyCache', {
      tableName: 'BharatKiro-AnalogyCache',
      partitionKey: { name: 'cache_key', type: dynamodb.AttributeType.STRING },
      billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
      timeToLiveAttribute: 'ttl',
      encryption: dynamodb.TableEncryption.AWS_MANAGED
    });

    const projectHistoryTable = new dynamodb.Table(this, 'ProjectHistory', {
      tableName: 'BharatKiro-ProjectHistory',
      partitionKey: { name: 'user_id', type: dynamodb.AttributeType.STRING },
      sortKey: { name: 'project_id', type: dynamodb.AttributeType.STRING },
      billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
      encryption: dynamodb.TableEncryption.AWS_MANAGED
    });

    projectHistoryTable.addGlobalSecondaryIndex({
      indexName: 'user_id-created_at-index',
      partitionKey: { name: 'user_id', type: dynamodb.AttributeType.STRING },
      sortKey: { name: 'created_at', type: dynamodb.AttributeType.NUMBER },
      projectionType: dynamodb.ProjectionType.ALL
    });

    // ========== Lambda Functions ==========
    // Shared Lambda layer for common dependencies
    const sharedLayer = new lambda.LayerVersion(this, 'SharedLayer', {
      code: lambda.Code.fromAsset('lambda/layers/shared'),
      compatibleRuntimes: [lambda.Runtime.PYTHON_3_12],
      description: 'Shared dependencies: boto3, aws-xray-sdk, langdetect'
    });

    // Query Handler Lambda
    const queryHandler = new lambda.Function(this, 'QueryHandler', {
      functionName: 'BharatKiro-QueryHandler',
      runtime: lambda.Runtime.PYTHON_3_12,
      handler: 'index.handler',
      code: lambda.Code.fromAsset('lambda/query-handler'),
      timeout: cdk.Duration.seconds(30),
      memorySize: 512,
      layers: [sharedLayer],
      environment: {
        USER_PROFILES_TABLE: userProfilesTable.tableName,
        CACHE_TABLE: analogyCacheTable.tableName,
        STATE_MACHINE_ARN: '', // Will be set after Step Functions creation
        POWERTOOLS_SERVICE_NAME: 'query-handler'
      },
      tracing: lambda.Tracing.ACTIVE
    });

    // LTA Engine Lambda
    const ltaEngine = new lambda.Function(this, 'LTAEngine', {
      functionName: 'BharatKiro-LTAEngine',
      runtime: lambda.Runtime.PYTHON_3_12,
      handler: 'index.handler',
      code: lambda.Code.fromAsset('lambda/lta-engine'),
      timeout: cdk.Duration.seconds(60),
      memorySize: 1024,
      layers: [sharedLayer],
      environment: {
        CACHE_TABLE: analogyCacheTable.tableName,
        BEDROCK_MODEL_ID: 'anthropic.claude-3-5-sonnet-20241022-v2:0',
        CULTURAL_CONTEXT_BUCKET: docsBucket.bucketName
      },
      tracing: lambda.Tracing.ACTIVE
    });

    // Polly Service Lambda
    const pollyService = new lambda.Function(this, 'PollyService', {
      functionName: 'BharatKiro-PollyService',
      runtime: lambda.Runtime.PYTHON_3_12,
      handler: 'index.handler',
      code: lambda.Code.fromAsset('lambda/polly-service'),
      timeout: cdk.Duration.seconds(30),
      memorySize: 512,
      layers: [sharedLayer],
      environment: {
        AUDIO_BUCKET: audioBucket.bucketName,
        CLOUDFRONT_DOMAIN: '' // Will be set after CloudFront creation
      },
      tracing: lambda.Tracing.ACTIVE
    });

    // Auto-Doc Generator Lambda
    const autoDocGenerator = new lambda.Function(this, 'AutoDocGenerator', {
      functionName: 'BharatKiro-AutoDocGenerator',
      runtime: lambda.Runtime.PYTHON_3_12,
      handler: 'index.handler',
      code: lambda.Code.fromAsset('lambda/autodoc-generator'),
      timeout: cdk.Duration.minutes(5),
      memorySize: 3008,
      layers: [sharedLayer],
      environment: {
        DOCS_BUCKET: docsBucket.bucketName,
        BEDROCK_MODEL_ID: 'anthropic.claude-3-5-sonnet-20241022-v2:0',
        PROJECT_HISTORY_TABLE: projectHistoryTable.tableName
      },
      tracing: lambda.Tracing.ACTIVE
    });

    // Amazon Q Auditor Lambda
    const amazonQAuditor = new lambda.Function(this, 'AmazonQAuditor', {
      functionName: 'BharatKiro-AmazonQAuditor',
      runtime: lambda.Runtime.PYTHON_3_12,
      handler: 'index.handler',
      code: lambda.Code.fromAsset('lambda/q-auditor'),
      timeout: cdk.Duration.minutes(2),
      memorySize: 1024,
      layers: [sharedLayer],
      environment: {
        BEDROCK_MODEL_ID: 'anthropic.claude-3-5-sonnet-20241022-v2:0'
      },
      tracing: lambda.Tracing.ACTIVE
    });

    // Persona Switcher Lambda
    const personaSwitcher = new lambda.Function(this, 'PersonaSwitcher', {
      functionName: 'BharatKiro-PersonaSwitcher',
      runtime: lambda.Runtime.PYTHON_3_12,
      handler: 'index.handler',
      code: lambda.Code.fromAsset('lambda/persona-switcher'),
      timeout: cdk.Duration.seconds(10),
      memorySize: 256,
      layers: [sharedLayer],
      environment: {
        USER_PROFILES_TABLE: userProfilesTable.tableName
      },
      tracing: lambda.Tracing.ACTIVE
    });

    // ========== IAM Permissions ==========
    // Bedrock permissions
    const bedrockPolicy = new iam.PolicyStatement({
      effect: iam.Effect.ALLOW,
      actions: ['bedrock:InvokeModel', 'bedrock:InvokeModelWithResponseStream'],
      resources: ['arn:aws:bedrock:*:*:model/anthropic.claude-*']
    });

    ltaEngine.addToRolePolicy(bedrockPolicy);
    autoDocGenerator.addToRolePolicy(bedrockPolicy);
    amazonQAuditor.addToRolePolicy(bedrockPolicy);

    // Polly permissions
    pollyService.addToRolePolicy(new iam.PolicyStatement({
      effect: iam.Effect.ALLOW,
      actions: ['polly:SynthesizeSpeech'],
      resources: ['*']
    }));

    // DynamoDB permissions
    userProfilesTable.grantReadWriteData(queryHandler);
    userProfilesTable.grantReadWriteData(personaSwitcher);
    analogyCacheTable.grantReadWriteData(queryHandler);
    analogyCacheTable.grantReadWriteData(ltaEngine);
    projectHistoryTable.grantReadWriteData(autoDocGenerator);

    // S3 permissions
    audioBucket.grantReadWrite(pollyService);
    docsBucket.grantReadWrite(autoDocGenerator);
    docsBucket.grantRead(ltaEngine);

    // ========== Step Functions ==========
    const queryProcessingWorkflow = new sfn.StateMachine(this, 'QueryProcessingWorkflow', {
      stateMachineName: 'BharatKiro-QueryProcessing',
      definitionBody: sfn.DefinitionBody.fromFile('stepfunctions/query-processing.json'),
      timeout: cdk.Duration.minutes(5),
      tracingEnabled: true
    });

    // Grant Step Functions permission to invoke Lambdas
    queryHandler.grantInvoke(queryProcessingWorkflow);
    ltaEngine.grantInvoke(queryProcessingWorkflow);
    pollyService.grantInvoke(queryProcessingWorkflow);
    personaSwitcher.grantInvoke(queryProcessingWorkflow);

    // Update Query Handler with Step Functions ARN
    queryHandler.addEnvironment('STATE_MACHINE_ARN', queryProcessingWorkflow.stateMachineArn);
    queryProcessingWorkflow.grantStartExecution(queryHandler);

    // ========== API Gateway ==========
    const api = new apigateway.RestApi(this, 'BharatKiroAPI', {
      restApiName: 'Bharat-Kiro Mentor API',
      description: 'API for Bharat-Kiro Mentor platform',
      deployOptions: {
        stageName: 'prod',
        throttlingRateLimit: 100,
        throttlingBurstLimit: 200,
        tracingEnabled: true,
        loggingLevel: apigateway.MethodLoggingLevel.INFO,
        dataTraceEnabled: true,
        metricsEnabled: true
      },
      defaultCorsPreflightOptions: {
        allowOrigins: ['https://bharatkiro.com'],
        allowMethods: ['GET', 'POST', 'PUT', 'OPTIONS'],
        allowHeaders: ['Content-Type', 'Authorization'],
        allowCredentials: true
      }
    });

    // Cognito Authorizer
    const authorizer = new apigateway.CognitoUserPoolsAuthorizer(this, 'CognitoAuthorizer', {
      cognitoUserPools: [userPool],
      authorizerName: 'BharatKiroAuthorizer',
      identitySource: 'method.request.header.Authorization'
    });

    // API Resources
    const queryResource = api.root.addResource('query');
    queryResource.addMethod('POST', new apigateway.LambdaIntegration(queryHandler), {
      authorizer,
      authorizationType: apigateway.AuthorizationType.COGNITO
    });

    const personaResource = api.root.addResource('persona');
    personaResource.addMethod('PUT', new apigateway.LambdaIntegration(personaSwitcher), {
      authorizer,
      authorizationType: apigateway.AuthorizationType.COGNITO
    });

    const documentationResource = api.root.addResource('documentation');
    documentationResource.addMethod('POST', new apigateway.LambdaIntegration(autoDocGenerator), {
      authorizer,
      authorizationType: apigateway.AuthorizationType.COGNITO
    });

    const auditResource = api.root.addResource('audit');
    auditResource.addMethod('POST', new apigateway.LambdaIntegration(amazonQAuditor), {
      authorizer,
      authorizationType: apigateway.AuthorizationType.COGNITO
    });

    // ========== CloudFront Distribution ==========
    const distribution = new cloudfront.Distribution(this, 'Distribution', {
      defaultBehavior: {
        origin: new origins.S3Origin(audioBucket),
        viewerProtocolPolicy: cloudfront.ViewerProtocolPolicy.REDIRECT_TO_HTTPS,
        cachePolicy: cloudfront.CachePolicy.CACHING_OPTIMIZED,
        allowedMethods: cloudfront.AllowedMethods.ALLOW_GET_HEAD,
        compress: true
      },
      additionalBehaviors: {
        '/docs/*': {
          origin: new origins.S3Origin(docsBucket),
          viewerProtocolPolicy: cloudfront.ViewerProtocolPolicy.REDIRECT_TO_HTTPS,
          cachePolicy: cloudfront.CachePolicy.CACHING_OPTIMIZED
        }
      },
      priceClass: cloudfront.PriceClass.PRICE_CLASS_200,
      enableLogging: true,
      logBucket: new s3.Bucket(this, 'CloudFrontLogsBucket')
    });

    // Update Polly Service with CloudFront domain
    pollyService.addEnvironment('CLOUDFRONT_DOMAIN', distribution.distributionDomainName);

    // ========== CloudWatch Dashboard ==========
    const dashboard = new cloudwatch.Dashboard(this, 'BharatKiroDashboard', {
      dashboardName: 'BharatKiro-Metrics'
    });

    dashboard.addWidgets(
      new cloudwatch.GraphWidget({
        title: 'API Gateway Requests',
        left: [api.metricCount(), api.metricClientError(), api.metricServerError()]
      }),
      new cloudwatch.GraphWidget({
        title: 'Lambda Performance',
        left: [
          queryHandler.metricInvocations(),
          ltaEngine.metricInvocations(),
          pollyService.metricInvocations()
        ]
      }),
      new cloudwatch.GraphWidget({
        title: 'Lambda Errors',
        left: [
          queryHandler.metricErrors(),
          ltaEngine.metricErrors(),
          pollyService.metricErrors()
        ]
      }),
      new cloudwatch.SingleValueWidget({
        title: 'Active Users (24h)',
        metrics: [userProfilesTable.metricConsumedReadCapacityUnits()]
      })
    );

    // ========== Alarms ==========
    new cloudwatch.Alarm(this, 'HighErrorRateAlarm', {
      metric: queryHandler.metricErrors(),
      threshold: 10,
      evaluationPeriods: 2,
      alarmDescription: 'Alert when Lambda error rate is high',
      treatMissingData: cloudwatch.TreatMissingData.NOT_BREACHING
    });

    new cloudwatch.Alarm(this, 'HighLatencyAlarm', {
      metric: api.metricLatency(),
      threshold: 3000, // 3 seconds
      evaluationPeriods: 3,
      statistic: 'p99',
      alarmDescription: 'Alert when API latency exceeds 3 seconds'
    });

    // ========== Outputs ==========
    new cdk.CfnOutput(this, 'ApiUrl', {
      value: api.url,
      exportName: 'ApiGatewayUrl'
    });

    new cdk.CfnOutput(this, 'UserPoolIdOutput', {
      value: userPool.userPoolId,
      exportName: 'UserPoolId'
    });

    new cdk.CfnOutput(this, 'UserPoolClientIdOutput', {
      value: userPoolClient.userPoolClientId,
      exportName: 'UserPoolClientId'
    });

    new cdk.CfnOutput(this, 'CloudFrontDomain', {
      value: distribution.distributionDomainName,
      exportName: 'CloudFrontDomain'
    });

    new cdk.CfnOutput(this, 'AmplifyAppUrl', {
      value: `https://${mainBranch.branchName}.${amplifyApp.defaultDomain}`,
      exportName: 'AmplifyAppUrl'
    });
  }
}
```

### Deployment Commands

```bash
# Prerequisites
npm install -g aws-cdk
pip install aws-cdk-lib constructs

# Install project dependencies
cd bharat-kiro-infrastructure
npm install

# Configure AWS credentials
aws configure

# Bootstrap CDK (first time only, per region/account)
cdk bootstrap aws://ACCOUNT-ID/ap-south-1

# Synthesize CloudFormation template (validate)
cdk synth

# View changes before deployment
cdk diff

# Deploy all stacks
cdk deploy --all --require-approval never

# Deploy specific stack
cdk deploy BharatKiroStack

# View stack outputs
cdk outputs

# Destroy all resources (cleanup)
cdk destroy --all
```

### CI/CD Pipeline (GitHub Actions)

```yaml
name: Deploy Bharat-Kiro Infrastructure

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.12'
      
      - name: Install dependencies
        run: |
          npm install
          pip install -r requirements.txt
      
      - name: Run unit tests
        run: npm test
      
      - name: Run security scan
        run: |
          npm install -g snyk
          snyk test
  
  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-south-1
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install CDK
        run: npm install -g aws-cdk
      
      - name: Install dependencies
        run: npm install
      
      - name: Run Amazon Q security review
        run: python scripts/security_review.py
      
      - name: CDK Synth
        run: cdk synth
      
      - name: CDK Deploy
        run: cdk deploy --all --require-approval never
      
      - name: Run integration tests
        run: npm run test:integration
      
      - name: Notify on success
        if: success()
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'Bharat-Kiro deployed successfully!'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

## Security Architecture

### Authentication & Authorization

**Amazon Cognito User Pools**:
- Email/password authentication
- Social login (Google, GitHub)
- MFA support for sensitive operations
- JWT token-based API access

**IAM Roles & Policies**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "bedrock:InvokeModel"
      ],
      "Resource": "arn:aws:bedrock:*:*:model/anthropic.claude-*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "polly:SynthesizeSpeech"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem"
      ],
      "Resource": "arn:aws:dynamodb:*:*:table/BharatKiro-*"
    }
  ]
}
```

### Data Protection

**Encryption**:
- At Rest: S3 SSE, DynamoDB encryption
- In Transit: TLS 1.2+ for all API calls
- Secrets: AWS Secrets Manager for API keys

**Data Privacy**:
- No storage of user queries beyond cache TTL (24 hours)
- Anonymized analytics only
- GDPR-compliant data handling
- User data deletion on request

### Amazon Q Security Integration

**Automated Security Reviews**:
- IAM policy analysis for least privilege
- Code vulnerability scanning
- Infrastructure security best practices
- Compliance checks (AWS Well-Architected)

**Implementation**:
```python
# Pre-deployment security check
def run_security_review():
    q_client = boto3.client('q')
    
    # Analyze IAM policies
    iam_review = q_client.analyze_iam_policies(
        stackName='BharatKiroStack'
    )
    
    # Check for security issues
    if iam_review['criticalIssues'] > 0:
        raise Exception("Critical security issues found")
    
    # Review Lambda function permissions
    lambda_review = q_client.review_lambda_permissions(
        functions=['QueryHandler', 'LTAEngine', 'AutoDocGenerator']
    )
    
    return {
        'iam_score': iam_review['score'],
        'lambda_score': lambda_review['score'],
        'recommendations': iam_review['recommendations']
    }
```

## Performance Optimization

### Caching Strategy

**Multi-Layer Caching**:
1. DynamoDB: Query results (24-hour TTL)
2. CloudFront: Static assets and audio files
3. Lambda: In-memory caching for cultural context
4. API Gateway: Response caching (5 minutes)

**Cache Invalidation**:
- Automatic on content updates
- Manual purge via admin API
- Version-based cache keys

### Cost Optimization

**Free Tier Utilization**:
- Lambda: 1M requests/month free
- S3: 5GB storage free
- DynamoDB: 25GB storage free
- CloudFront: 1TB transfer free (12 months)

**Estimated Monthly Cost** (10,000 users, 5 queries/user/day):
- AWS Amplify (hosting): $15
- Lambda (50M invocations): $20
- Step Functions (10M state transitions): $25
- Bedrock (Claude 3.5, 100M tokens): $80
- Polly (10M characters): $40
- S3 (100GB storage + requests): $15
- DynamoDB (on-demand): $30
- API Gateway (50M requests): $35
- CloudFront (500GB transfer): $42
- CloudWatch + X-Ray: $20
- Cognito (10K MAU): $5
- **Total: ~$327/month**
- **Per User: $0.033/month**

**Free Tier Benefits** (first 12 months):
- Lambda: 1M requests/month free
- S3: 5GB storage, 20K GET, 2K PUT free
- DynamoDB: 25GB storage, 25 RCU/WCU free
- CloudFront: 1TB transfer free
- API Gateway: 1M requests free
- **Estimated savings: ~$100/month**

## Monitoring & Observability

### CloudWatch Integration

**Metrics**:
- Lambda invocation count and duration
- API Gateway request count and latency
- Bedrock token usage
- Error rates by service

**Alarms**:
- Lambda error rate > 5%
- API latency > 3 seconds
- DynamoDB throttling events
- Cost exceeds budget threshold

**Dashboards**:
- Real-time user activity
- Service health status
- Cost tracking
- User satisfaction metrics

### Logging Strategy

```python
import logging
import json

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def log_query(user_id, query, language, response_time):
    logger.info(json.dumps({
        'event': 'query_processed',
        'user_id': user_id,
        'language': language,
        'response_time_ms': response_time,
        'timestamp': datetime.utcnow().isoformat()
    }))
```

## Deployment Strategy

### CI/CD Pipeline

```yaml
# GitHub Actions workflow
name: Deploy Bharat-Kiro

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-south-1
      
      - name: Run security review
        run: python scripts/security_review.py
      
      - name: Deploy CDK stack
        run: |
          npm install
          cdk deploy --require-approval never
```

### Rollback Strategy

- Blue/Green deployment for Lambda functions
- CloudFormation stack versioning
- Automated rollback on alarm triggers
- Database migration reversibility

## Future Enhancements

### Phase 2 Features
- Support for Tamil, Telugu, Bengali languages
- Real-time collaborative coding
- Mobile native apps (iOS/Android)
- Integration with AWS Amplify for frontend
- Advanced analytics with Amazon QuickSight

### Scalability Roadmap
- Multi-region deployment
- GraphQL API with AWS AppSync
- Real-time updates via WebSockets
- Machine learning model fine-tuning
- Community-contributed analogies

## Conclusion

Bharat-Kiro Mentor leverages AWS's serverless, event-driven architecture to create a scalable, cost-effective, and culturally aware learning platform. By combining:

- **AWS Step Functions** for reliable orchestration and automatic retries
- **Amazon Bedrock (Claude 3.5)** for intelligent content generation and analogies
- **Amazon Polly Neural** for natural-sounding vernacular audio
- **Amazon Q** for real-time code auditing and best practices
- **AWS Amplify** for seamless frontend deployment
- **DynamoDB + S3** for efficient storage with semantic caching
- **CloudWatch + X-Ray** for comprehensive observability

We enable rural Indian students to overcome language barriers, learn through scaffolding-based education, and build on AWS 10x faster. The platform reduces documentation time from hours to seconds, provides persona-adaptive learning experiences, and ensures students learn security best practices from day one through automated code auditing.

The architecture is designed to scale from hundreds to millions of users while maintaining sub-3-second response times and staying within budget constraints through intelligent caching, serverless compute, and cost optimization strategies.


