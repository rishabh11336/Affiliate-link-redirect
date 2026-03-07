<div align="center">

# 🐾 Affiliate Link Redirect

**A smart, serverless affiliate link management system for product showcase websites.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![GitHub Actions](https://img.shields.io/badge/CI-GitHub%20Actions-blue?logo=github-actions)](https://github.com/rishabh11336/Affiliate-link-redirect/actions)
[![Submodules](https://img.shields.io/badge/Submodules-3-green)](https://github.com/rishabh11336/Affiliate-link-redirect/blob/main/.gitmodules)

</div>

---

## 📖 Table of Contents

- [Overview](#-overview)
- [High-Level Architecture (HLD)](#-high-level-architecture-hld)
- [Repository Structure](#-repository-structure)
- [Technology Stack](#-technology-stack)
- [Low-Level Design (LLD)](#-low-level-design-lld)
  - [Data Flow: Browsing Products](#1-data-flow-browsing-products)
  - [Data Flow: Adding a Product (Admin)](#2-data-flow-adding-a-product-admin)
- [Quick Start](#-quick-start)
- [Environment Variables](#-environment-variables)
- [Deployment](#-deployment)
- [CI/CD](#-cicd)
- [Contributing](#-contributing)
- [License](#-license)

---

## 🌟 Overview

**Affiliate Link Redirect** is a zero-database, GitHub-as-backend affiliate product management system. It powers product showcase websites (like pet product blogs) with:

- 🛍️ A **public storefront** to display and filter affiliate products
- 🔐 A **secure admin dashboard** to add, edit, and delete products and images
- 📦 A **GitHub-hosted data layer** storing all products and images — no separate database needed

> Built for [Punji The Labrador](https://github.com/rishabh11336/Punji_The_Labrador) — a YouTube channel that recommends dog products with Amazon affiliate links.

---

## 🏗️ High-Level Architecture (HLD)

```
┌───────────────────────────────────────────────────────────────┐
│                   AFFILIATE-LINK-REDIRECT                     │
│              (Parent Monorepo with Git Submodules)            │
└───────────────┬───────────────────┬───────────────────────────┘
                │                   │                   │
                ▼                   ▼                   ▼
 ┌──────────────────┐  ┌─────────────────────┐  ┌─────────────────────────┐
 │  🌐 FRONTEND     │  │  ⚙️  BACKEND         │  │  🗄️  DATA REPOSITORY    │
 │  Static Website  │  │  Flask Admin API    │  │  Products + Images      │
 │                  │  │                     │  │                         │
 │ Punji_The_       │  │ punji_the_labrador_ │  │ Punji_The_Labrador_     │
 │ Labrador/        │  │ backend/            │  │ Images-JSON/            │
 │                  │  │                     │  │                         │
 │  • index.html    │  │  • Flask Routes     │  │  • products.json        │
 │  • script.js     │  │  • Auth middleware  │  │  • images/products/     │
 │  • styles.css    │  │  • GitHub Service   │  │                         │
 └──────┬───────────┘  └──────────┬──────────┘  └────────────┬────────────┘
        │                         │                           │
        │  Fetches products.json  │                           │
        │◄────────────────────────┼───────────────────────────┘
        │  (via GitHub raw URL)   │  Reads/Writes via GitHub API
        │                         │◄─────────────────────────►│
        ▼                         ▼
 ┌──────────────┐       ┌──────────────────┐
 │   VISITOR    │       │   SITE ADMIN     │
 │   (Browser)  │       │   (Browser)      │
 └──────────────┘       └──────────────────┘
```

### Key Design Decisions

| Decision | Reason |
|---|---|
| **GitHub as database** | Zero hosting cost, built-in versioning, public CDN for images |
| **Git submodules** | Independent deployment of frontend, backend, and data |
| **Static frontend** | Fast CDN delivery, no server required for public users |
| **Flask admin** | Lightweight backend only used by the site owner |
| **Base64 image encoding** | Stores images directly in GitHub via the API |

---

## 📁 Repository Structure

This is a **monorepo** managed with Git submodules. Each subdirectory is a separate GitHub repository.

```
Affiliate-link-redirect/                  ← You are here
├── .github/
│   └── workflows/
│       └── sync-submodules.yml           ← Daily submodule sync (GitHub Actions)
├── .gitmodules                           ← Submodule URLs
├── LICENSE                               ← MIT License
├── README.md                             ← This file
│
├── Punji_The_Labrador/                   ← [Submodule] Public-facing website
│   ├── index.html                        ← Product showcase page (SEO optimized)
│   ├── script.js                         ← Product fetch, filter, search, sort logic
│   ├── styles.css                        ← Mobile-first responsive CSS
│   ├── robots.txt                        ← SEO crawler rules
│   ├── sitemap.xml                       ← SEO sitemap
│   └── CNAME                            ← Custom domain config
│
├── punji_the_labrador_backend/           ← [Submodule] Admin Flask API
│   ├── app.py                            ← Flask routes & controllers
│   ├── requirements.txt                  ← Python dependencies
│   ├── Procfile                          ← Render/Heroku deploy config
│   ├── .env.example                      ← Environment variable template
│   ├── services/
│   │   ├── github_service.py             ← GitHub API CRUD operations
│   │   └── image_service.py             ← Image resize & optimization
│   ├── utils/
│   │   └── auth.py                       ← Login decorator & session auth
│   ├── templates/                        ← Jinja2 HTML templates
│   │   ├── login.html
│   │   ├── admin.html
│   │   ├── edit.html
│   │   └── success.html
│   └── static/                           ← CSS/JS for admin panel
│
└── Punji_The_Labrador_Images-JSON/       ← [Submodule] Data & image store
    ├── products.json                     ← Product catalog (JSON)
    └── images/
        └── products/                     ← Product images (GitHub CDN)
```

### Submodule Links

| Submodule | Repository | Description |
|---|---|---|
| `Punji_The_Labrador/` | [rishabh11336/Punji_The_Labrador](https://github.com/rishabh11336/Punji_The_Labrador) | Public storefront |
| `punji_the_labrador_backend/` | [rishabh11336/punji_the_labrador_backend](https://github.com/rishabh11336/punji_the_labrador_backend) | Admin Flask API |
| `Punji_The_Labrador_Images-JSON/` | [rishabh11336/Punji_The_Labrador_Images-JSON](https://github.com/rishabh11336/Punji_The_Labrador_Images-JSON) | Products + images data |

---

## 🛠️ Technology Stack

| Layer | Technology | Purpose |
|---|---|---|
| **Frontend** | HTML5, CSS3, Vanilla JS | Public product showcase |
| **Backend** | Python, Flask 3.0 | Admin API & auth |
| **Data Store** | GitHub API + JSON | Product catalog & images |
| **Image Processing** | Pillow | Resize, optimize, convert |
| **GitHub Integration** | PyGithub 2.5 | Read/write repo content |
| **Deployment** | Gunicorn, Procfile | Production WSGI server |
| **CI/CD** | GitHub Actions | Daily submodule sync |

---

## 🔬 Low-Level Design (LLD)

### 1. Data Flow: Browsing Products

```
User visits Website
        │
        ▼
index.html loads (HTML + CSS)
        │
        ▼
script.js: fetchProducts()
        │
        ▼
GET https://raw.githubusercontent.com/
    rishabh11336/Punji_The_Labrador_Images-JSON/
    main/products.json
        │
        ▼
Parse JSON → products[]
        │
        ▼
renderProducts()
    ├── filterProducts()    ← by category button
    ├── sortProducts()      ← by price / rating / featured
    └── searchProducts()    ← by name / description
        │
        ▼
createProductCard(product)
    ├── Render name, price, rating, badge
    ├── Lazy-load product image
    └── "Buy on Amazon" → affiliateLink (opens in new tab)
        │
        ▼
Inject cards into #product-grid (DOM)
```

### 2. Data Flow: Adding a Product (Admin)

```
Admin submits form (POST /upload)
        │
        ▼
Flask app.py — validate session (@require_auth)
        │
        ▼
image_service.process_image(file)
    ├── Validate file type  (PNG / JPG / WebP)
    ├── Resize to max 800×800 px
    ├── Convert to RGB (strip alpha)
    ├── Compress to JPEG quality 85
    └── Save to uploads/ with UUID filename
        │
        ▼
github_service.upload_image(filepath, filename)
    ├── Read file bytes → Base64 encode
    ├── GitHub API PUT /repos/.../images/products/{filename}
    └── Returns CDN URL  (raw.githubusercontent.com)
        │
        ▼
github_service.add_product(product_data)
    ├── GET current products.json  (fetch SHA + content)
    ├── Append new product with auto-incremented id
    ├── GitHub API PUT /repos/.../products.json
    └── Commit message: "add product: {name}"
        │
        ▼
image_service.cleanup(filepath)  ← remove temp file
        │
        ▼
Redirect → success.html
        │
        ▼
Frontend: next page load fetches updated products.json
```

### 3. Admin API Routes

| Method | Route | Description | Auth |
|---|---|---|---|
| `GET` | `/` | Redirect to login or admin | — |
| `GET/POST` | `/login` | Admin login form | — |
| `GET` | `/logout` | Clear session | ✅ |
| `GET` | `/admin` | Product dashboard | ✅ |
| `POST` | `/upload` | Add new product | ✅ |
| `GET` | `/products` | List all products (JSON) | ✅ |
| `GET/POST` | `/edit/<id>` | Edit product | ✅ |
| `POST` | `/delete/<id>` | Delete product | ✅ |
| `GET` | `/health` | Health check | — |

### 4. Product Data Schema

```json
{
  "id": 1,
  "name": "Premium Dog Chew Bone",
  "category": "toys",
  "price": 29.99,
  "originalPrice": 34.99,
  "rating": 4.8,
  "reviews": 1523,
  "image": "https://raw.githubusercontent.com/rishabh11336/Punji_The_Labrador_Images-JSON/main/images/products/{uuid}.jpg",
  "description": "Long-lasting chew bone for large dogs...",
  "affiliateLink": "https://amzn.to/xxxxxxx",
  "badge": "Top Rated"
}
```

---

## 🚀 Quick Start

### Prerequisites

- Python 3.9+
- A GitHub account with a [Personal Access Token](https://github.com/settings/tokens) (repo scope)

### 1. Clone the Monorepo

```bash
git clone --recurse-submodules https://github.com/rishabh11336/Affiliate-link-redirect.git
cd Affiliate-link-redirect
```

### 2. Set Up the Backend

```bash
cd punji_the_labrador_backend
python3 -m venv venv
source venv/bin/activate          # Windows: venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env
```

Edit `.env` with your credentials — see [Environment Variables](#-environment-variables).

```bash
python app.py
# Admin panel → http://localhost:5000
```

### 3. Run the Frontend Locally

```bash
cd Punji_The_Labrador
python -m http.server 8000
# Open → http://localhost:8000
```

---

## 🔑 Environment Variables

All configuration is managed through environment variables in [`punji_the_labrador_backend/.env.example`](https://github.com/rishabh11336/punji_the_labrador_backend/blob/main/.env.example).

| Variable | Description | Example |
|---|---|---|
| `GITHUB_TOKEN` | Personal Access Token with `repo` scope | `ghp_xxxxxx` |
| `GITHUB_REPO_OWNER` | GitHub username owning the data repo | `rishabh11336` |
| `DATA_REPO_NAME` | Name of the data submodule repository | `Punji_The_Labrador_Images-JSON` |
| `ADMIN_USERNAME` | Admin panel login username | `admin` |
| `ADMIN_PASSWORD` | Admin panel login password | `SecurePass123!` |
| `SECRET_KEY` | Flask session secret key | `random-long-string` |
| `FLASK_ENV` | `development` or `production` | `production` |

> ⚠️ **Never commit your `.env` file.** It is already listed in `.gitignore`.

---

## ☁️ Deployment

### Frontend (GitHub Pages / Netlify)

The [`Punji_The_Labrador/`](https://github.com/rishabh11336/Punji_The_Labrador) submodule is a static site — deploy it directly to **GitHub Pages** or **Netlify** with no build step required.

### Backend (Render / Railway / Heroku)

The [`punji_the_labrador_backend/`](https://github.com/rishabh11336/punji_the_labrador_backend) submodule includes a `Procfile`:

```
web: gunicorn app:app
```

1. Connect the backend repository to [Render](https://render.com) or [Railway](https://railway.app)
2. Add all environment variables from `.env.example` in the platform's dashboard
3. Deploy — the platform uses `Procfile` automatically

---

## ⚙️ CI/CD

A GitHub Actions workflow at [`.github/workflows/sync-submodules.yml`](./.github/workflows/sync-submodules.yml) keeps all submodules in sync automatically.

```
Schedule: every day at 00:00 UTC
         │
         ▼
Checkout parent repo + all submodules (recursive)
         │
         ▼
git submodule update --remote --merge
         │
         ▼
Any changes? ──Yes──► Commit + Push to main
         │
         No
         │
         ▼
Done ✅
```

You can also trigger it manually from the [Actions tab](https://github.com/rishabh11336/Affiliate-link-redirect/actions).

---

## 🤝 Contributing

1. Fork this repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m 'feat: add some feature'`
4. Push to the branch: `git push origin feature/your-feature`
5. Open a Pull Request

For submodule-specific changes, contribute directly to the relevant submodule repository listed in the [Submodule Links](#submodule-links) table.

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](./LICENSE) file for details.

---

<div align="center">

Made with ❤️ for [Punji The Labrador 🐶](https://github.com/rishabh11336/Punji_The_Labrador)

</div>
