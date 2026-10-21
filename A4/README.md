# The Ordinary, Installed

A Flask web app that turns an everyday object into an imagined contemporary artwork. It generates a museum tombstone, an 80–120 word extended label in an exaggerated International Art English (IAE) register, and an exhibition image based on that label.

## Project structure

```text
.
├── app.py                 Flask routes, prompts, OpenAI calls, and validation
├── templates/index.html   Object form and gallery-style label/image display
├── static/style.css       Responsive gallery-inspired styling
├── requirements.txt       Python dependencies
├── Procfile               Gunicorn web process
├── render.yaml            Render service configuration
├── .env.example           Local environment-variable names (no real secrets)
└── .gitignore             Keeps keys and local files out of Git
```

## Run locally

You need Python 3.10 or later and an OpenAI API key with access to the selected text and image models. From the project directory:

1. Create and activate a virtual environment: `python -m venv .venv` then activate it (`.venv\Scripts\Activate.ps1` in PowerShell, or `source .venv/bin/activate` on macOS/Linux).
2. Install dependencies: `pip install -r requirements.txt`.
3. Copy `.env.example` to `.env`, then set `OPENAI_API_KEY` and a long random `FLASK_SECRET_KEY`. The app does not load `.env` automatically; export the variables in your shell or use a local environment manager. In PowerShell, for the current session: `$env:OPENAI_API_KEY="..."; $env:FLASK_SECRET_KEY="..."`.
4. Start the app with `python app.py` and visit <http://localhost:5000>.

Each submission calls the OpenAI text and image APIs and may incur usage charges. To run only the web server without generating anything, start it and view the page; generation requires a configured key. The key is read from the server environment, never embedded in HTML or returned to the browser. Keep `.env` out of version control.

### Manual local verification

With the server running and a valid API key set, open the home page and submit an ordinary object such as “a scratched stainless-steel travel mug with a black lid.” Check that the result shows the artist and bio, title and year, medium, dimensions, provenance, credit line, and an extended label whose displayed count is between 80 and 120 words. Confirm an exhibition image appears beside it. Also try a two-character description and a description longer than 500 characters; both should be rejected before any API call. To check missing-key handling, stop the server, unset `OPENAI_API_KEY`, restart it, and submit a valid description; the page should explain that the key is not configured. These checks use the live APIs when generating a result and may incur charges.

## Use the app

Enter a short, concrete description of an ordinary object from a pocket, bag, or desk (3–500 characters), then choose **Install the work**. The result places an AI-generated exhibition view beside the tombstone and extended wall label. If image generation fails, the label remains available and the page explains that the image could not be made. If the key is missing, the app gives a configuration message. API failures are logged on the server and presented as a brief retry message.

## Prompt design and IAE features

The label system prompt makes six explicit rules. It asks the model to preserve the object's ordinary identity; make humor from the gap between use and institutional importance; invent all artist, theorist, book, and provenance identities; return a fixed set of JSON fields; keep the extended label to 80–120 words while citing an invented theorist and book; and keep dimensions plausible and legible.

The selected IAE features are deliberate abstraction, nominalization, institutional language, and the tension between material specificity and theoretical interpretation. The prompt calls out devices such as “the conditions of its visibility” and asks for exaggeration while maintaining a concrete description of the submitted thing. The app uses a strict JSON schema so every required tombstone field is returned as text, then validates the extended-label word count. If the prose misses the 80–120 word range, it automatically asks once for a corrected version before showing an error.

## Label-to-image chain

The image-generation function receives the completed label only. It builds its prompt from the label's title, artist, medium, dimensions, provenance, and extended label, then asks for a restrained white-cube installation photograph with a low plinth and track lighting. The raw user description is not sent to the image model. The prompt directs the image model to preserve the described object's identity, material, and scale. The app displays a base64 PNG response as a data URL; it also supports an image URL if the API returns one.

## Why is a language model fluent in IAE?

Language models learn statistical patterns from very large collections of text. Exhibition writing, art criticism, press releases, and theory circulate online in recognizable forms; their recurring vocabulary and sentence structures provide patterns a model can reproduce. IAE is especially legible as a style because abstraction, nominalization, institutional framing, and references to theory recur across many examples. The model does not need to hold the beliefs behind those texts to imitate their surface. This makes it useful for exposing the dialect's conventions—and for making its rhetorical inflation visible when applied to a hair clip or a transit card.

## Deploy on Render

1. Place all project files in the `A4` folder of `casafresca/CART498-GenAI`, replacing the empty `A4/gg` placeholder. Do not commit `.env`, API keys, or other secrets.
2. In Render, either choose **New → Blueprint**, connect `casafresca/CART498-GenAI`, and set the Blueprint Path to `A4/render.yaml`; or create a **Web Service** and set **Root Directory** to `A4`, build command to `pip install -r requirements.txt`, and start command to `gunicorn app:app --bind 0.0.0.0:$PORT`.
3. In the service's **Environment** settings, set `OPENAI_API_KEY` to your secret key. Set `FLASK_SECRET_KEY` to a long random value if Render did not generate one from the Blueprint. Keep both values in Render's environment settings, not in Git.
4. Deploy. Render installs the requirements and runs Gunicorn. Open the service URL, submit a sample object, and confirm both the label and image appear. Check Render logs if generation fails; model availability, API access, and account billing can affect requests.

The included Blueprint sets `rootDir: A4` for the course monorepo, uses Python 3.12.8, generates a Flask secret key, and prompts for the OpenAI key. The included `Procfile` is provided for platforms that use it; Render uses the command in `render.yaml` or the service settings.

## Configuration

| Variable | Required | Purpose |
|---|---:|---|
| `OPENAI_API_KEY` | Yes, for generation | Server-side OpenAI credential |
| `FLASK_SECRET_KEY` | Yes in production | Signs Flask session cookies; set a long random value |
| `OPENAI_TEXT_MODEL` | No | Text model override; defaults to `gpt-4o-mini` |
| `OPENAI_IMAGE_MODEL` | No | Image model override; defaults to `gpt-image-1` |
| `PORT` | No | Local server port; defaults to `5000` (Render supplies its own runtime port to Gunicorn) |

## Notes

Generated labels and images are imaginative outputs, not claims about real provenance or real artists. Results are rendered in the response and are not stored in a database. For a class deployment, review API usage and billing limits before sharing the public URL.
