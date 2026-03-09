# Django Built-in Authentication Models

> A practical reference for building and extending authentication systems using Django's built-in auth base models.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [AbstractUser](#2-abstractuser)
3. [AbstractBaseUser](#3-abstractbaseuser)
4. [BaseUserManager](#4-baseusermanager)
5. [PermissionsMixin](#5-permissionsmixin)
6. [Full Example — Email Login with All Four Models](#6-full-example--email-login-with-all-four-models)
7. [Example — Phone Number Login](#7-example--phone-number-login)
8. [Comparison](#8-comparison)
9. [Summary](#9-summary)

---

## 1. Introduction

Django ships with built-in authentication base models that allow developers to either extend the default user model or build a fully custom authentication system from scratch.

| Model | Role |
|---|---|
| `AbstractUser` | Extend the default Django user model |
| `AbstractBaseUser` | Build a completely custom auth system |
| `BaseUserManager` | Handle user and superuser creation logic |
| `PermissionsMixin` | Add permissions and group support to any user model |

> ⚠️ **Important:** Always set `AUTH_USER_MODEL = 'my_app.User'` in `settings.py` **before** the first migration. Changing it afterwards requires a full database reset.

---

## 2. AbstractUser

`AbstractUser` is a full-featured extension of Django's default `User` model. It includes all standard fields and permissions out of the box — you only need to add your custom fields.

**Predefined Fields:**

| Field | Description |
|---|---|
| `username` | Unique username |
| `password` | Hashed password |
| `first_name` | User's first name |
| `last_name` | User's last name |
| `email` | Email address |
| `is_staff` | Admin panel access flag |
| `is_active` | Account active/inactive status |
| `is_superuser` | Full permissions flag |
| `last_login` | Timestamp of last login |
| `date_joined` | Timestamp of account creation |
| `groups` | Many-to-Many relationship with groups |
| `user_permissions` | Many-to-Many relationship with permissions |

**Example — Extending with custom fields:**

```python
from django.contrib.auth.models import AbstractUser
from django.db import models


class User(AbstractUser):
    phone_number = models.CharField(max_length=15, blank=True)
    avatar       = models.ImageField(upload_to='avatars/', null=True, blank=True)
```

```python
# settings.py
AUTH_USER_MODEL = 'my_app.User'
```

> 💡 **Tip:** Use `AbstractUser` when you only need to add fields to the default user model. It's the fastest path to a custom user.

---

## 3. AbstractBaseUser

`AbstractBaseUser` is a minimal base class for building a completely custom authentication system. It provides only core auth functionality — everything else must be defined manually.

**Predefined Fields:**

| Field | Description |
|---|---|
| `password` | Hashed password |
| `last_login` | Timestamp of last login |

**Required Manual Definitions:**

| Attribute | Purpose |
|---|---|
| `USERNAME_FIELD` | The field used as the login identifier |
| `REQUIRED_FIELDS` | Fields required when creating a superuser via CLI |
| `objects` | A custom `BaseUserManager` instance |

**Example:**

```python
from django.contrib.auth.models import AbstractBaseUser
from django.db import models


class User(AbstractBaseUser):
    email     = models.EmailField(unique=True)
    is_active = models.BooleanField(default=True)
    is_admin  = models.BooleanField(default=False)

    objects = UserManager()           # defined in Section 4

    USERNAME_FIELD  = 'email'
    REQUIRED_FIELDS = []
```

> ⚠️ **Important:** `AbstractBaseUser` does not include `is_staff`, `is_superuser`, or permissions. Pair it with `PermissionsMixin` if you need them.

---

## 4. BaseUserManager

`BaseUserManager` is the class responsible for all user creation logic. It is **required** when using `AbstractBaseUser` — without it, Django has no way to create users programmatically or via the `createsuperuser` command.

**Inherited Utility Methods:**

| Method | Purpose |
|---|---|
| `normalize_email(email)` | Lowercases the domain part of an email address |
| `make_random_password()` | Generates a random password string |
| `get_by_natural_key(username)` | Fetches a user by their `USERNAME_FIELD` value |

**Methods You Must Define:**

| Method | Purpose |
|---|---|
| `create_user()` | Creates and saves a standard user |
| `create_superuser()` | Creates and saves an admin user via CLI |

**Example:**

```python
from django.contrib.auth.models import BaseUserManager


class UserManager(BaseUserManager):

    def create_user(self, email, password=None, **extra_fields):
        if not email:
            raise ValueError('Email is required')

        email = self.normalize_email(email)
        user  = self.model(email=email, **extra_fields)
        user.set_password(password)          # hashes the password
        user.save(using=self._db)
        return user

    def create_superuser(self, email, password=None, **extra_fields):
        extra_fields.setdefault('is_admin',     True)
        extra_fields.setdefault('is_superuser', True)

        return self.create_user(email, password, **extra_fields)
```

> 📌 **Note:** Always use `set_password()` instead of assigning the password directly. Direct assignment stores plain text — `set_password()` hashes it correctly.

> 💡 **Tip:** `**extra_fields` makes your manager flexible — any additional model fields passed in are handled automatically without changing the method signature.

---

## 5. PermissionsMixin

`PermissionsMixin` adds Django's full permission and group system to any custom user model built on `AbstractBaseUser`.

**Predefined Fields:**

| Field | Description |
|---|---|
| `is_superuser` | Grants all permissions without explicit assignment |
| `groups` | Many-to-Many relationship with `Group` |
| `user_permissions` | Many-to-Many relationship with `Permission` |

**Inherited Methods:**

| Method | Purpose |
|---|---|
| `has_perm(perm)` | Returns `True` if user has the specified permission |
| `has_module_perms(app_label)` | Returns `True` if user has any permission in the app |

**Example — Combining with AbstractBaseUser:**

```python
from django.contrib.auth.models import AbstractBaseUser, PermissionsMixin
from django.db import models


class User(AbstractBaseUser, PermissionsMixin):
    email     = models.EmailField(unique=True)
    is_staff  = models.BooleanField(default=False)
    is_active = models.BooleanField(default=True)

    objects = UserManager()

    USERNAME_FIELD  = 'email'
    REQUIRED_FIELDS = []
```

> 💡 **Tip:** `has_perm()` and `has_module_perms()` are required by Django's admin. Without `PermissionsMixin`, you must implement them manually.

---

## 6. Full Example — Email Login with All Four Models

A complete production-ready custom user setup using all four components together.

```python
from django.contrib.auth.models import AbstractBaseUser, BaseUserManager, PermissionsMixin
from django.db import models


# ── Manager ───────────────────────────────────────────────────────────────────

class UserManager(BaseUserManager):

    def create_user(self, email, password=None, **extra_fields):
        if not email:
            raise ValueError('Email is required')
        email = self.normalize_email(email)
        user  = self.model(email=email, **extra_fields)
        user.set_password(password)
        user.save(using=self._db)
        return user

    def create_superuser(self, email, password=None, **extra_fields):
        extra_fields.setdefault('is_staff',     True)
        extra_fields.setdefault('is_superuser', True)
        extra_fields.setdefault('is_active',    True)
        return self.create_user(email, password, **extra_fields)


# ── Model ─────────────────────────────────────────────────────────────────────

class User(AbstractBaseUser, PermissionsMixin):
    email      = models.EmailField(unique=True)
    first_name = models.CharField(max_length=50, blank=True)
    last_name  = models.CharField(max_length=50, blank=True)
    is_staff   = models.BooleanField(default=False)
    is_active  = models.BooleanField(default=True)
    created_at = models.DateTimeField(auto_now_add=True)

    objects = UserManager()

    USERNAME_FIELD  = 'email'
    REQUIRED_FIELDS = []

    def __str__(self):
        return self.email
```

```python
# settings.py
AUTH_USER_MODEL = 'my_app.User'
```

---

## 7. Example — Phone Number Login

Replace the default `username` login with `phone_number` using `AbstractUser`.

```python
from django.contrib.auth.models import AbstractUser
from django.db import models


class User(AbstractUser):
    username     = None
    phone_number = models.CharField(max_length=15, unique=True)

    USERNAME_FIELD  = 'phone_number'
    REQUIRED_FIELDS = []
```

Setting `username = None` removes the default username field entirely. Django will now use `phone_number` as the login identifier.

---

## 8. Comparison

| Model | Provides | Use When | Difficulty |
|---|---|---|---|
| `User` (default) | All fields + permissions | No customisation needed | Easy |
| `AbstractUser` | All fields + permissions | Adding extra fields to default user | Medium |
| `AbstractBaseUser` | Password + last\_login only | Building fully custom auth logic | Advanced |
| `BaseUserManager` | `create_user`, `create_superuser` | Always required with `AbstractBaseUser` | Advanced |
| `PermissionsMixin` | Groups + permissions + `has_perm()` | Needed alongside `AbstractBaseUser` | Medium |

---

## 9. Summary

```
AbstractUser        →  Extend the default user model with extra fields
AbstractBaseUser    →  Define the custom user model structure
BaseUserManager     →  Define how users and superusers are created
PermissionsMixin    →  Add Django's permission and group system
```

> ⚠️ **Important:** `AbstractBaseUser` + `BaseUserManager` + `PermissionsMixin` always work together. Using `AbstractBaseUser` without the other two results in an incomplete authentication system.