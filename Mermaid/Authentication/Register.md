```mermaid
sequenceDiagram
    participant Client
    participant AuthEndpoint
    participant RegisterUserCommand
    participant IdentityService
    participant UserManager
    participant Database

    Client->>AuthEndpoint: POST /api/Authentication/Register
    AuthEndpoint->>RegisterUserCommand: Send(command)
    RegisterUserCommand->>IdentityService: CreateUserAsync(email, password)
    IdentityService->>UserManager: FindByEmailAsync(email)
    UserManager->>Database: Query UserAccount
    Database-->>UserManager: UserAccount/null
    alt User exists
        IdentityService-->>RegisterUserCommand: Error: ACCOUNT_EMAIL_ALREADY_EXISTS
        RegisterUserCommand-->>AuthEndpoint: Error (ACCOUNT_EMAIL_ALREADY_EXISTS)
        AuthEndpoint-->>Client: 400 Bad Request (ACCOUNT_EMAIL_ALREADY_EXISTS)
    else User doesn't exist
        IdentityService->>UserManager: CreateAsync(userAccount, password)
        UserManager->>Database: Insert UserAccount + User
        Database-->>UserManager: Success
        UserManager-->>IdentityService: Success
        IdentityService-->>RegisterUserCommand: Success + UserId
        RegisterUserCommand-->>AuthEndpoint: Success
        AuthEndpoint-->>Client: 200 OK + UserId
    end
