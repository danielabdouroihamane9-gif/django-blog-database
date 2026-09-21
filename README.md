# Django Blog Database & ORM Implementation

A modular Django application demonstrating relational database design, custom user authentication models, complex QuerySet operations, and performance optimization techniques using Django's Object-Relational Mapper (ORM) with SQLite.

---

## Architecture & Database Schema

The core database schema is defined within the `blog` application, consisting of four interconnected models:

```Markdown
┌─────────────┐
│    User     │ (AbstractUser)
└──────┬──────┘
│
┌──────┴──────┐
▼             ▼
Post ◄────── Comment
│
▼
Tag
```

### Data Models & Relationships

1. **User (`blog.User`)**

   * Extends Django’s built-in `AbstractUser` to support custom profiles.
   * **Fields:** `username`, `email`, `password`, `bio` (TextField), `avatar` (URLField).
   * **Configured in `settings.py` via:** `AUTH_USER_MODEL = 'blog.User'`
2. **Tag (`blog.Tag`)**

   * Categorizes blog posts by topic.
   * **Fields:** `name` (`CharField` with `unique=True`).
3. **Post (`blog.Post`)**

   * Represents articles created by users.
   * **Fields:** `title`, `body`, `author` (ForeignKey to `User`), `published` (BooleanField), `created_at` (DateTimeField), `tags` (ManyToManyField to `Tag`).
   * **Database Indexing:** Indexed on `published` and `created_at` to optimize search and sorting operations.
4. **Comment (`blog.Comment`)**

   * User engagement on individual posts.
   * **Fields:** `post` (ForeignKey to `Post` with `related_name='comments'`), `author` (ForeignKey to `User`), `body`, `created_at`.

---

## Key Features & ORM Techniques Applied

* **Custom User Inheritance:** Seamlessly extended Django's authentication model without disrupting default admin functionalities.
* **Relational Mapping:** Implemented One-to-Many (`ForeignKey`) and Many-to-Many (`ManyToManyField`) relationships with proper `on_delete` behaviors.
* **Database Aggregation:** Used `annotate()` and `Count()` to compute dynamic fields (such as comment counts per post) on the fly without storing redundant columns.
* **Query Optimization:** Applied `select_related()` for single-valued relationships (ForeignKeys) to minimize query load and prevent $N+1$ database performance bottlenecks.
* **Reverse Lookups:** Configured `related_name='comments'` to allow clean, intuitive reverse relational querying from parent models.

---

## Setup & Installation Guide

### 1. Clone the Repository & Setup Environment

```powershell
git clone [https://github.com/danielabdouroihamane9-gif/django-blog-database.git](https://github.com/danielabdouroihamane9-gif/django-blog-database.git)
cd django-blog-database

# Create virtual environment
python -m venv venv

# Activate virtual environment (Windows PowerShell)
.\venv\Scripts\activate
```

### 2. Install Dependencies

```powershell
pip install django
```

### 3. Run Database Migrations

```powershell
python manage.py makemigrations
python manage.py migrate
```

### 4. Create Admin Superuser

```powershell
python manage.py createsuperuser
```

### 5. Start Development Server

```powershell
python manage.py runserver
```

Access the Django Admin panel at `http://127.0.0.1:8000/admin/`.

## Common ORM Queries & Usage

You can test these database queries directly inside the Django interactive shell (`python manage.py shell`):

## 1. Fetch Published Posts

```Python
from blog.models import Post
published_posts = Post.objects.filter(published=True).select_related('author')
```

### 2. Filter Posts by Tag

```Python
python_posts = Post.objects.filter(tags__name="python")
```

### 3. Top 5 Most Commented Posts (Aggregation)

```Python
from django.db.models import Count

top_posts = Post.objects.annotate(
    comment_count=Count('comments')
).order_by('-comment_count')[:5]

for post in top_posts:
    print(post.title, post.comment_count)
```

### 4. User Comment Aggregation

```Python
from blog.models import User
from django.db.models import Count

users = User.objects.annotate(total_comments=Count('comment'))
for user in users:
    print(user.username, user.total_comments)
```

## Project Structure

```Markdown
django_database/
│
├── config/             # Project configuration directory
│   ├── settings.py     # Application settings & database configurations
│   ├── urls.py         # Root URL routing
│   └── wsgi.py
│
├── blog/               # Core application
│   ├── migrations/     # Database migration blueprints
│   ├── admin.py        # Admin panel model registrations
│   ├── models.py       # User, Post, Tag, Comment schema definitions
│   └── views.py
│
├── db.sqlite3          # Local development database (Git ignored)
├── manage.py           # Django CLI management script
├── .gitignore          # Git exclusion rules
└── README.md           # Project documentation
```
