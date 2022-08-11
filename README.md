# fastapi

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
    - edit todo.py
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

- Building a simple CRUD app
    ```
    Melanjutkan todo
    menambahkan update dan delete
    ```
    - model.py
        ```py
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
        -d '{
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
