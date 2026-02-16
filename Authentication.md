# Authentication

## Admin

System Admin is authenticated with a password. Spring Boot provides the authentication flow and endpoint.

## Telegram's mini-app

The flow:
1. Mini app gets initData from Telegram bot
2. Mini app sends a request to the backend with the initData
3. Backend verifies Telegram signature (using bot token secret)
4. Backend finds the ModeratorProfile in the database matching the Telegram ID (and connected user ID)
5. The backend generates a JWT session token, signs it with the backend secret: 
    ```json
    {
      "sub": "internal_user_id",
      "telegram_id": 123456789,
      "iat": 1700000000,
      "exp": 1700003600
    }
    ```
6. The backend sends it to the mini app for authentication:
    ```json
    {
      "access_token": "eyJhbGciOi..."
    }
    ```
7. The mini app stores the JWT session token in the local storage and future API calls use JWT
    ```text
    GET /api/v1/...
    Authorization: Bearer eyJhbGciOi...
    ```

This is not implemented yet but may be in the future: 
- The token refresh pattern (access token and refresh token)
- Prevent replay attacks: store the last auth_date per user and reject older ones
