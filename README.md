=====================================================================
RACEDAY API  (WebApplication2)
=====================================================================

A RESTful ASP.NET Core Web API for managing race events (runs, walks
and cycles), user accounts, event enrolments and race results.

There are two user roles:

  Organiser    Creates and manages events and categories, and
               records results.
  Participant  Registers, browses events, enrols in an event
               category, cancels enrolments and views results.


---------------------------------------------------------------------
1. TECHNOLOGY
---------------------------------------------------------------------

  Framework       ASP.NET Core, .NET 8.0 (Web API with controllers)
  Database        SQL Server via Entity Framework Core 8 (code first)
  Auth            Server-side session (cookie), role-based checks
  Passwords       SHA256 hash, Base64 encoded
  API docs        Swagger / OpenAPI (Swashbuckle 6.6.2)
  Tests           xUnit 2.6.6, Moq 4.20, EF Core InMemory provider
  CI              GitHub Actions (build and test on Windows)


---------------------------------------------------------------------
2. FOLDER LAYOUT
---------------------------------------------------------------------

  WebApplication2/                    Solution root (this folder)
    WebApplication2.sln               Visual Studio solution
    README.md                         Original readme (Markdown)
    *.md                              Extra guides (see section 9)
    run-api.bat,
    start-api-complete.bat,
    start-api.ps1, start-api.sh       Helper scripts to build and run
    diagnostic.bat                    Environment / connection check
    .github/workflows/
      build-and-test.yml              CI pipeline

    WebApplication2/                  The API project
      Program.cs                      Startup, DI, session, CORS, Swagger
      appsettings.json                Connection string and logging
      appsettings.Development.json    Development overrides
      WebApplication2.http            Sample requests for VS / VS Code
      Properties/launchSettings.json  Ports and launch profiles
      Controllers/                    HTTP endpoints
        AuthController.cs             Register, login, logout, status
        UsersController.cs            Profile and user lookup
        EventsController.cs           Event CRUD
        CategoriesController.cs       Categories within an event
        EnrolmentsController.cs       Enrolments within an event
        ResultsController.cs          Results within an event
      Models/                         Entities: User, Event, Category,
                                      EventEnrolment, Result
      DTOs/                           Request / response objects
      Data/RaceDayDbContext.cs        EF Core database context
      Services/AuthenticationService.cs
                                      Password hashing and verification

    WebApplication2.Tests/            Unit tests (xUnit)
      Controllers/                    Auth, Events, Enrolments, Results
      Services/                       AuthenticationService tests
      Data/                           DbContext tests (in-memory DB)

  Note: the .vs folder and any bin/obj folders are Visual Studio and
  build output. They are not source code and can be deleted.


---------------------------------------------------------------------
3. DATA MODEL
---------------------------------------------------------------------

  User            Id, FirstName, LastName, Email, PhoneNumber,
                  PasswordHash, Role (Organiser | Participant),
                  CreatedAt, UpdatedAt

  Event           Id, Name, Description, EventDate, Location,
                  Distance, EventType (Run | Walk | Cycle),
                  OrganiserId, CreatedAt, UpdatedAt

  Category        Id, Name, Description, EventId, CreatedAt
                  (for example an age group or distance for an event)

  EventEnrolment  Id, ParticipantId, EventId, CategoryId, EnrolledAt

  Result          Id, ParticipantId, EventId, FinishTime,
                  FinishingPosition, RecordedAt

  Relationships: an Event has many Categories, Enrolments and
  Results. A User has many Enrolments and Results. An Enrolment links
  one Participant to one Event and one Category.


---------------------------------------------------------------------
4. REQUIREMENTS
---------------------------------------------------------------------

  - .NET 8.0 SDK              (check with: dotnet --version)
  - SQL Server, SQL Server Express or LocalDB
  - Visual Studio 2022 (recommended) or any editor plus the dotnet CLI
  - EF Core tools for the database step:
        dotnet tool install --global dotnet-ef


---------------------------------------------------------------------
5. GETTING STARTED
---------------------------------------------------------------------

  1. Unzip the folder and open a terminal in the solution root
     (the folder containing WebApplication2.sln).

  2. Check the connection string in
     WebApplication2/appsettings.json. The default is:

         Server=(localdb)\mssqllocaldb;Database=RaceDayDb;
         Trusted_Connection=true;

     Change it if you use a different SQL Server instance.

  3. Restore and build:

         dotnet restore
         dotnet build

  4. Create the database. The project does not include migrations
     and does not create the database on startup, so create them
     first. Run from the solution root:

         dotnet ef migrations add InitialCreate --project WebApplication2
         dotnet ef database update --project WebApplication2

     (In Visual Studio, use Package Manager Console:
      Add-Migration InitialCreate, then Update-Database.)

  5. Run the API:

         dotnet run --project WebApplication2

     Or use one of the helper scripts in the solution root
     (run-api.bat, start-api.ps1 or start-api.sh).

  6. Open Swagger in a browser (Development environment only):

         http://localhost:5014/swagger
         https://localhost:7071/swagger

     Ports come from Properties/launchSettings.json. The IIS Express
     profile uses http://localhost:10277 (SSL port 44361).


