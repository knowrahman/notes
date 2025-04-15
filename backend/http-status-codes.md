## HTTP Status Code Categories

| Range      | Category               | Meaning                          |
|------------|------------------------|----------------------------------|
| 1xx        | Informational          | Request received, continuing     |
| 2xx        | Success                | Request was successful           |
| 3xx        | Redirection            | Further action needed            |
| 4xx        | Client Error           | Problem with the request         |
| 5xx        | Server Error           | Server failed to handle request  |

---

## Most Common Status Codes to Remember

### ✅ 2xx: Success

| Code | Meaning                        | Use Case                           |
|------|--------------------------------|-------------------------------------|
| 200  | OK                             | Successful GET or POST             |
| 201  | Created                        | Resource successfully created      |
| 204  | No Content                     | Successful but no data returned    |

---

### 🚫 4xx: Client Errors

| Code | Meaning                        | Use Case                           |
|------|--------------------------------|-------------------------------------|
| 400  | Bad Request                    | Invalid input or malformed request |
| 401  | Unauthorized                   | Missing or invalid auth credentials |
| 403  | Forbidden                      | Authenticated but not allowed      |
| 404  | Not Found                      | Resource doesn’t exist             |
| 409  | Conflict                       | Resource conflict (e.g. duplicate) |
| 422  | Unprocessable Entity (Web API)| Validation failed (semantic errors)|

---

### 💥 5xx: Server Errors

| Code | Meaning                        | Use Case                           |
|------|--------------------------------|-------------------------------------|
| 500  | Internal Server Error          | Unexpected error on server         |
| 502  | Bad Gateway                    | Invalid response from upstream     |
| 503  | Service Unavailable            | Server is down or overloaded       |
| 504  | Gateway Timeout                | Upstream server didn’t respond     |

---

## Other Useful Ones

| Code | Meaning                        | Use Case                            |
|------|--------------------------------|--------------------------------------|
| 301  | Moved Permanently              | URL permanently changed             |
| 302  | Found (Temporary Redirect)     | Temporary redirect                  |
| 304  | Not Modified                   | Use browser cache                   |
| 405  | Method Not Allowed             | Wrong HTTP method used              |

---

## Summary of Must-Know Codes

| Group | Must-Know Codes               |
|-------|-------------------------------|
| 2xx   | 200, 201, 204                 |
| 4xx   | 400, 401, 403, 404, 409, 422 |
| 5xx   | 500, 502, 503, 504           |

---

