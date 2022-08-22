# 1. Intro FastAPI
## Getting Started with FastAPI
- Technical Requirement
    - https://github.com/PacktPublishing/Building-Python-Web-APIs-with-FastAPI/tree/main/ch01
    - GIT instalation
    - Creating isolated development environments with Virtualenv
    ```sh
    mkdir todos && cd todos
    python3 -m venv venv
    source venv/bin/activate
    ```
    - Building a simple FastAPI application
        - install fastapi
        ```sh
        pip install fastapi uvicorn

        ```
        - create api.py
        ```py
        from fastapi import FastAPI
        app = FastAPI()

        @app.get("/")
        async def welcome() -> dict:
            return { "message": "Hello World"}
        ```
        - Run
        ```sh
        uvicorn api:app --port 8000 --reload
        curl http://0.0.0.0:8080/

        {"message":"Hello World"}
        ```

## Routing in FastAPI
- Routing in FastAPI
    - https://github.com/PacktPublishing/Building-Python-Web-APIs-with-FastAPI/tree/main/ch02/todos
    - Understanding routing in FastAPI
        - Single Route
        ```py
        #aplikasi diatas
        ...
        app = FastAPI() #--> route
        ...
        ```
    - Routing with the APIRouter class
        - `touch todo.py`
        ```py
        from fastapi import APIRouter

        # package routing dari fastapi
        todo_router = APIRouter()

        #  temporary in-app database
        todo_list = []

        # -------- TODO ROUTE ------------------
        # Input data
        @todo_router.post("/todo")
        async def add_todo(todo: dict) -> dict:
            todo_list.append(todo)
            return {"message": "Todo added successfully"}
        
        # Menampilkan data
        @todo_router.get("/todo")
        async def retrieve_todos() -> dict:
            return {"todos": todo_list}

        ```
        ```
        Note:
        - APIRouter() dan FastAPI() berfungsi sama
        - Beda dengan FastAPI(), uvicorn tidak bisa menggunakan APIRouter() untuk menampilkan app
        - APIRouter harus di definiskan dalam variabel/instance untuk bisa dikonsumsi
        ```
    - api.py
        ```py
        from todo import todo_router

        ...
        ...

        app.include_router(todo_router)
        ```
    - test run app
        ```py
        uvicorn api:app --port 8000 --reload 
        curl http://0.0.0.0:8080/ # GET request --> {"message":"Hello World"}

        ```
    - Cek GET 
        ```py
        curl -X 'GET' 'http://127.0.0.1:8000/todo' -H 'accept: application/json'

        # Hasil json
        {
        "todos": []
        }
        ```
    - Cek POST
        ```py
        curl -X 'POST' \
            'http://127.0.0.1:8000/todo' \
            -H 'accept: application/json' \
            -H 'Content-Type: application/json' \
            -d '{
            "id": 1,
            "item": "First Todo is to finish this book!"
        }'

        # Hasil json
        {
        "message": "Todo added successfully."
        }
        ```
- Validating request bodies using Pydantic models
    - add model.py
        ```py
        # Pydantic adalah library yang menghandle validasi
        from Pydantic import BaseModel
        class Todo(BaseMode):
            id: int
            item: str

        ```
    - edit todo.py supaya menggunakan model sebagai datanya
        ```py
        from model import Todo
        ...
        async def add_todo(todo: Todo) -> dict:
            ...
        ```
    - tes validasi POST --> input value = dict kosong / {} 
        ```sh
        curl -X 'POST' \
            'http://127.0.0.1:8000/todo' \
            -H 'accept: application/json' \
            -H 'Content-Type: application/json' \
            -d '{
        }'
        ```
        ```
        Hasil error:
        {"detail":[{"loc":["body","id"],"msg":"field required","type":"value_error.missing"},{"loc":["body","item"],"msg":"field required","type":"value_error.missing"}]}
        ```
    - tes validasi POST --> input value sesuai tipe data
        ```sh
        curl -X 'POST' \
            'http://127.0.0.1:8000/todo' \
            -H 'accept: application/json' \
            -H 'Content-Type: application/json' \
            -d '{
            "id": 2,
            "item": "Validation models help with input types"
        }'
        ```
        ```
        Hasil sukses:
        {"message":"Todo added successfully"}
        ```
    - Nested pydentic model (Contoh)
        ```py
        class Item(BaseModel)
            item: str
            status: str
        class Todo(BaseModel)
            id: int
            item: Item
        ```
        ```json
        //hasil
        {
            "id": 1,
            "item": {
                "item": "Nested models",
                "Status": "completed"
                }
        }
        ```
- Path and query parameters
    - Path parameters
        - todo.py
        ```py
        from fastapi import APIRouter, Path
        ...

        @todo_router.get("/todo/{todo_id}")
        async def get_single_todo(todo_id: int = Path(..., title="The ID of the todo to retrieve.")) -> dict:
            for todo in todo_list:
                if todo.id == todo_id:
                    return {
                        "todo": todo
                    }
            return {
                "message": "Todo with supplied ID doesn't exist."
            }

        ```
        ```
        Note:
        {todo_id} adalah path parameter, sehingga memungkinkan app mencocokan todo dengan ID
        ```
        - Test GET dengan parameter 1
            ```
            curl -X 'GET' \
            'http://127.0.0.1:8000/todo/1' \
            -H 'accept: application/json'
            ```
            ```
            Hasil:
            {"todo":{"id":1,"item":"First Todo is to finish this book!"}}
            ```
    - Query Parameter
        ```
        Sifatnya opsional
        biasanya tampil di url setelah tanda ?
        ```
        ```py
        #Contoh
        async query_route(query: str = Query(None):
            return query
        ```
