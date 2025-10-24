/automated-home-grocecery-ordering-system\
A lightweight, extensible AI agent that helps users browse, add, and reorder groceries, plan shopping lists, and manage deliveries — built for demos, prototypes, and production-ready integrations.

Table of contents

Project Overview

Features

Quick Start

Architecture

Getting Started — Local Development

Usage Examples

Configuration

API & Endpoints

Testing

Deployment

Data, Privacy & Security

Extending the Agent

Troubleshooting

Roadmap

Contributing & License

Project overview

This repository contains an AI-driven conversational agent (chatbot) that supports common grocery-order tasks:

Search and suggest items

Build and modify shopping lists / carts

Place orders with simple checkout flows (mock or integrated)

Handle delivery scheduling and order tracking

Personalized recommendations (recent purchases, favorites)

Small admin tools for inventory and catalogs

It's intentionally modular so you can swap components (NLP model, DB, payment gateway) and integrate it into web, mobile, or voice channels.

Features

Natural-language shopping: “Add 2 avocados and a bottle of milk to my cart”

Multi-turn dialog with context and cart state preservation

Intent classification + slot-filling + entity extraction

Shopping list import/export (CSV/JSON)

Rules engine for inventory and substitutions

Mock payment & delivery simulator (production-ready hooks)

REST API + CLI + minimal web UI example

Config-driven connectors for catalogs and stores

Quick start
# clone
git clone https://github.com/your-org/grocery-ai-agent.git
cd grocery-ai-agent

# create venv (or use Docker)
python -m venv .venv
source .venv/bin/activate

# install
pip install -r requirements.txt

# copy example env and update keys
cp .env.example .env

# run locally
python run_agent.py


Then open the example web UI at http://localhost:8080 or use the CLI:

python cli.py --prompt "Add two large bananas and a dozen eggs to my cart"

Architecture

High-level modules:

agent/ — core dialog manager, intent/router, and conversation state

nlp/ — intent classification, entity extraction, slot filling (pluggable)

catalog/ — product catalog connector, search, substitution rules

cart/ — cart state, pricing, discounts, coupons

order/ — checkout flow, payments (mock/real), delivery scheduling

api/ — REST API (FastAPI)

ui/ — minimal web UI demo

tests/ — unit and integration tests

scripts/ — helper scripts for data seeding, DB migrations

Diagram (simplified):

User → UI / Voice → API → Agent → NLP / Catalog / Cart → Order → (Payment / Delivery)

Getting started — local development
Prereqs

Python 3.10+

Node.js (optional, for web UI)

Docker & docker-compose (optional)

Install
pip install -r requirements.txt

Environment variables

Create .env from .env.example. Key variables:

# .env
FLASK_ENV=development
OPENAI_API_KEY=sk-xxxx            # if using OpenAI or another model
CATALOG_DB_URL=sqlite:///data.db
SECRET_KEY=change_me
PAYMENT_MODE=mock                 # or 'stripe'

Seed sample catalog
python scripts/seed_catalog.py --file sample_catalog.csv

Run server
uvicorn api.main:app --reload
# OR
python run_agent.py   # starts API + demo UI

Usage examples
CLI
python cli.py --prompt "I need milk, bread, and coffee. Prefer almond milk."


Expected output (example):

Bot: Added 1x Almond Milk (1L), 1x Whole Wheat Bread, 1x Ground Coffee to your cart.
Bot: Would you like delivery today or tomorrow?

REST API

POST /v1/chat

POST /v1/chat
Content-Type: application/json
Authorization: Bearer <API_KEY>

{
  "session_id": "user-123",
  "message": "Add 2 avocados and a bottle of orange juice to my cart"
}


Response:

{
  "reply": "Added 2 Avocados and 1 Orange Juice (1L) to your cart. Anything else?",
  "cart": { "items": [...], "total": 9.48 }
}

Web UI

The demo UI connects to /v1/chat. It shows the conversation and cart in real time.

Configuration
Intent / NLU

config/intents.yml — sample:

intents:
  - name: add_item
    examples:
      - add {quantity} {item}
      - put {quantity} {item} in my cart
  - name: remove_item
    examples:
      - remove {item}

Catalog sample (CSV)

sample_catalog.csv columns:

sku,name,brand,category,price,unit,stock,substitutions
0001,Whole Milk,Generic,Dairy,2.49,1L,120,"0002;0003"

API & Endpoints (short)

POST /v1/chat — send message, receive agent reply (session-based)

GET /v1/cart/{session_id} — get cart for session

POST /v1/checkout — start checkout (mock/stripe)

GET /v1/orders/{order_id} — order status

POST /v1/catalog/search — search products

Full OpenAPI spec is in api/openapi.json.

Testing

Run unit tests:

pytest -q


Integration tests (requires seeded DB):

pytest tests/integration -q


Suggested test cases:

Add/remove items (single & multi-turn)

Quantity normalization (e.g., "half dozen" → 6)

Substitution when item out of stock

Delivery scheduling edge cases

Deployment
Docker (example)

Dockerfile provided. Build and run:

docker build -t grocery-agent:latest .
docker run -p 8080:8080 --env-file .env grocery-agent:latest

docker-compose (example)

docker-compose.yml includes API + DB + optional Redis for session store.

Production tips

Use Redis for conversation/session state

Use persistent DB (Postgres) for catalogs and order history

Set PAYMENT_MODE=stripe and configure keys

Use HTTPS and rotate secrets via a secret manager

Data, privacy & security

Customer payment handling is simulated by default. For real payments, integrate a PCI-compliant provider (Stripe, Adyen).

Personally identifiable information (PII) should be stored only when necessary; encrypt at rest and in transit.

Provide GDPR/CCPA hooks: data export & deletion endpoints (suggested in api/privacy.py).

Extending the agent

Swap NLP backend: the nlp module implements an abstract NLPProvider — add providers for Rasa, spaCy, or cloud LLMs.

Add new channels: implement connectors for WhatsApp, Twilio SMS, Alexa or Google Assistant in connectors/.

Add recommendation engine: plug in a collaborative filtering or rules-based recommender in recommender/.

Troubleshooting

Agent doesn't understand phrase: Add examples to config/intents.yml and retrain the NLU model.

Prices don't match: Ensure catalog/ is seeded correctly and caching is disabled if using external catalog.

Checkout fails in production: Check payment provider keys and webhooks.

Roadmap (suggested)

Persistent user profiles + cross-device sync

Smart cart suggestions (recipes → ingredients)

Real-time order tracking integration with courier partners

Loyalty and discount engine

Voice-first assistant skill

Contributing

Contributions welcome! Please:

Fork the repo

Create a feature branch

Open a PR with tests and documentation

Follow the code style in .editorconfig and pre-commit hooks configured in .pre-commit-config.yaml.

License

This project is provided under the MIT License — see LICENSE for details. (Change to your preferred license.)

Example files included

run_agent.py — bootstrap script for local demo

api/ — FastAPI server

cli.py — simple CLI client

ui/ — minimal web UI (React/Vue skeleton)

scripts/seed_catalog.py — seed script

sample_catalog.csv — sample products

.env.example — environment file template

Example conversation (full)

User: "I need ingredients for pancakes — eggs, milk, and flour. Also coffee."
Agent: "Added 1x All-purpose Flour (1kg), 1x Milk (1L), 12x Eggs, and 1x Ground Coffee to your cart. Would you like a suggested syrup or butter?"
User: "Yes — add maple syrup. Schedule delivery tomorrow morning."
Agent: "Maple syrup added. Delivery scheduled for tomorrow, 9:00–11:00 AM. Checkout now?"
