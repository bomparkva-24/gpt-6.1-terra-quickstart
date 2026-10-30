# GPT-6.1 Terra Quickstart

Python and cURL examples for calling **GPT-6.1 Terra** through CometAPI using the OpenAI Python SDK.

This guide covers:

- Text generation
- Reasoning effort
- PNG image input
- Structured JSON output
- Chat Completions

[View GPT-6.1 Terra on CometAPI](https://www.cometapi.com/models/openai/gpt-6-1-terra/?utm_source=github&utm_medium=distribution&utm_campaign=bom&utm_content=gpt-6-1-terra)

> Confirm the model ID, endpoint availability, supported parameters, and current pricing in CometAPI’s documentation. Gateway compatibility does not guarantee support for every OpenAI API feature. These examples have not been tested against the live endpoint.

## Requirements

- Python 3.10 or later
- A CometAPI account and API key
- Access to `gpt-6.1-terra`

## Setup

Install the OpenAI Python SDK:

```bash
python -m pip install --upgrade openai
```

Set your API key:

```bash
export COMETAPI_KEY="your_key_here"
```

Keep your API key private. Do not commit it to your repository.

## cURL

Send a basic text request through the Responses API:

```bash
curl --fail-with-body \
  "https://api.cometapi.com/v1/responses" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $COMETAPI_KEY" \
  -d '{
    "model": "gpt-6.1-terra",
    "input": "Explain what an API is in one sentence."
  }'
```

## Python

Create an `examples` directory and save the following as
`examples/quickstart.py`:

```python
import base64
import json
import os
import sys
from pathlib import Path

from openai import OpenAI

api_key = os.environ.get("COMETAPI_KEY")
if not api_key:
    raise RuntimeError("Set COMETAPI_KEY before running this script.")

client = OpenAI(
    base_url="https://api.cometapi.com/v1",
    api_key=api_key,
    timeout=120.0,
)

MODEL = "gpt-6.1-terra"


def response_text(response):
    """Reject incomplete responses or responses without text."""
    if response.status != "completed":
        raise RuntimeError(
            f"Response did not complete: {response.status}"
        )

    if not response.output_text:
        raise RuntimeError("The response contained no output text.")

    return response.output_text


# 1. Text generation with reasoning effort.
# Confirm that "low" is supported by your endpoint.
response = client.responses.create(
    model=MODEL,
    reasoning={"effort": "low"},
    input="Explain what an API is in one sentence.",
)

print("\nText response:")
print(response_text(response))


# 2. Optional PNG image input.
# Pass a PNG file path as the first command-line argument.
if len(sys.argv) > 1:
    image_path = Path(sys.argv[1])
    image_bytes = image_path.read_bytes()

    if not image_bytes.startswith(b"\x89PNG\r\n\x1a\n"):
        raise ValueError("This example accepts PNG files only.")

    encoded_image = base64.b64encode(image_bytes).decode("ascii")

    response = client.responses.create(
        model=MODEL,
        input=[
            {
                "role": "user",
                "content": [
                    {
                        "type": "input_text",
                        "text": (
                            "Describe this screenshot in two sentences."
                        ),
                    },
                    {
                        "type": "input_image",
                        "image_url": (
                            f"data:image/png;base64,{encoded_image}"
                        ),
                    },
                ],
            }
        ],
    )

    print("\nImage response:")
    print(response_text(response))


# 3. Structured output.
schema = {
    "type": "object",
    "properties": {
        "title": {"type": "string"},
        "tags": {
            "type": "array",
            "items": {"type": "string"},
        },
    },
    "required": ["title", "tags"],
    "additionalProperties": False,
}

response = client.responses.create(
    model=MODEL,
    input=(
        "Suggest a blog title and three tags "
        "about AI invoice automation."
    ),
    text={
        "format": {
            "type": "json_schema",
            "name": "blog_post",
            "schema": schema,
            "strict": True,
        }
    },
)

post = json.loads(response_text(response))

print("\nStructured response:")
print(json.dumps(post, indent=2, ensure_ascii=False))


# 4. Chat Completions.
completion = client.chat.completions.create(
    model=MODEL,
    messages=[
        {
            "role": "user",
            "content": "Say hello in Filipino.",
        }
    ],
)

if not completion.choices:
    raise RuntimeError("Chat Completions returned no choices.")

message = completion.choices[0].message
if not message.content:
    raise RuntimeError("Chat Completions returned no text.")

print("\nChat Completions response:")
print(message.content)
```

Run the text and structured-output examples:

```bash
python examples/quickstart.py
```

Include a PNG screenshot:

```bash
python examples/quickstart.py "/path/to/screenshot.png"
```

The script makes three requests without an image, or four with one.
Each request may incur charges.

## Which endpoint should I use?

| Use case | Endpoint |
|---|---|
| Text generation | Responses or Chat Completions, if supported |
| Examples using `input_image` or `text.format` shown above | Responses |
| Provider-hosted tools | Check each tool’s endpoint requirements |

Function calling can be available through both Responses and Chat
Completions. Hosted tools such as web search and computer use have
separate support requirements.

Check CometAPI’s documentation before assuming that a tool available
through OpenAI is also available through the gateway.

## Pricing

Check the [current model listing](https://www.cometapi.com/models/openai/gpt-6-1-terra/?utm_source=github&utm_medium=distribution&utm_campaign=bom&utm_content=gpt-6-1-terra)
for input, output, caching, and tool charges.

When estimating costs:

- Check how reasoning tokens are billed.
- Confirm caching eligibility and any cache-write charges.
- Include tool usage, retries, and repeated context.
- Compare estimates with actual usage and billing.

## Troubleshooting

- **Authentication error:** Check that `COMETAPI_KEY` is set and valid.
- **Model not found:** Confirm the model ID and account access.
- **Unsupported parameter:** Check support for reasoning effort,
  image input, or structured output on the selected endpoint.
- **Incomplete response:** Inspect the response status and provider
  limits before retrying.
- **JSON parsing error:** Check for a refusal, incomplete output, or
  unsupported structured-output configuration.
- **Image request rejected:** Check supported formats, file-size
  limits, and request-size limits.

## Data handling

Send only content you are authorized to share with CometAPI and its
upstream providers. Review their data-handling policies before
uploading confidential documents or screenshots.

## Links

- [GPT-6.1 Terra model page and pricing](https://www.cometapi.com/models/openai/gpt-6-1-terra/?utm_source=github&utm_medium=distribution&utm_campaign=bom&utm_content=gpt-6-1-terra)
- [CometAPI documentation](https://apidoc.cometapi.com)

## License

To release this project under the MIT License, add an MIT `LICENSE`
file with your name and copyright year.