- Request Body
    - Pengertian
        ```
        Request body adalah data yang dikirimkan ke API menggunakan routing method (POST, UPDATE)

        Contoh:
            {
            "id": 2,
            "item": "Validation models help with input types.."
            }
        ```
    - FastAPI Automatic Docs
        - Swagger --> http://127.0.0.1:8000/docs
        - ReDoc --> http://127.0.0.1:8000/redoc
        - set json schema = sebagai format contoh pengisian json. embed config ke model.
            - model.py
            ```py
            class Todo(BaseModel):

                ...

                class Config:
                    Schema_extra = {
                        "Example": {
                        "id": 1,
                        "item": "Example schema!"
                        }
                    }


            ```

- Building a simple CRUD app (UPDATE + DELETE)
    ```
    Melanjutkan todo
    menambahkan update dan delete
    ```
    - model.py
        ```py
        # TodoItem sebagai data sumber value parameter per item
        ...

        class TodoItem(BaseModel):
        item: str

        class Config:
            schema_extra = {
                "example": {
                    "item": "Read the next chapter of the book"
                }
            }
        ```
    - UPDATE Module
        - todo.py

        ```py
        @todo_router.put("/todo/{todo_id}")
        async def update_todo(todo_data: TodoItem, todo_id: int = Path(..., title="The ID of the todo to be updated")) -> dict:
            for todo in todo_list:
                if todo.id == todo_id:
                    todo.item = todo_data.item
                    return {
                        "message": "Todo updated successfully."
                    }
            return {
                "message": "Todo with supplied ID doesn't exist."
            }
        ```
    - Tes POST (input data item baru)
        ```
         curl -X 'POST' \
        'http://127.0.0.1:8000/todo' \
        -H 'accept: application/json' \
        -H 'Content-Type: application/json' \
        -d '{   q
        "id": 1, "item": "Example Schema!"
        }'
        ```
    - Tes PUT (Update data item)
        ```
        curl -X 'PUT' \
        'http://127.0.0.1:8000/todo/1' \
        -H 'accept: application/json' \
        -H 'Content-Type: application/json' \
        -d '{
        "item": "Read the next chapter of the book."
        }'
        ```
    - DELETE module
        - todo.py
        ```py
        @todo_router.delete("/todo/{todo_id}")
        async def delete_single_todo(todo_id: int) -> dict:
            for index in range(len(todo_list)):
                todo = todo_list[index]
                if todo.id == todo_id:
                    todo_list.pop(index)
                    return {
                        "message": "Todo deleted successfully."
                    }
            return {
            "message": "Todo with supplied ID doesn't exist."
            }

        @todo_router.delete("/todo")
        async def delete_all_todo() -> dict:
            todo_list.clear()
            return {
                "message": "Todos deleted successfully."
            }

        ```
    - Tes Delete
        ```py
        ## ADD Items first
        curl -X 'POST' \
        'http://127.0.0.1:8000/todo' \
        -H 'accept: application/json' \
        -H 'Content-Type: application/json' \
        -d '{
        "id": 1,
        "item": "Example Schema!"
        }'

        ## Delete item

        curl -X 'DELETE' \
        'http://127.0.0.1:8000/todo/1' \
        -H 'accept: application/json'
        ```
## Response Models and Error Handling
- Understanding responses in FastAPI
    ```
    Response = feedback yang diterima dari interaksi dengan API route melalui HTTP Verbs. output berupa json/xml
    ```
    - Response Header = Informasi tambahan yang menyertai response body. contoh Content-Type
    - Response Body = Berisi Data yang dikirim dari server. 
    - Status code 
        ```
        1XX = request diterima server
        2XX = Request berhasil
        3XX = Request dialihkan
        4XX = Error dari sisi klien
        5XX = Error dari sisi server
        ```
- Building response models
    - model.py
        ```py
        ...
        from typing import List

        ...

        class TodoItems(BaseModel):
            todos: List[TodoItem]

            class Config:
                schema_extra = {
                    "example": {
                        "todos": [
                            {
                                "item": "Example schema 1!"
                            },
                            {
                                "item": "Example schema 2!"
                            }
                        ]
                    }
                }

        ```
        - todo.py
            ```py
            from model import Todo, TodoItem, TodoItems
            ...

            @todo_router.get("/todo", response_model=TodoItems)
            async def retrieve_todo() -> dict:
            return {
            "todos": todo_list
            }
            ```

        - Run
            ```py
            uvicorn api:app --host=0.0.0.0 --port 8000 --reload
            ```
        - Tes POST
            ```py
            url -X 'POST' \
            'http://127.0.0.1:8000/todo' \
            -H 'accept: application/json' \
            -H 'Content-Type: application/json' \
            -d '{
                "id": 1,
                "item": "This todo will be retrieved without exposing my ID!"
            }'
            ```
        - Tes GET
            ```py
            curl -X 'GET' \
            'http://127.0.0.1:8000/todo' \
            -H 'accept: application/json'
            ```
