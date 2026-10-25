# Fictional Bike Shop FAQ Bot

A chatbot answering questions about a fictional bike shop, built with Python and the Gemini API.

## Features
- Remembers the conversation (sends the last 5 messages with each request)
- Uses a system instruction to define the assistant's role, tone and rules
- Answers from a text file with shop data (bico-bazar.txt)
- Handles errors: retries on overload, switches to another model on limit errors

## How it works
The bot is built from a few small functions, each doing one job:

- `load_text(filename)` reads a text file and returns its content. If the file is missing, it prints a warning and returns an empty string instead of crashing.
- `ask_with_memory(history, question)` keeps the conversation as a list of messages (`user` / `model`). It adds the new question, sends only the last `MAX_MESSAGES` messages to the model, and stores the answer. If the request fails, the question is removed again so the history stays consistent.
- `call_with_fallback(contents)` tries the models from the `MODELS` list one by one. If a model reaches its quota limit, it switches to the next one.
- `call_gemini(contents, model)` sends one request to the Gemini API. On temporary overload (HTTP 503) it waits 2 seconds and retries up to 3 times; on quota errors (429) it reports a limit so that the fallback can switch models.
- `error_type(message)` turns an error message into a simple label (`overload`, `limit`, `unknown`).

The assistant's behaviour comes from a system instruction (role, language, tone, rules such as "never invent prices"). The shop data (`bico-bazar.txt`) is appended to it, so the model answers from the file and says so when it doesn't know something.

## How to run
1. Open the notebook in Google Colab.
2. Add your own Gemini API key as a Colab secret named GEMINI_API_KEY.
3. Upload bico-bazar.txt to the Colab files.
4. Run all cells and type your question. Type "vége" to quit.

## Notes
- All data in bico-bazar.txt is fictional.
- Free API tiers have daily limits; the bot may switch models.
- Built while learning, with help from AI assistants for explanations and code review.

## What I learned
- Writing and calling functions, using parameters and `return` values, and keeping each function focused on one task.
- Variable scope: what a function can see and what it returns to the outside.
- Error handling with `try/except`, retrying temporary errors, and handling different error types differently.
- Working with lists of dictionaries (the conversation history) and slicing to limit its length.
- Writing a system instruction and testing it with a fixed set of questions, including questions the data cannot answer.
- Providing a model with my own data from a text file, and noticing how it handles missing or ambiguous information.
- Refactoring: renaming everything to English names and checking that the behaviour didn't change.
