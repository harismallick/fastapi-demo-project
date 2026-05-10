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

- 