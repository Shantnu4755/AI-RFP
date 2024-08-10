# Code Structure of the Project

## Introduction
This document outlines the structure and functionality of the modules and functions used in this project.

## Utilizing GPT for Presentation Generation

The primary functionality of this application is contained in `flaskapp.py`, where various functions handle different aspects of the application. A crucial part of this script is the endpoint defined for generating presentations based on user input:

```python
@app.route('/generator', methods=['GET', 'POST'])
def generate():
    if request.method == 'POST':
        number_of_slide = request.form['number_of_slide']
        user_text = request.form['user_text']
        template_choice = request.form.get('template_choice')
        presentation_title = request.form['presentation_title']
        presenter_name = request.form['presenter_name']
        insert_image = 'insert_image' in request.form
```

In this snippet, the `/generator` endpoint accepts both `GET` and `POST` requests. For `POST` requests, the application gathers necessary information from the user via a form submission. This information includes the number of slides, text content, template choice, presentation title, presenter name, and an option to insert images.

The following snippet shows how we prepare the user’s input for processing by the GPT model:

```python
user_message = f"I want you to come up with the idea for the PowerPoint. The number of slides is {number_of_slide}. " \
                f"The content is: {user_text}. The title of content for each slide must be unique, " \
                f"and extract the most important keyword within two words for each slide. Summarize the content for each slide."
```

Instead of passing the raw `user_text` directly to GPT, we construct a formatted message, `user_message`, which clearly communicates the user’s request to GPT. This method ensures that the generated presentation aligns with the specified requirements. It can handle various ways of phrasing the user’s request.

For example, whether a user submits a request as `Evolution of AI` or phrases it as `Can you make a presentation for Evolution of AI with clear examples?`, the application processes the request effectively.

Keyword extraction is subsequently used for retrieving relevant images via the Pexels API.

In `flaskapp.py`, three functions are executed:
- `chat_development()` from `gpt_generate.py` located in `myapp/utils` to get GPT's response.
- `parse_response()` from `text_pp.py` located in `myapp/utils` to process the assistant's response and obtain slide content.
- `create_ppt()` from `text_pp.py` located in `myapp/utils` to generate the PowerPoint presentation with the slide content, template choice, presenter’s name, and image insertion option.

```python
assistant_response = chat_development(user_message)
# Check the response (for debugging)
print(assistant_response)
slides_content = parse_response(assistant_response)
create_ppt(slides_content, template_choice, presentation_title, presenter_name, insert_image)
```

The `print(assistant_response)` statement is used to verify if GPT is responding correctly.

### `gpt_generate.py`

**build_conversation**

```python
def build_conversation(user_message):
    return [
        {"role": "system",
         "content": "You are an assistant that provides ideas for PowerPoint presentations. Provide the user with summarized content for each slide based on the number of slides. "
                    "The format should be: Slide X: {title of the content} /n Content: /n content with bullet points. "
                    "Keyword: /n Provide the most important keyword (within two words) representing the slide."},
        {"role": "user", "content": user_message}
    ]
```

This function accepts `user_message` and creates a structured conversation array. The "system" role provides instructions to the GPT model, setting the context for generating the response. This ensures the model delivers the output in the desired format.

**generate_assistant_message**

```python
def generate_assistant_message(conversation):
    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=conversation
    )
    return response['choices'][0]['message']['content']
```

This function sends a request to OpenAI’s ChatCompletion endpoint using `openai.ChatCompletion.create`, specifying the model and conversation array. It extracts and returns the content of the generated message.

**chat_development**

```python
conversation = build_conversation(user_message)
    try:
        assistant_message = generate_assistant_message(conversation)
    except openai.error.RateLimitError as e:
        assistant_message = "Rate limit exceeded. Sleeping for a bit..."

    return assistant_message
```

The `try` block attempts to obtain a response from GPT by calling `generate_assistant_message`. If a `RateLimitError` occurs (indicating the API rate limit has been exceeded), the `except` block assigns a message indicating the issue. 

If you encounter the "Rate limit exceeded" message, check your OpenAI API usage on the [OPENAI API Usage](https://platform.openai.com/account/usage) page to review your API rate limits and usage.

---