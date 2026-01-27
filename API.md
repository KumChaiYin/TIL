# Building Your Own AI API (The "Server" Side)

## The Code Structure
```python
# This is essentially what runs inside the Docker container
from fastapi import FastAPI
from pydantic import BaseModel
import my_model_logic # Your custom model code

app = FastAPI()

# 1. Load Model (Do this ONCE when app starts, not every request)
print("Loading model into RAM/GPU...")
model = my_model_logic.load_model() 

# 2. Define Input Format (Schema)
class InputData(BaseModel):
    text: str
    temperature: float

# 3. Define the Endpoint (The "URL")
@app.post("/generate")
async def predict(data: InputData):
    # 4. Inference Logic
    result = model.run(data.text, temperature=data.temperature)
    
    # 5. Return JSON
    return {"status": "success", "output": result}
```

## The Workflow (What happens under the hood)
1. Serialization: The user sends JSON. FastAPI converts that text into Python objects.
2. Inference: Your Python script runs the heavy math (Matrix Multiplication) on the CPU or GPU.
3. Deserialization: You convert the numpy array or result string back into JSON.
4. Transport: You send it back over HTTP.

