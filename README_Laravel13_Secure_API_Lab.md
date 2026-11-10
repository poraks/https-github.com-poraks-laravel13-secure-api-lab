# Laravel 13 Secure API with Role-Based Access Control (RBAC)

*Name:* Chareles Anthony V. De leon

*Section:* BSICT-3B
*Course:* Integrative Programming & Technologies II (Module 5 - Lesson 2)

## Overview

A secure API built with Laravel 13 and Sanctum Bearer-token authentication.
Two custom middleware classes restrict endpoints by user role:

- CheckRole: accepts one or more allowed roles (for example role:admin,student)
- AdminMiddleware: admin-only

The users table has a role column (default student). Public registration
always creates a student account and never accepts a role from the request.
Admin accounts are created only through trusted server-side code
(php artisan tinker).

## RBAC routes

| Endpoint | Method | Middleware | Admin | Student | No token |
|---|---|---|---|---|---|
| /api/profile | GET | auth:sanctum | 200 | 200 | 401 |
| /api/students | GET | auth:sanctum, role:admin,student | 200 | 200 | 401 |
| /api/admin/dashboard | GET | auth:sanctum, admin | 200 | 403 | 401 |
| /api/student/dashboard | GET | auth:sanctum, role:student | 403 | 200 | 401 |

Public endpoints: POST /api/register, POST /api/login.
Protected: POST /api/logout (auth:sanctum).

## Local setup and testing

1. Start MySQL (XAMPP) and copy .env.example to .env, then set your database values.
2. Run:

   
   composer install
   php artisan key:generate
   php artisan migrate
   php artisan serve
   

3. Create one admin and one student account with php artisan tinker
   (use your own temporary passwords and do not commit them).
4. In Postman, set the header Accept: application/json, call
   POST /api/login, then use the returned token in
   Authorization > Bearer Token for the protected routes.

## Postman screenshots

Located in the screenshots/ folder:

| # | Request | Token | Status | File |
|---|---|---|---|---|
| 1 | POST /api/login | None (admin credentials) | 200 | 01-admin-login.png |
| 2 | GET /api/admin/dashboard | Admin | 200 | 02-admin-allowed.png |
| 3 | GET /api/student/dashboard | Admin | 403 | 03-admin-denied.png |
| 4 | POST /api/login | None (student credentials) | 200 | 04-student-login.png |
| 5 | GET /api/student/dashboard | Student | 200 | 05-student-allowed.png |
| 6 | GET /api/admin/dashboard | Student | 403 | 06-student-denied.png |
| 7 | GET /api/students | Student | 200 | 07-shared-route.png |
| 8 | GET /api/admin/dashboard | None | 401 | 08-no-token.png |

*Note:* POST /api/login returns 200 with JSON containing message,
token_type, and access_token.

## Reflection

*What is the difference between authentication and authorization?*

Authentication checks who you are (for example a valid login and token).
Authorization checks what you are allowed to do (your role or permissions).

*Why does a student receive 403 on the admin dashboard, but a request without a token receives 401?*

Without a token, the server cannot identify the user, so auth:sanctum returns
401 Unauthenticated. A student has a valid token, so they are authenticated,
but their role is not allowed on the admin route, so the middleware returns
403 Forbidden.