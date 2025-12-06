import os
from flask import Flask, request, render_template_string
import openai

app = Flask(__name__)
openai.api_key = "YOUR_API_KEY_HERE"

HTML = """
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>AI Code Reviewer</title>
    <style>
        body { font-family: Arial, sans-serif; background: #f2f2f7; padding: 40px; }
        .box {
            background: white; width: 750px; margin: auto; padding: 30px;
            border-radius: 12px; box-shadow: 0 4px 20px rgba(0,0,0,0.1);
        }
        input[type="file"] { width: 100%; padding: 12px; margin-bottom: 15px; }
        button {
            padding: 14px 20px; background: #006cff; color: white; border: none;
            border-radius: 8px; cursor: pointer; font-size: 15px;
        }
        button:hover { background: #004fc2; }
        pre {
            white-space: pre-wrap; background: #eef3ff; padding: 20px;
            border-radius: 8px; font-size: 14px; border: 1px solid #d0d7ff;
        }
    </style>
</head>
<body>
    <div class="box">
        <h1>AI Code Reviewer</h1>
        <p>Upload any code file and get professional AI feedback.</p>

        <form method="POST" enctype="multipart/form-data">
            <input type="file" name="codefile" accept=".py,.js,.html,.css,.java,.c,.cpp,.json,.txt" required>
            <button type="submit">Review Code</button>
        </form>

        {% if result %}
            <h3>AI Review Result:</h3>
            <pre>{{ result }}</pre>
        {% endif %}
    </div>
</body>
</html>
"""

def review_code(code_text):
    prompt = f"""
You are a senior software engineer. Review the following code.
Provide:
- Quality analysis
- Bug detection
- Security issues
- Optimization suggestions
- Cleaner alternatives
- Final improved version of the code

Code:
\"\"\"
{code_text}
\"\"\"
"""

    response = openai.ChatCompletion.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt}]
    )

    return response["choices"][0]["message"]["content"]

@app.route("/", methods=["GET", "POST"])
def home():
    result = None

    if request.method == "POST":
        file = request.files["codefile"]
        code = file.read().decode("utf-8", errors="ignore")
        result = review_code(code)

    return render_template_string(HTML, result=result)

if __name__ == "__main__":
    app.run(debug=True)
