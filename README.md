# NLP Text Processing API

A full-stack NLP system built with FastAPI, PostgreSQL, and Streamlit. It can clean and process text, analyse sentiment, and is secured with JWT authentication. The whole app runs in Docker.

---

## What This Project Does

You send text to the API and it can:
- Clean it (remove punctuation, lowercase)
- Tokenize it (split into words)
- Preprocess it (remove stopwords, lemmatize)
- Predict whether the sentiment is **positive** or **negative**

All requests are protected — you must register and log in to use the API. Every processed text is saved to a PostgreSQL database with a timestamp.

---

## Project Structure

```
nlp-api/
├── main.py                  # FastAPI app and all endpoints
├── auth.py                  # JWT authentication logic
├── database.py              # PostgreSQL connection and models
├── requirements.txt         # Python dependencies
├── Dockerfile               # Docker config for the API
├── docker-compose.yml       # Runs API + database together
├── model/
│   ├── train.py             # Script to train the sentiment model
│   ├── sentiment_model.pkl  # Trained model (generated after training)
│   └── vectorizer.pkl       # TF-IDF vectorizer (generated after training)
└── frontend/
    └── app.py               # Streamlit UI
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| API framework | FastAPI |
| Database | PostgreSQL |
| ORM | SQLAlchemy |
| Authentication | JWT (python-jose + passlib) |
| NLP | NLTK |
| ML Model | Scikit-learn (Logistic Regression + TF-IDF) |
| Frontend | Streamlit |
| Containerisation | Docker + Docker Compose |

---

## How to Run It

### Option 1 — With Docker (recommended)

Make sure Docker Desktop is running, then:

```bash
git clone https://github.com/aakwadwo/nlp-api.git
cd nlp-api
```

Train the model first (you only need to do this once):

```bash
pip install scikit-learn nltk pandas numpy
python model/train.py
```

Then start everything:

```bash
docker-compose up --build
```

The API will be running at `http://127.0.0.1:8002/docs`

---

### Option 2 — Without Docker (local)

**Step 1 — Install dependencies**

```bash
pip install -r requirements.txt
```

**Step 2 — Set up PostgreSQL**

Make sure PostgreSQL is running, then create the database:

```bash
psql postgres
```

```sql
CREATE DATABASE nlpdb;
\q
```

**Step 3 — Train the model**

```bash
python model/train.py
```

**Step 4 — Start the API**

```bash
uvicorn main:app --reload
```

API runs at `http://127.0.0.1:8000/docs`

**Step 5 — Start the frontend (in a separate terminal)**

```bash
streamlit run frontend/app.py
```

Frontend runs at `http://localhost:8501`

---

## API Endpoints

| Method | Endpoint | Protected | What it does |
|---|---|---|---|
| GET | `/` | No | Health check |
| POST | `/register` | No | Create a new account |
| POST | `/login` | No | Log in and get a token |
| POST | `/clean` | Yes | Lowercase and remove punctuation |
| POST | `/tokenize` | Yes | Split text into tokens |
| POST | `/preprocess` | Yes | Full preprocessing pipeline |
| POST | `/predict` | Yes | Predict sentiment |
| GET | `/history` | Yes | View all past requests |

All protected endpoints require a Bearer token in the header. You get this token by logging in.

---

## Example Usage

**Register:**
```json
POST /register
{ "username": "kwadwo", "password": "test1234" }
```

**Login:**
```json
POST /login
{ "username": "kwadwo", "password": "test1234" }
```

**Predict sentiment:**
```json
POST /predict
{ "text": "This course is amazing and very helpful" }
```

**Response:**
```json
{
  "original": "This course is amazing and very helpful",
  "processed": "course amazing helpful",
  "sentiment": "positive",
  "confidence": "58.1%"
}
```

---

## The NLP Pipeline

When you send text to `/preprocess` or `/predict`, it goes through these steps:

1. **Lowercase** — `"AI is Great"` → `"ai is great"`
2. **Remove punctuation** — `"ai is great!"` → `"ai is great"`
3. **Tokenize** — `["ai", "is", "great"]`
4. **Remove stopwords** — removes words like "is", "the", "a" → `["ai", "great"]`
5. **Lemmatize** — reduces words to root form e.g. "running" → "run"
6. **TF-IDF vectorize** — converts tokens to numbers the model understands
7. **Predict** — Logistic Regression model outputs positive or negative

---

## The Sentiment Model

- **Algorithm:** Logistic Regression
- **Features:** TF-IDF with bigrams (1-2 word combinations)
- **Training data:** 30 labelled sentences (15 positive, 15 negative)
- **Accuracy:** ~67% on the small test set

> Note: Accuracy is modest because of the small training dataset. For production, you would replace this with a large dataset like IMDB Reviews (50,000 samples) and expect 90%+ accuracy.

---

## Environment Variables

The app uses one environment variable:

| Variable | Description | Default |
|---|---|---|
| `DATABASE_URL` | PostgreSQL connection string | `postgresql+psycopg2://kwadwo:@localhost/nlpdb` |

In Docker, this is set automatically via `docker-compose.yml`.

---

## Built By

Kwadwo — as part of an NLP Systems Engineering project covering API development, database integration, authentication, ML model deployment, containerisation, and full-stack design.
