## Learning FastAPI fundamentals by building a blog app

The tutorial resource can be found [here](https://www.youtube.com/watch?v=iukOehU5aF4).

### Video 1 - "Getting Started - Web App + REST API"
#### Notes

- FastAPI autogenerates API documentation in OpenAPI format.
- In dev mode, the docs can be accessed from _localhost:8000/docs_
- Endpoints decorators can be stacked in FastAPI allowing for multiple endpoints to result in the same response:

```python
@app.get("/posts", response_class=HTMLResponse, include_in_schema=False)
@app.get("/api/posts-html", response_class=HTMLResponse, include_in_schema=False)
def get_posts_html():
    return f"<h1>{posts[0]["title"]}</h1>"
```

- The endpoints that are added to the auto-generated API doc can be regulated with flags.
- An example of this is shown above with the __include_in_schema__ flag.

- Command to start the fastAPI application in dev mode:
```bash
uv run fastapi dev main.py
```

### Video 2 - "HTML Frontend for Your API - Jinja2 Templates"
#### Notes

- Similar to Flask, static HTML template pages can be created using Jinja2 in FastAPI.
- Need to create a directory called **templates** and all static HTML templates need to be added there.
- FastAPI will scan that subdirectory for the required template.
- The templates directory is loaded into the application as follows:

```python
from fastapi.templating import Jinja2Templates

templates = Jinja2Templates(directory="templates")
```

- An API endpoint returning an HTML page generated from a template must return a templates.TemplateResponse() object.
- Three main arguments need to be passed in for creating the TemplateResponse instance:
  - The incoming Request object
  - The name of the template file in the templates directory
  - The context to be rendered on the HTML page.
- Similar to jsx, jinja2 allows for loops and conditionals to be incorporated into the template files, allowing for conditional rendering.
- A simple layout file would look something like this:

```html
<!DOCTYPE html>
<!-- All html content that will be common between different html pages that will inherit this layout -->
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>
            {% if title %}
                FastAPI Blog - {{ title }}
            {% else %}
                FastAPI Blog
            {% endif %}
        </title>
    </head>
    <body>
    <!-- All variable content will go inside the block content -->
        {% block content %}
        {% endblock content %}
    </body>
</html>
```

- A html file that inherits from the layout file would look like this:

```html
{% extends "layout.html" %}
<!-- In each HTML page inheriting from a layout, only added the context that is unique to the page inside block content -->
{% block content %}
    <h1>Home Page</h1>
    <p>This is using a template!</p>
    {% for post in posts %}
        <h2>{{ post.title }}</h2>
        <p>{{ post.content }}</p>
    {% endfor %}
{% endblock content %}
```

##### Styling the HTML document
- CSS files, images and other elements to style and decorate the HTML pages must be added to a directory called **static** within the project.
- Static files are mounted onto the application in FastAPI as follows:

```python
from fastapi.staticfiles import StaticFiles

app.mount("/static", StaticFiles(directory="static"), name="static")
# First argument is the subdirectory path
# Second argument is a StaticFiles object with paths to all static files
# Third argument is the name used to refer to this mounted application within the FastAPI app.
```

### Video 3: Path Parameters - Validation and Error Handling
#### Notes

- URL endpoints can be made dynamic by rendering a different resource based on the passed-in parameter.
- Example of a dynamic URL used: /api/posts/{post_id}
- Here, passing in a different id will render the data linked to that ID.
- Can create a "post.html" which can be renderend as a template response to such a dynamic URL.
- To handle HTTP errors, can use FastAPI's exception handler wrapper.

```python
# wrapper for the exception handling function:
# Need to import HTTPException from fastapi or from starlette.exceptions

@app.exception_handler(ExceptionHandlingClass)
def exception_handling_function():
    pass
```

- Validation errors are handled differently to HTTP errors.
- For example, if {post_id} is defined as an integer and the user passes in a string, then this would classify as a validation error.
- These errors are handled with the following import:

```python
from fastapi.exceptions import RequestValidationError

@app.exception_handler(RequestValidationError)
def validation_exception_handler(request: Request, exception: RequestValidationError):
    if request.url.path.startswith("/api"):
        return JSONResponse(
            status_code=status.HTTP_422_UNPROCESSABLE_CONTENT,
            content={"detail": exception.errors()},
        )

    return templates.TemplateResponse(
        request,
        "error.html",
        {
            "status_code": status.HTTP_422_UNPROCESSABLE_CONTENT,
            "title": status.HTTP_422_UNPROCESSABLE_CONTENT,
            "message": "Invalid request. Please check your input and try again.",
        },
        status_code=status.HTTP_422_UNPROCESSABLE_CONTENT,
    )

# For api calls, can return the raw exeception.errors() as content
# For page renders, create a dictionary of key-value pairs to feed to the error.html template.
# For page renders, use the status code 422
```

### Video 4: Pydantic Schemas - Request and Response Validation
#### Notes

- Pydantic is built into FastAPI, so the structure of data from incoming requests and for outgoing responses can be validated at runtimne.
- This helps ensure data consistency and reliability.
- Create custom schemas for requests and responses and import them into the py file containing the API functions.
- In the HTTP method definition, add the *response_model* argument to trigger pydantic validation.

```python
@app.get("/api/posts", response_model=list[PostResponse])
def get_posts():
    return posts
```

- Missing fields or incorrect data for any fields will trigger a 422 error code.
- This can be tested using OpenAPI or other API testing tools like Postman.

### Video 5: Adding a Database - SQLAlchemy Models and Relationships
#### Notes

- Will be using sqlite3 to start with, and DB interactions will be coded using sqlalchemy.
- Sqlalchemy will allow for the database to be changed with minimal change to the db-interaction code.
- In a typical Python fullstack application, you have Pydantic contracts to validate the data being exchanged at the presentation and business layer, and you have sqlalchemy contracts to validate data being exchanged at the persistence and DB layers.
- Why not have one single contract being used in all the layers? 
  - SQLModel, a library made by the same devs that made FastAPI allows for this.
  - But, separation of concerns is better.
  - And its industry standard to use SQLAlchemy for backends with Python.
- The sqlalchemy session should be accessed as a context manager, to ensure DB connections are closed after a transaction.
- Define tables, their fields and their relationships as classes.
- These 'table' classes inherit from DeclarativeBase from sqlalchemy.orm

```
In SQLAlchemy, DeclarativeBase is the modern foundation for defining your database models. It serves as the "parent" class that links your Python classes to actual database tables.

Think of it as the translator that tells SQLAlchemy, "This Python class isn't just a regular object; it’s a blueprint for a specific table in my database.

Key Benefits
Type Safety: When used with Mapped and mapped_column, it provides excellent support for type checkers like Mypy or Pyright.

Centralization: It keeps all your schema definitions in Python code (Single Source of Truth), so you don't have to write raw SQL CREATE TABLE statements.

Introspection: Because all models share the same base, you can easily reflect or inspect your entire database schema through the Base.metadata object.
```

- When defining a table schema, use the relationship() method to define a JOIN operation between tables.
- The **back_populates** flag allows for Python to synchronise the relationship between tables without needing to refresh the session from the database.

##### How to pass a DB session into persistence layer or business layer operations?
- Use the Depends method import from fastapi
- This allows for injecting the DB session into functions.

##### Table creation
- When the app is initialised, this is handled by the following line of code:
```python
Base.metadata.create_all(bind=engine)

# Refer to main.py and database.py
```
- This operation is idempotent. If the table already exists, it will not be over-ridden by the creation of a new one.
- But how is database migration handled using sqlalchemy, if the table schema is created based on the python code?
- This is handled by another python library called Alembic. This will be covered in a future lesson.

- Key thing to remember about how to route method to jinja templates correctly:
```python
@app.get("/users/{user_id}/posts", include_in_schema=False, name="user_posts")
def user_posts_page(
    request: Request,
    user_id: int,
    db: Annotated[Session, Depends(get_db)],
):
pass
```
- To reference this route in a jinja template, "user_posts" needs to be used, not the name of the function!
- So, in the HTML template, access the route and its attributes as follows:
```html
    <a class="me-2" href="{{ url_for('user_posts', user_id=post.author.id) }}">{{ post.author.username }}</a>
```

### Video 6: Completing CRUD - Update and Delete (PUT, PATCH, DELETE)
#### Notes

- PUT is used to change the entire entry in the db table.
- PATCH is used to changed particular field values on the db entry.
- Since the PostCreate schema that we created requires all mandatory fields to be entered by the client for the API request to be correctly validated by pydantic, it cannot be used with PATCH, only with PUT.
- Need to create a new data validation schema to use with PATCH (PostUpdate) where the data fields are optional. This will allow the client to only provide the fields that they wish to change, while still performing input validation with Pydantic.
- For updating an entry using PATCH, using the following lines of code with sqlalchemy:

```python
    # post_data is a pydantic schema model inheriting from BaseModel
    # the "exclude_unset=True" flag is essential to ensure that only the fields provided by the client are updated.
    # Without this flag, other fields not provided by the client would be changed to schema defaults, which is not the expected behaviour

    update_data = post_data.model_dump(exclude_unset=True)
    for field, value in update_data.items():
        setattr(post, field, value)
```
- When deleting a user, a design decision needs to be made:
  - Do you want to delete the user but preserve the posts created by them? (Example of this is Reddit)
  - Do you want to delete the user and all the posts made by them? (Example of this woule be Instagram)
- In this project, the latter decision is made. This will require recursively deleting all posts made by the user.
- In sqlalchemy, this is achieved with a simple parameter added to the table schema for the "users" table

```python
class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(Integer, primary_key=True, index=True)
    username: Mapped[str] = mapped_column(String(50), unique=True, nullable=False)
    email: Mapped[str] = mapped_column(String(120), unique=True, nullable=False)
    image_file: Mapped[str | None] = mapped_column(
        String(200),
        nullable=True,
        default=None,
    )

    posts: Mapped[list[Post]] = relationship(
        back_populates="author",
        cascade="all, delete-orphan"
    )
    # The cascade flag here will delete all entries in the posts table linked to a particular
    # user in the user table.
```
- The "cascade" parameter in the JOIN command linking "users" to "posts" achieves the above goal.

### Video 7: Sync vs Async - Converting Your App to Asynchronous
#### Notes

- Added a new dependency: aiosqlite
- This allows for asynchronous interactions with sqlite3 databases.
- Ensure that "greenlet" is also installed. This is needed to sqlalchemy's async mode!

##### Lazy loading vs eager loading and how it impacts sqlalchemy and fastAPI

```
1. Lazy Loading (The Default "Trap")
In synchronous SQLAlchemy, lazy loading is the default. When you access a relationship (like user.posts), SQLAlchemy emits a SQL query at that exact moment to fetch the data.

The Async Problem:
In an async environment (like FastAPI with AsyncSession), lazy loading is disabled by default.
If you try to access user.posts without pre-fetching it, SQLAlchemy will raise a MissingGreenlet error. This is because accessing an attribute is a synchronous action, but fetching data from a database requires an awaitable network call.

Pro: Saves memory if you don't use the relationship.

Con: Causes errors in async code unless using a specific (and often discouraged) "async lazy" proxy.

Con: N+1 Problem: If you load 10 users and then lazy-load their posts, you trigger 1 extra query per user (11 queries total).

2. Eager Loading (The Async Standard)
Eager loading tells SQLAlchemy to fetch the related data in the same initial query (or immediately after). This is the required approach for 90% of FastAPI async use cases.

There are two primary ways to do this:

A. Joined Loading (joinedload)
This uses a SQL LEFT OUTER JOIN to get everything in one single result set.

Best for: Many-to-One or One-to-One relationships (e.g., fetching a Post and its Author).

SQL Example: SELECT * FROM posts JOIN users ON ...

B. Selectin Loading (selectinload)
This emits two queries: one for the main objects and one for all related objects using an IN clause.

Best for: One-to-Many or Many-to-Many relationships (e.g., fetching a User and all their Posts). It is generally more efficient than joinedload for collections.

SQL Example:

SELECT * FROM users;

SELECT * FROM posts WHERE user_id IN (1, 2, 3...);
```
- An example eager loading:

```python
from sqlalchemy.orm import selectinload
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select

@app.get("/users/{user_id}")
async def get_user(user_id: int, db: AsyncSession = Depends(get_db)):
    # We explicitly tell SQLAlchemy to fetch 'posts' immediately
    result = await db.execute(
        select(User)
        .options(selectinload(User.posts)) 
        .where(User.id == user_id)
    )
    user = result.scalar_one_or_none()
    return user # This now includes the posts list safely
```
- db.add() operations do not require async await.
- Other db operations (db.commit(), db.refresh(), db.delete()) **do** require await.

##### Update to the db.refresh() call to account for async
```
In your FastAPI code, await db.refresh(new_post, attribute_names=["author"]) is acting as a targeted eager load immediately after the database transaction.

Here is the technical breakdown of why it's there and what it’s doing:

1. The Post-Commit "Expired" State
When you run await db.commit(), SQLAlchemy expires all objects currently tracked in the session. This is a safety measure; it assumes that since the transaction is finished, the data in your Python objects might be stale compared to the database.

If you tried to return new_post immediately after the commit, your Pydantic model (PostResponse) would try to access new_post.author. Since the object is "expired," SQLAlchemy would try to lazy-load the author, which—as we discussed—crashes in async sessions.

2. What attribute_names=["author"] specifically does
The refresh method tells SQLAlchemy: "Go back to the database right now and update this object's data."

By adding attribute_names=["author"], you are giving it two specific instructions:

Update the Post: Refresh the basic columns of new_post (like id and date_posted) which were generated by the database.

Eager Load the Relationship: Specifically fetch the related User object (the author) in the same refresh operation.
```

- Without this new parameter, in async mode, as lazy loading is disabled, sqlalchemy would only fetch fields from the "posts" table, when it hits the "author" field, it would show up as empty, and Python would throw an error.
- SQL syntax of "await db.refresh(new_post, attribute_names=["author"])":

```SQL
SELECT posts.*, users.* 
FROM posts 
JOIN users ON users.id = posts.user_id 
WHERE posts.id = [new_id];
```

### Video 8: Routers - Organizing Routes into Modules with APIRouter

- Routers allows for the organisation of related API endpoints into separate python files.
- These files can be imported into the main app file to call the different endpoints.
- This helps seperate related code and keep the code repository organised.
- This is similar to blueprints from Flask.
- In order to create routes, need to import *APIRouter* from fastapi

### Video 9: Frontend Forms - Connecting JavaScript to Your API
#### Notes

- For jinja templates to work with html script tags, need to place the js inside:
```html
{% block scripts %} 
<script>
    console.log("Put js code here")
</script>
{% endblock scripts %}
```
- This is a block for scripts defined in layout.html

### Video 10: Authentication - Registration and Login with JWT
#### Notes

- Need to install a password hashing algorithm.
- The two viable options for Python are bcrypt and argon2.
- Argon2 is considered the superior hashing algorithm as it is designed to be resistant to GPU based attacks.
- Argon2 is used in this project with pwdlib

```bash
uv add "pwdlib[argon2]"
```

- For working with json web tokens, the project will be using pyjwt.
- For working with environment variables, using pydantic-settings.

##### Pydantic-settings vs python-dotenv

- dotenv is for working with .env files and environment variables.
- pydantic-settings (ps) is used to manage configuration variables for the entire application.
- With ps, environment variables can be mapped to type-safe objects.
- ps also fails early if config values are incorrect.
- Full autocomplete support in the IDE as well.

##### Limitations of SQLAlchemy

- In video 10, we need to update the "users" table in the database to include a password-hash field.
- This would typically require a database migration, but this is not possible in sqlalchemy.
- So, for the content covered in video 10, we will have to delete the database file, update the table schema in our Python code, then the next time the table is created by the sqlalchemy engine, it will include the password-hash file.
- This limitation will be addressed in a future video, covering another Python library, Alembic, which allows for database migrations to be performed.

##### Generating a JSON web token

- JWTs contain three parts:
  1. Header: The header includes the signing algorithm and the token type.
  2. Payload: The payload contains the claims or the JSON object.
  3. Signature: A string that is generated via a cryptographic algorithm that can be used to verify the integrity of the JSON payload.
- In the "create_access_token" method in auth.py, all three components are being passed into the jwt.encode() method.
- To decode the jwt, need to pass four parameters:

```python
    payload = jwt.decode(
        token,
        settings.secret_key.get_secret_value(),
        algorithms=[settings.algorithm],
        options={"require": ["exp", "sub"]},
    )
    # secret_key is the SECRET_KEY in our .env file.
```

- Good practice to make string entries and string comparisons case insensitive.
- With a normal python String object, we can call the lower() method.
- To lower case the string stored in an SQL DB, need to use the "func" import from sqlalchemy, which is a FunctionGenerator.
- It is a middleware object that allows you to call any SQL method like LOWER, UPPER, etc.

##### JWT in the frontend
- JWT tokens should be stored in local storage. This is considered the standard practice.
- But local storage variables are vulnerable to XSS (cross site scripting).
- For this reason, its considered good practice to put a time limit on the validity of the token.
- We have done this in the auth.py file.
- When the user logs out, the JS code should clear the localstorage of the JWT token.

### Video 11: Authorization - Protecting Routes and Verifying Current User
#### Notes

- Whether or not the authenticated user has the 'authorisation' to edit, delete a post will be determined by cross-referencing the userID of the authenticated user to the userID linked to the posts.
- If the two IDs do not match, then this particular user is not authorised to make any changes to the particular post in question.
- The authrisation checks will use the "sub" attribute encoded into the token payload.
- The sub key contains a stringified version of the UserID that was generated by the database.
- So, we can use the verifiy_access_token() method defined in auth.py to retrieve the ID from the token and then perform user authorisation checks.

```
Difference between HTTP codes 401 (Unauthorised) and 403 (Forbidden)

- Use 401 when you want the end user to know that a particular action was not completed because they weren't logged in.
- Use 403 when you want the user to know that authentication wasn't a problem, they are logged in correctly, but they are not allowed to perform the operation that was attempted.
```
- If everythin is working correctly, then in the frontend, when one user clicks on another user's post, the Update/Delete buttons should not be visible to them.
- And while using the API, if a user attempts to delete another user's post, they would receive a HTTP 403 response from the server.

### Video 12: File Uploads - Image Processing, Validation, and Storage
#### Notes

- Downloading/Uploading large files will require streaming multiple payloads over the HTTP connection.
- To do this, the Python-Multipart package is required.
- If fastapi is installed with the correct flags (as we did in this tutorial series), then this package should have been installed.
- If it isn't, then install manually for this video tutorial.
- Multipart form data is the object that can be streamed over HTTP. Need to set the content type to this when sending a file over HTTP.
- FormData object should be created in the frontend for sending files over HTTP, not JSON.
- In our backend function to handle file upload, we have created the function parameter called 'file'.
- The FormData **must** contain the File object appended to the FormData object with the key 'file'. Otherwise the backend function will not work correctly.
- As image processing is a CPU bound task, we do not want to await this function execution directly in the event loop. This would block the entire event loop, negating the benefits of async code execution.
- Instead, we want to pass the image processing function to "run_in_threadpool" imported from starlette.
- This will offload the image processing to another thread, and the event loop can track the status of that thread in a non-blocking fashion.
- 

### Video 13: Pagination - Loading More Data with Query Parameters
#### Notes

- populate_db.py helper script created to generate enough posts to be able to test the pagination functionality.
- Also create the "populate_images" directory with the necessary test profile pics. 
- Need to setup a contract between the frontend and the API for the paginated response.
  - Things like how many posts to return per request.
  - How should the posts be ordered? By author, by create date?
  - This contract needs to be added in schemas.py
- It looks something like this:

```python
## Paginated Post Response Schema
class PaginatedPostsResponse(BaseModel):
    posts: list[PostResponse]
    total: int
    skip: int
    limit: int
    has_more: bool

# Total represents all the posts in the DB
# Skip represents how many rows of the DB need to be skipped before selecting the rows for the query
# Limit represents how many rows of data to send per request
# has_more is a boolean which the frontend can use to disable the "load more" button.
# The orderby operation is crucial for consistent pagination behaviour.
```
```
why didn't he use the "skip" argument for pagination in the user_posts_page endpoint?

In the user_posts_page route (31:26), the creator is implementing server-side rendering for the initial page load.

He did not include the skip argument in this specific template route because the homepage and user-specific post pages are intended to always start from the first batch of results (offset 0) when a user first navigates to that URL.

Here is why the implementation differs slightly from the API endpoint:

Initial Load: When a user visits the URL (e.g., /user/1), the server fetches the first 10 posts to render the page immediately. Because it is always the first page, there is no need to skip any records.
Client-Side Fetching: Subsequent batches are handled by JavaScript (26:36 - 28:35). The JavaScript logic maintains the current_offset state (27:01), which keeps track of how many posts have already been loaded. When the user clicks the "Load More" button, the JavaScript sends a request to the API, which does accept skip and limit parameters to fetch the next set of data.
In short: the server-side template route is hardcoded to provide the initial state, while the dynamic API endpoint provides the flexible data retrieval for pagination.
```

- I ran into an issue when testing this code, with static files getting caches in the frontend.
- This was preventing the "Load More" button in the updated home.html from working.
- This can be resolved by configuring fastapi to instruct the browser to not cache static files.
- How to achieve this:

```

To stop FastAPI from caching your static files, you can intercept the response headers whenever a file is requested. By default, FastAPI doesn't set strict cache-control headers, which lets browsers make their own assumptions and aggressively cache files.

You can fix this cleanly by creating a custom subclass of FastAPI's `StaticFiles` that forces the browser to check for changes every single time.

Here is exactly how to set it up:

```python
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles

app = FastAPI()

class NoCacheStaticFiles(StaticFiles):
    """
    A custom StaticFiles handler that injects headers to prevent 
    the browser from aggressively caching frontend assets.
    """
    async def get_response(self, path: str, scope):
        response = await super().get_response(path, scope)
        
        # Inject headers to tell browsers never to store or reuse a cached version
        response.headers["Cache-Control"] = "no-cache, no-store, must-revalidate"
        response.headers["Pragma"] = "no-cache"
        response.headers["Expires"] = "0"
        
        return response

# Mount your static files using the new custom class instead of the default StaticFiles
app.mount("/static", NoCacheStaticFiles(directory="static"), name="static")

```

### What these headers actually do:

* **`no-store`**: Tells the browser (and any intermediate caches/proxies) not to save a copy of the file to the local disk at all.
* **`no-cache` / `must-revalidate**`: Forces the browser to validate with your FastAPI server before using any previously downloaded version of the file.
* **`Expires: 0`**: Marks the asset as instantly expired so it is never considered "fresh."

### A Quick Warning for Production

While turning off caching is perfect for development so you can see your JavaScript updates instantly, it's a bad idea for a live production server. It forces your users to re-download every image, CSS, and JS file on every single page click, slowing down your app and wasting your server's bandwidth.

If you want to keep caching active for production but disable it safely for your local machine, you can toggle it using an environment variable:

```python
import os
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles

app = FastAPI()

# Check if you are running locally or in production
IS_DEVELOPMENT = os.getenv("ENVIRONMENT", "development") == "development"

if IS_DEVELOPMENT:
    # Use the custom no-cache class we built above
    app.mount("/static", NoCacheStaticFiles(directory="static"), name="static")
else:
    # Use standard FastAPI static files (enables normal caching)
    app.mount("/static", StaticFiles(directory="static"), name="static")

```
- **Don't do this for production.**


### Video 14: Password Reset - Email, Tokens, and Background Tasks
#### Notes

- Following good security practices is absolutely crucial to setting up a good password reset function, and many background tasks alike.
- The password reset link is emailed to the user. To implement this, we will need to use the SMTP protocol.
- In Python, the async smtp library is aiosmtplib.
- Python's builtin SMTP library is synchronous, so it cannot be used in an async application.
- For testing emails, need a sandbox.
- In this tutorial, [mailtrap](https://mailtrap.io/home) free tier is used to sandbox testing of sending emails.
- Can create an email template in Jinja2, similar to other templates that get rendered in the browser.
- The difference is that these templates connected to an HTTP request. So cannot use TemplateResponse to serve the email HTML.
- Need to get the template as an environment variable and the call the render() method on the Jinja2Templates object.
- Use in-line CSS in HTML templates sent over SMTP as CSS files can get ignored. In some cases, all forms of CSS might get ignored.
- 