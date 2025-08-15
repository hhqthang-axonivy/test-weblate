```rst
===============================
API Reference and Developer Guide For Developer
===============================

.. meta::
   :description: Complete API reference for developers
   :keywords: API, REST, authentication, endpoints

Overview
========

This comprehensive guide covers our REST API, authentication methods, and integration examples for developers building applications with our platform.

.. contents:: Table of Contents
   :local:
   :depth: 2

Authentication
==============

API Token Authentication
------------------------

To authenticate with our API, you need to include your API token in the request headers:

.. code-block:: http

   GET /api/v1/users HTTP/1.1
   Host: api.example.com
   Authorization: Bearer your-api-token-here
   Content-Type: application/json

.. important::
   Keep your API tokens secure! Never expose them in client-side code or public repositories.

OAuth 2.0 Flow
---------------

For applications requiring user authorization, use OAuth 2.0:

1. **Authorization Request**
   
   Redirect users to our authorization endpoint:
   
   .. code-block:: url
   
      https://auth.example.com/oauth/authorize?
        client_id=your_client_id&
        response_type=code&
        scope=read+write&
        redirect_uri=https://yourapp.com/callback

2. **Token Exchange**
   
   Exchange the authorization code for an access token:
   
   .. code-block:: bash
   
      curl -X POST https://auth.example.com/oauth/token \
        -H "Content-Type: application/x-www-form-urlencoded" \
        -d "grant_type=authorization_code" \
        -d "code=AUTHORIZATION_CODE" \
        -d "client_id=YOUR_CLIENT_ID" \
        -d "client_secret=YOUR_CLIENT_SECRET"

API Endpoints
=============

User Management
---------------

Get User Profile
~~~~~~~~~~~~~~~~

Retrieve information about the current user:

.. http:get:: /api/v1/user

   **Example Request:**
   
   .. code-block:: bash
   
      curl -H "Authorization: Bearer TOKEN" \
           https://api.example.com/api/v1/user

   **Example Response:**
   
   .. code-block:: json
   
      {
        "id": 12345,
        "username": "john_doe",
        "email": "john@example.com",
        "created_at": "2024-01-15T10:30:00Z",
        "last_login": "2024-03-10T14:22:00Z",
        "profile": {
          "first_name": "John",
          "last_name": "Doe",
          "timezone": "UTC",
          "language": "en"
        }
      }

   :statuscode 200: Success
   :statuscode 401: Unauthorized - Invalid or missing token
   :statuscode 429: Rate limit exceeded

Update User Profile
~~~~~~~~~~~~~~~~~~~

.. http:patch:: /api/v1/user

   Update user profile information.

   **Request Body:**
   
   .. code-block:: json
   
      {
        "profile": {
          "first_name": "Jane",
          "timezone": "America/New_York",
          "language": "es"
        }
      }

Project Management
------------------

List Projects
~~~~~~~~~~~~~

.. http:get:: /api/v1/projects

   Retrieve a list of projects accessible to the authenticated user.

   :query int limit: Number of results per page (default: 50, max: 100)
   :query int offset: Pagination offset (default: 0)
   :query string status: Filter by project status (active, archived, draft)
   :query string search: Search projects by name or description

Create New Project
~~~~~~~~~~~~~~~~~~

.. http:post:: /api/v1/projects

   Create a new project.

   **Required Fields:**
   
   .. code-block:: json
   
      {
        "name": "My New Project",
        "description": "Project description goes here",
        "visibility": "private",
        "settings": {
          "auto_deploy": true,
          "notifications": true
        }
      }

Error Handling
==============

Standard Error Response
-----------------------

All API errors follow a consistent format:

.. code-block:: json

   {
     "error": {
       "code": "VALIDATION_ERROR",
       "message": "The request data is invalid",
       "details": [
         {
           "field": "email",
           "message": "Email address is required"
         }
       ],
       "request_id": "req_abc123def456"
     }
   }

Common Error Codes
------------------

.. glossary::

   AUTHENTICATION_REQUIRED
      The request requires authentication. Include a valid API token.

   INSUFFICIENT_PERMISSIONS
      The authenticated user lacks permission for this operation.

   VALIDATION_ERROR
      Request data validation failed. Check the details array for specific field errors.

   RATE_LIMIT_EXCEEDED
      Too many requests in a short time period. Wait before retrying.

   RESOURCE_NOT_FOUND
      The requested resource does not exist or is not accessible.

   SERVER_ERROR
      An internal server error occurred. Contact support if this persists.

Rate Limiting
=============

API requests are subject to rate limiting:

.. list-table:: Rate Limits by Plan
   :header-rows: 1
   :widths: 20 20 30 30

   * - Plan Type
     - Requests/Hour
     - Burst Limit
     - Overage Policy
   * - Free
     - 1,000
     - 100/minute
     - Block requests
   * - Professional
     - 10,000
     - 500/minute
     - Charge for overage
   * - Enterprise
     - 100,000
     - 2,000/minute
     - Custom limits

Rate limit information is included in response headers:

.. code-block:: http

   HTTP/1.1 200 OK
   X-RateLimit-Limit: 1000
   X-RateLimit-Remaining: 999
   X-RateLimit-Reset: 1609459200

SDK Examples
============

Python SDK
-----------

.. code-block:: python

   import requests
   from datetime import datetime
   
   class APIClient:
       def __init__(self, api_token, base_url="https://api.example.com"):
           self.session = requests.Session()
           self.session.headers.update({
               'Authorization': f'Bearer {api_token}',
               'Content-Type': 'application/json'
           })
           self.base_url = base_url
       
       def get_user_profile(self):
           """Retrieve current user profile"""
           response = self.session.get(f"{self.base_url}/api/v1/user")
           response.raise_for_status()
           return response.json()
       
       def create_project(self, name, description, visibility="private"):
           """Create a new project"""
           data = {
               "name": name,
               "description": description,
               "visibility": visibility,
               "created_at": datetime.utcnow().isoformat()
           }
           response = self.session.post(f"{self.base_url}/api/v1/projects", json=data)
           response.raise_for_status()
           return response.json()

JavaScript SDK
---------------

.. code-block:: javascript

   class APIClient {
       constructor(apiToken, baseURL = 'https://api.example.com') {
           this.apiToken = apiToken;
           this.baseURL = baseURL;
       }
   
       async makeRequest(endpoint, options = {}) {
           const url = `${this.baseURL}${endpoint}`;
           const config = {
               headers: {
                   'Authorization': `Bearer ${this.apiToken}`,
                   'Content-Type': 'application/json',
                   ...options.headers
               },
               ...options
           };
   
           const response = await fetch(url, config);
           
           if (!response.ok) {
               throw new Error(`API request failed: ${response.statusText}`);
           }
           
           return response.json();
       }
   
       async getUserProfile() {
           return this.makeRequest('/api/v1/user');
       }
   
       async createProject(projectData) {
           return this.makeRequest('/api/v1/projects', {
               method: 'POST',
               body: JSON.stringify(projectData)
           });
       }
   }

Webhooks
========

Event Types
-----------

Our platform sends webhooks for the following events:

.. hlist::
   :columns: 2

   * ``project.created``
   * ``project.updated``
   * ``project.deleted``
   * ``user.registered``
   * ``user.updated``
   * ``deployment.started``
   * ``deployment.completed``
   * ``deployment.failed``

Webhook Configuration
---------------------

Configure webhooks in your project settings:

1. Navigate to **Project Settings** → **Webhooks**
2. Click **Add Webhook**
3. Enter your endpoint URL
4. Select events to subscribe to
5. Configure retry settings

.. danger::
   Webhook endpoints must respond with HTTP 200-299 status codes within 10 seconds, or the delivery will be considered failed.

Testing and Development
=======================

Sandbox Environment
-------------------

Use our sandbox environment for testing:

.. code-block:: bash

   # Sandbox API base URL
   export API_BASE_URL="https://sandbox-api.example.com"
   
   # Use test API tokens (they start with 'test_')
   export API_TOKEN="test_sk_1234567890abcdef"

Mock Data
---------

The sandbox includes realistic mock data:

* 50+ sample users
* 100+ sample projects  
* Realistic timestamps and relationships
* Predictable responses for testing edge cases

Support and Resources
=====================

Getting Help
------------

.. note::
   Our support team is available 24/7 for Enterprise customers, and during business hours (9 AM - 6 PM UTC) for all other plans.

**Documentation:**
   * API Reference: https://docs.example.com/api
   * Tutorials: https://docs.example.com/tutorials
   * Code Examples: https://github.com/example/api-examples

**Community:**
   * Stack Overflow: Tag your questions with ``example-api``
   * Discord: https://discord.gg/example-developers
   * GitHub Discussions: https://github.com/example/api-feedback

**Direct Support:**
   * Email: developers@example.com
   * Support Portal: https://support.example.com

Changelog
=========

Version 2.1 (March 2024)
-------------------------

**New Features:**
   * Added batch operations for user management
   * Improved rate limiting with burst capacity
   * New webhook event types for deployments

**Breaking Changes:**
   * Deprecated ``/api/v1/legacy`` endpoints (removal planned for v3.0)
   * Changed default pagination limit from 25 to 50

**Bug Fixes:**
   * Fixed timezone handling in date filters
   * Resolved intermittent 500 errors in project creation

---

.. admonition:: Stay Updated
   :class: tip

   Subscribe to our developer newsletter at https://example.com/developers/newsletter to receive updates about API changes, new features, and best practices.

*Last updated: March 15, 2024 | API Version: 2.1.0*
```