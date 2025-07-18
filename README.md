# Run DeepSeek on Android Locally using Termux + llama.cpp 

**Yes, you can run local LLMs on your Android phone — completely offline — using `llama.cpp` in Termux!**  
This guide walks you step by step through compiling `llama.cpp`, downloading quantized `.gguf` models, running Deepseek R1 7B, and even setting up a simple Chat UI.

---

## **Table of Contents**
1. [What You’ll Need](#what-youll-need)
2. [Installing Termux and Required Packages](#installing-termux-and-required-packages)
3. [Cloning and Building llama.cpp](#cloning-and-building-llamacpp)
4. [Downloading TinyLLaMA and LLaMA 2 7B Quantized Models](#downloading-models)
5. [Running the Model in CLI](#running-the-model)
6. [Setting Up a Basic Web Chat UI](#setting-up-a-basic-chat-ui)
7. [Final Thoughts & Tips](#final-tips)

---

## **What You’ll Need**

- **A relatively recent Android phone**
  - Minimum **4 GB RAM** (8 GB or more )
  - Minimum 4-core CPU (Snapdragon 7xx or 8xx preferred)
- **Termux** (Download from [F-Droid](https://f-droid.org/en/packages/com.termux/), not Play Store)
- At least:
  - ~4–5GB Free space for Deepseek model

---

## **Installing Termux and Required Packages**

Open Termux and run the following:

```bash
pkg update && pkg upgrade -y
pkg install git cmake clang build-essential wget python -y
```

Make sure you have storage access enabled:
```bash
termux-setup-storage
```

This allows file access from `~/storage` for downloaded models.

---

## **Cloning and Building llama.cpp**

```bash
# Clone the llama.cpp repo
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp

# Compile the main binary
cmake -B build
cmake --build build --config Release
```

This builds the `./main` program that runs GGUF models locally.

---

## **Downloading Models**

### **1. DeepSeek R1 - Great for Phones **

DeepSeek is multilang, lightweight and works well on mid-range phones.

**Download command:**

```bash
mkdir -p LLM
cd LLM

# Download the quantized GGUF model
wget https://huggingface.co/unsloth/DeepSeek-R1-Distill-Qwen-7B-GGUF/resolve/main/DeepSeek-R1-Distill-Qwen-7B-Q4_K_M.gguf

cd ..
```

---

### **2. DeepSeek R1 7B - Requires 8+ GB RAM (Use Q4_0 or Q5_0)**

If your phone is powerful (Snapdragon 8 Gen 1+, or 12GB RAM), try this:

```bash
mkdir -p models/llama2
cd models/llama2

# Download a 7B quantized model (Q4_0 = decent quality, smaller size)
wget https://huggingface.co/unsloth/DeepSeek-R1-Distill-Qwen-7B-GGUF/resolve/main/DeepSeek-R1-Distill-Qwen-7B-Q4_K_M.gguf

cd ../../
```

---

## **Running the Model**

You’re now ready to run the model using the CLI!

### **Run DeepSeek:**

```bash
./llama-cli -m LLM/DeepSeek-R1-Distill-Qwen-7B-Q4_K_M.gguf -p "Привет, как дела?"
```

---

## **Setting Up a Basic Chat UI (Flask-based)**

You can build a simple web-based chat UI running locally on your Android phone.

### **1. Install Flask and dependencies**

```bash
pip install flask flask-cors
```

### **2. Create a Python Script: `chat_server.py`**

```bash
nano chat_server.py
```

Paste the following:

```python
from flask import Flask, request, jsonify
import subprocess

app = Flask(__name__)

@app.route('/chat', methods=['POST'])
def chat():
    prompt = request.json.get("prompt", "")
    if not prompt:
        return jsonify({"error": "No prompt given."}), 400

    # Call llama.cpp with the prompt
    result = subprocess.run(
        ["./main", "-m", "models/tinyllama/DeepSeek-R1-Distill-Qwen-7B-Q4_K_M.gguf", "-p", prompt],
        stdout=subprocess.PIPE,
        stderr=subprocess.PIPE,
        text=True
    )
    
    return jsonify({"response": result.stdout})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=7860)
```

### **3. Run the server**

```bash
python chat_server.py
```

Now go to your browser and POST to:

```
http://localhost:7860/chat
```

Or use an app like **Postman** to test it.

---

### **Optional: Create an HTML Chat Interface**

Make a simple chat page (`chat.html`) like:

```html
<!DOCTYPE html>
<html>
<head>
    <title>DeepSeek Chat</title>
</head>
<body>
    <h2>Chat with DeepSeek</h2>
    <textarea id="input" rows="4" cols="50"></textarea><br>
    <button onclick="send()">Send</button>
    <pre id="response"></pre>

    <script>
        function send() {
            fetch("http://localhost:7860/chat", {
                method: "POST",
                headers: {"Content-Type": "application/json"},
                body: JSON.stringify({prompt: document.getElementById("input").value})
            })
            .then(res => res.json())
            .then(data => {
                document.getElementById("response").innerText = data.response;
            });
        }
    </script>
</body>
</html>
```

Open it in a browser and chat locally!

---

## **Final Tips**

- Use **DeepSeek** for best performance on most phones.
- Use **`-n 256`** to control number of output tokens (example: `-n 256 -p "Explain gravity"`).
- Monitor RAM usage — Deepseek 7B needs around 4–6 GB RAM for Q4.
- You can create wrapper scripts or integrate with Android web UIs using Termux:API if you want!

---

## **Conclusion**

Running local LLMs on Android is now a reality with `llama.cpp`, especially with small models like **TinyLLaMA**. For larger models like LLaMA 7B, high-end phones or tablets work well — and with Flask, you can even build your own mobile ChatGPT clone **completely offline**.

