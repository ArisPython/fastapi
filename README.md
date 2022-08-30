# Fastapi
- ToDo App
    - cd todo
    - source venv/bin/activate
    - todos: uvicorn api:app --host=0.0.0.0 --port 8000 --reload
    - http://127.0.0.1:8000/docs
    - http://127.0.0.1:8000/todo

- Planner App
    - cd planner
    - source venv/bin/activate
    - echo DATABASE_URL=mongodb://localhost:27017/planner >> .env
    - mongod --dbpath store
    - python main.py
    - http://127.0.0.1:8080/docs