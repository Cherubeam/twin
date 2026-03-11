# Twin

AI-powered digital twin — a serverless chat application where visitors can talk to an AI representation of you, deployed on AWS.

## Architecture

```
User Browser
    | HTTPS
CloudFront (CDN)
    |
S3 Static Website (Frontend)
    | HTTPS API Calls
API Gateway
    |
Lambda Function (Backend)
    |
    |-- AWS Bedrock (AI responses)
    +-- S3 Memory Bucket (conversation persistence)
```

## Tech Stack

**Backend:** Python 3.13, FastAPI, AWS Bedrock (Amazon Nova), Lambda (via Mangum), boto3

**Frontend:** Next.js 16, React 19, Tailwind CSS 4, TypeScript, Lucide React

**Infrastructure:** AWS Lambda, API Gateway, S3, CloudFront

## Project Structure

```
twin/
├── backend/
│   ├── data/                # Profile data (facts.json, style.txt, summary.txt, linkedin.pdf)
│   ├── server.py            # FastAPI app — API endpoints, Bedrock integration
│   ├── context.py           # System prompt builder
│   ├── resources.py         # Loads profile data files
│   ├── lambda_handler.py    # Mangum adapter for AWS Lambda
│   ├── deploy.py            # Lambda deployment ZIP builder (Docker-based)
│   ├── requirements.txt     # Python dependencies
│   └── pyproject.toml       # Project metadata
├── frontend/
│   ├── app/page.tsx         # Main page
│   ├── components/twin.tsx  # Chat UI component
│   ├── next.config.ts       # Static export configuration
│   └── package.json         # Node dependencies
└── docs/
    └── architecture.md      # Architecture diagram & cost info
```

## Getting Started

### Prerequisites

- Python 3.13+
- Node.js 18+
- AWS credentials configured (`aws configure`) with Bedrock access
- [uv](https://docs.astral.sh/uv/) (recommended) or pip

### Backend Setup

```bash
cd backend

# Create virtual environment and install dependencies
uv sync
# or: pip install -r requirements.txt

# Configure environment variables
cp .env .env.local  # Edit with your values (see Environment Variables below)

# Run the development server
uv run uvicorn server:app --reload --port 8000
```

The API will be available at `http://localhost:8000`.

### Frontend Setup

```bash
cd frontend

# Install dependencies
npm install

# Run the development server
npm run dev
```

The frontend will be available at `http://localhost:3000`.

## Environment Variables

Create a `.env` file in the `backend/` directory:

| Variable | Description | Default |
|---|---|---|
| `CORS_ORIGINS` | Comma-separated allowed origins | `http://localhost:3000` |
| `DEFAULT_AWS_REGION` | AWS region for Bedrock | `eu-central-1` |
| `BEDROCK_MODEL_ID` | Bedrock model to use | `eu.amazon.nova-lite-v1:0` |
| `USE_S3` | Use S3 for conversation storage | `false` |
| `S3_BUCKET` | S3 bucket for conversation history | (empty) |
| `MEMORY_DIR` | Local directory for conversation files | `../memory` |

**Available Bedrock models:**
- `amazon.nova-micro-v1:0` — fastest, cheapest
- `amazon.nova-lite-v1:0` — balanced (default)
- `amazon.nova-pro-v1:0` — most capable, higher cost

> Note: Depending on your region, you may need the `us.` or `eu.` prefix on model IDs.

## API Endpoints

| Method | Path | Description |
|---|---|---|
| `GET` | `/` | API info (model, storage mode) |
| `GET` | `/health` | Health check |
| `POST` | `/chat` | Send a message — accepts `{ message, session_id? }`, returns `{ response, session_id }` |
| `GET` | `/conversation/{session_id}` | Retrieve conversation history |

## Deployment

### Backend (Lambda)

1. Build the deployment package (requires Docker):
   ```bash
   cd backend
   python deploy.py
   ```
2. Upload `lambda-deployment.zip` to AWS Lambda
3. Set the handler to `lambda_handler.handler`
4. Configure environment variables (`USE_S3=true`, `S3_BUCKET`, etc.)
5. Set up API Gateway to route to the Lambda function

### Frontend (S3 + CloudFront)

1. Build the static export:
   ```bash
   cd frontend
   npm run build
   ```
2. Upload the `out/` directory to an S3 bucket
3. Create a CloudFront distribution pointing to the S3 bucket

## Customization

**Personality & knowledge** — Edit files in `backend/data/`:
- `facts.json` — basic profile information (name, role, location, etc.)
- `summary.txt` — free-form notes about the person
- `style.txt` — communication style guidelines
- `linkedin.pdf` — LinkedIn profile export for career context

**System prompt** — Modify `backend/context.py` to change how the AI presents itself.

**AI model** — Set `BEDROCK_MODEL_ID` in your `.env` to switch between Nova models.

**Frontend API URL** — Update the fetch URL in `frontend/components/twin.tsx` to point to your API Gateway endpoint.
