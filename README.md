# Fictional Bike Shop FAQ Bot

A chatbot for a fictional bike shop chain. It answers questions about opening hours, products, prices and policies
using tool calling: the model asks small Python functions that read a JSON data file.

## Features
- Remembers the conversation (sends the last N messages with each request)
- Tool calling: `get_opening_hours`, `get_price`, `get_policy`, `get_contact`, `list_stores`
- Shop data lives in one JSON file (`bico_bazar_data.json`), no copies in the prompt
- A system instruction defines role, tone and rules (never invent facts, quote prices exactly)
- Error handling: retries on overload, switches to another model on quota limits
- Replies in the language of the user's question (Hungarian by default)
- Optional Gradio web interface

## How it works
The bot is built from a few small functions, each doing one job:

- `load_data(filename)` reads the JSON file into `DATA`.
- Tool functions read `DATA` and return short, clear results. When nothing matches, they say what exists
  (e.g. the list of cities), so the model can correct itself.
- `ask_with_memory(history, question)` keeps the conversation as a list of messages (`user` / `model`). It adds the new question, sends only the last `MAX_MESSAGES` messages to the model, and stores the answer. If the request fails, the question is removed again so the history stays consistent.
- `call_with_fallback(contents)` tries the models from the `MODELS` list one by one. If a model reaches its quota limit, it switches to the next one.
- `call_gemini(contents, model)` sends one request with the system instruction and the tools. It retries on
  overload (503) and reports quota errors (429), so `call_with_fallback` can switch models.
- `error_type(message)` turns an error message into a simple label (`overload`, `limit`, `unknown`).

The assistant's behaviour comes from a system instruction (role, language, tone, rules such as "never invent prices").

## How to run
1. Open the notebook in Google Colab.
2. Add your own Gemini API key as a Colab secret named GEMINI_API_KEY.
3. Upload `bico_bazar_data.json` to the Colab files.
4. Run all cells and type your question. Type "vége" to quit.

## Development notes
During development the bot was checked with a list of test questions (not included in this repository).
Each question was run several times, because model answers vary.
Example finding: the rule "do not repeat contact details" held in 19 of 20 runs.

## Notes
- All data in bico_bazar_data.json is fictional.
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
- Tool calling, and that tool results (not only the prompt) decide what the model can answer.
- Keeping data in one place (JSON) and designing tool outputs that help the model recover from misses.
- Testing a non-deterministic system: repeated runs, pass rates, and checking text with care.

## Version history
- **v2.0** (current): JSON data and tool calling.
- **v1.0**: shop data in a text file inside the system instruction (see the v1.0 release), also in archive/v1-txt/.