- Error Handling
    - Pengertian
        ```
        HTTPException class takes three arguments:
            • status_code: The status code to be returned for this disruption
            • detail: Accompanying message to be sent to the client
            • headers: An optional parameter for responses requiring headers
        ```
    - HTTPException --> menerapkannya ke Modul CRUD
        ```py
        from fastapi import APIRouter, Path, HTTPException, status
        
        ...
        @todo_router.get("/todo/{todo_id}")
        ...
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail="Todo with supplied ID doesn't exist",
            )
        ...

        @todo_router.put("/todo/{todo_id}")
        ...
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail="Todo with supplied ID doesn't exist",
            )
        ...

        @todo_router.delete("/todo/{todo_id}")
        ...
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail="Todo with supplied ID doesn't exist",
            )
        ```
    - Tes Menggunakan swagger
        - http://127.0.0.1:8000/docs

    - Bisa menambahkan status_code jika diinginkan
        ```py
        @todo_router.post("/todo", status_code=201)
        ```

## Templating in FastAPI
- Understanding Jinja
    - Basic
        ```py
        {% … %}         --> if/else, loops
        {{ todo.item }} --> print value dari variabel
        {# This is a great API book! #} --> comment
        ```
    - Filters
        - Basic
            ```js
            {{ variable | filter_name }}
            {{ variable | filter_name(*args) }}
            
            ```
        - Default filter
            ```js
            // Digunakan untuk output default ketika value bernilai none
            {{ todo.item | default('This is a default todo item') }}
            ```
        - Escape Filter
            ```js
            // render raw HTML output
            {{ "<title>Todo Application</title>" | escape }}
            <title>Todo Application</title>
            ```
        - Conversion Filters
            ```js
            // Konversi dari satu data type ke data type yang lain
            {{ 3.142 | int }}
            3
            {{ 31 | float }}
            31.0
            ```
        - Join Filter
            ```js
            // Mengubah elemen List[] menjadi string
            {{ ['Packt', 'produces', 'great', 'books!'] | join(' ') }}
            Packt produces great books!
            ```
        - Length Filter
            ```js
            //Menghitung panjang element, fungsi sama dengan len()
            Todo count: {{ todos | length }}
            Todo count: 4
            ```
    - If Statements
        ```js
        {% if todo | length < 5 %}
        You don't have much items on your todo list!
        {% else %}
        You have a busy day it seems!
        {% endif %}
        ```
    - Loops
        ```js
        {% for todo in todos %}
        <b> {{ todo.item }} </b>
        {% endfor %}
        ```
    - Macros
        ```js
        //use case for macros is to avoid the repetition of code and instead use a single function call
        {% macro input(name, value='', type='text', size=20 %}
        <div class="form">
            <input type="{{ type }}" name="{{ name }}" value="{{ value|escape }}" size="{{ size }}">
        </div>
        {% endmacro %}
        ```
        ```js
        {{ input('item') }}
        
        //return
        <div class="form">
            <input type="text" name="item" value="" size="20">
        </div>
        ```
    - Template Inheritance
        - https://jinja.palletsprojects.com/en/3.0.x/templates/
- Using Jinja templates in FastAPI



# Building and Securing FastAPI Applications
## Structuring FastAPI Applications
- Structuring in FastAPI applications
    - Building an event planner application
    - Implementing the models
    - Implementing routes
## Connecting to a Database
- Setting up SQLModel
    - Tables
    - Rows
    - Sessions
- Creating a database
    - Creating events
    - Read events
    - Update events
    - Delete events
- Setting up MongoDB
    - Document
    - Initializing the database
- CRUD operations
    - create
    - Read
    - Update
    - Delete
    - routes/events.py
    - routes/user.py

## Securing FastAPI Applications
- Authentication methods in FastAPI
    - Dependency injection
    - Creating and using a dependency
- Securing the application with OAuth2 and JWT
    - Hashing passwords
    - Creating and verifying access tokens
    - Handling user authentication
- Updating the application
    - Updating the user sign-in route
    - Updating event routes
    - Updating the event document class and routes
- Configuring CORS

# Testing And Deploying FastAPI Applications
## Testing FastAPI Applications
- Unit testing with pytest
- Writing tests for REST API endpoints
    - Testing the sign-up route
    - Testing the sign-in route
    - Testing CRUD endpoints
    - Testing READ endpoints
    - Testing the CREATE endpoint
    - Testing the UPDATE endpoint
    - Testing the DELETE endpoint
- Test coverage
## Deploying FastAPI Applications
- Preparing for deployment
    - Managing dependencies
    - Configuring environment variables
- Deploying with Docker
    - Writing the Dockerfile
    - Building the Docker image
    - Deploying our application locally
    - Running our application
- Deploying Docker images
    - Deploying databases