---------------------------------------------------------------------
6. API ENDPOINTS
---------------------------------------------------------------------

All routes start with /api. Full request and response details are in
API_DOCUMENTATION.md.

  Authentication
    POST   /api/auth/register
    POST   /api/auth/login
    POST   /api/auth/logout
    GET    /api/auth/status

  Users
    GET    /api/users/profile
    PUT    /api/users/profile
    GET    /api/users/{id}

  Events
    GET    /api/events
    GET    /api/events/{id}
    POST   /api/events
    PUT    /api/events/{id}
    DELETE /api/events/{id}

  Categories
    GET    /api/events/{eventId}/categories
    GET    /api/events/{eventId}/categories/{id}
    POST   /api/events/{eventId}/categories
    PUT    /api/events/{eventId}/categories/{id}
    DELETE /api/events/{eventId}/categories/{id}

  Enrolments
    GET    /api/events/{eventId}/enrolments
    GET    /api/events/{eventId}/enrolments/{id}
    POST   /api/events/{eventId}/enrolments
    DELETE /api/events/{eventId}/enrolments/{id}

  Results
    GET    /api/events/{eventId}/results
    GET    /api/events/{eventId}/results/{id}
    GET    /api/events/{eventId}/results/participant/{participantId}
    POST   /api/events/{eventId}/results
    PUT    /api/events/{eventId}/results/{id}
    DELETE /api/events/{eventId}/results/{id}


---------------------------------------------------------------------
7. AUTHENTICATION AND ACCESS
---------------------------------------------------------------------

  1. Register with POST /api/auth/register, choosing a role.
  2. Log in with POST /api/auth/login. The API stores the user id and
     role in a server-side session and returns a session cookie.
  3. Send that cookie with later requests. Swagger and browsers do
     this automatically; with curl use a cookie jar (-c / -b).
  4. Sessions expire after 30 minutes of inactivity.

  Creating, updating or deleting events is restricted to Organisers
  (the controller checks the role stored in the session). Other
  endpoints require a logged-in user.


---------------------------------------------------------------------
8. RUNNING THE TESTS
---------------------------------------------------------------------

  From the solution root:

      dotnet test

  There are about 32 tests covering the Auth, Events, Enrolments and
  Results controllers, the AuthenticationService and the database
  context. They use an in-memory database, so SQL Server is not
  needed to run them.


---------------------------------------------------------------------
9. OTHER DOCUMENTS IN THIS FOLDER
---------------------------------------------------------------------

  QUICK_START.md             Short setup steps
  SETUP_GUIDE.md             Detailed setup and troubleshooting
  API_DOCUMENTATION.md       Endpoint reference
  ARCHITECTURE.md            Design overview
  PROJECT_STRUCTURE.md       File-by-file description
  IMPLEMENTATION_SUMMARY.md  What was built
  DOCUMENTATION_INDEX.md     Index of all documents
  SUBMISSION_CHECKLIST.md    Submission checklist
  HTTP_SERVER_CONFIGURATION.md, HTTP_QUICK_CONNECT.md,
  HTTP_CONNECTION_STATUS.md  HTTP / HTTPS connection notes
  DIAGNOSTIC_REPORT.md, FIX_SUMMARY.md, ACTION_PLAN.md
                             Troubleshooting history


---------------------------------------------------------------------
10. KNOWN LIMITATIONS AND SECURITY NOTES
---------------------------------------------------------------------

  This project is suitable for coursework and development. Before
  any real deployment, consider the following:

  - Passwords use plain SHA256 with no salt. Use a salted, slow hash
    such as PBKDF2, bcrypt or Argon2 (or ASP.NET Core Identity).
  - CORS is set to allow any origin, method and header.
  - Sessions are held in memory, so they are lost on restart and do
    not work across multiple servers.
  - Swagger is only enabled in the Development environment.
  - No database migrations or seed data are included.
  - The default connection string uses LocalDB with Windows
    authentication, which suits development only.


---------------------------------------------------------------------
11. TROUBLESHOOTING
---------------------------------------------------------------------

  Cannot connect to the database
      Confirm SQL Server or LocalDB is running and the connection
      string is correct. Try: sqllocaldb info

  Invalid object name (table not found)
      The database has not been created. Run the migration commands
      in section 5, step 4.

  HTTPS certificate warning
      Run: dotnet dev-certs https --trust
      Or use the http profile (port 5014).

  Port already in use
      Change applicationUrl in Properties/launchSettings.json.

  401 or "not logged in" responses
      Log in first and make sure the session cookie is being sent.

  More help: SETUP_GUIDE.md and diagnostic.bat

  ci attachmenent
  <img width="1599" height="235" alt="image" src="https://github.com/user-attachments/assets/8d7e9fc0-2a8a-4635-a072-ee402e222017" />
  youtube link
  https://youtu.be/hvNlgGwcC0k?si=JuEMzTH7P3Qf8JIj
