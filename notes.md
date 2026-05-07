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
- 