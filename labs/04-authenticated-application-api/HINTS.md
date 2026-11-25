# Hints — Lab 04: Authenticated application API

Use these in order; stop as soon as one is enough to unblock you. These hints are for Level 1 only. Level 2–4 guidance will be created when those exercises become active; this file is not a solution key for those levels.

## Level 1 — Guided implementation

1. **Broad:** Draw the request path and label each boundary where identity is established or permission is decided. Cognito authenticates an application user; API Gateway validates the configured token; Lambda's execution role is a separate AWS identity used by the function to call AWS services.
2. **Token boundary:** Treat a JWT as untrusted input until the configured authorizer validates it. A decoded token payload by itself is not proof. The request context supplied by API Gateway is the trusted source for the verified caller claims.
3. **Authentication versus authorization:** A valid token answers “who is this caller?” It does not automatically mean every authenticated user may use every operation. The diagnostics route needs an explicit application rule for the operations group.
4. **Identity source:** Derive the profile subject and group membership from the verified authorizer context. Ignore or reject identity/group values in request parameters, headers, or body; otherwise a caller could choose their own permissions.
5. **Group claims:** Group membership is reflected in tokens issued for that user. Create/assign the operations test user before signing in; if membership changes, sign in again to obtain a token with the current claims.
6. **Request model:** API Gateway REST API request validation is attached to a method and a content type. Check that the validator is actually enabled for the deployed method and that the request `Content-Type` matches the model mapping.
7. **Denial behavior:** Test missing/invalid credentials separately from a valid token with insufficient permission. Those are different states and should produce different HTTP outcomes. Keep error bodies generic and omit raw authorizer or Lambda errors.
8. **Safe test client:** Keep a test token in process memory and send it only as the Authorization header to the local test calls. Never print it, commit it, or include it in captured command output. Use synthetic application-user passwords, distinct from AWS credentials.
9. **Evidence:** A template proves intended configuration, not runtime behavior. Pair resource inspection with live requests from each identity state: no token, invalid token, ordinary member, and operations member.
10. **Cost and teardown:** API Gateway requests and log ingestion are usage-based; user-pool pricing depends on the configured feature tier and current account pricing. Keep traffic low, avoid add-on features, and delete the stack promptly.

## Questions to answer while working

- Which component authenticates the application user? Which component decides whether the user is allowed to call the operations route?
- How is the caller's `sub` claim protected from a value supplied by the caller?
- What is the difference between a Cognito application user and the Lambda execution-role principal?
- What does API Gateway's request validator check, and what still needs validation or handling in the function?
- Why should an invalid token and an authenticated user lacking the required group produce different outcomes?
- What runtime evidence proves both the allowed and denied paths work?
- Which data is safe to log for diagnosis, and which data must stay out of logs and evidence?

Levels 2–4 are not active and have no implementation hints here.
