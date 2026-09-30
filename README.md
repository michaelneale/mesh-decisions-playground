# Mesh Decisions Playground

A small local web UI for **text → question → decision** using Laya through
[Mesh LLM](https://github.com/Mesh-LLM/mesh-llm). Ask a Yes/No question or choose
between 2–16 labels, then inspect probability bars and the original API response.

![Decisions playground, showing a fixture response](docs/decisions.png)

*Screenshot uses deterministic test data, not a benchmark.*

## Run

Requires **Node.js 22+** and **mesh-llm v0.77.0+** installed separately.
Start Laya in one terminal:

```sh
mesh-llm serve --model meshllm/laya-multilingual-F16-GGUF
```

Mesh downloads the compatible GGUF and reuses its cache. Laya defaults to CPU.
Wait until the model is ready. In another terminal:

```sh
git clone https://github.com/michaelneale/mesh-decisions-playground.git
cd mesh-decisions-playground
npm start
```

Keep `npm start` running, then open **http://127.0.0.1:8787** in a browser
on the same computer. This is a local web app, not a hosted website.
With Node.js installed, no `npm install` or build step is needed to run the playground.

To use another Mesh endpoint or playground port:

```sh
MESH_URL=http://127.0.0.1:9447 PORT=8788 npm start
```

`MESH_URL` is an origin, **without `/v1`**, credentials, or query parameters.
The app only proxies `GET /v1/models` and `POST /systemone`. It never starts,
stops, downloads models for, or changes settings on your Mesh node.

You can co-host a chat model using Mesh's repeated `--model` option:

```sh
mesh-llm serve \
  --model meshllm/laya-multilingual-F16-GGUF \
  --model Qwen/Qwen2.5-0.5B-Instruct-GGUF:Q8_0
```

## Model discovery and limitations

Only explicit IDs advertising `system_one` appear in the picker. `auto` and
`mesh` are excluded; architecture/name alone never establishes support.

**Start by connecting directly to the node serving Laya.** Mesh v0.77.0 has
been observed listing remote Laya without its capability, and returning 404
for a remote decision. This app does not repair mesh routing or promise remote
support. If no models appear, check the chosen node's `/v1/models`, model
readiness, and `MESH_URL`, then click Refresh. On a 404, connect directly to the
serving node. Management-console `/systemone` passthrough is not assumed.

Yes/No maps to `type: "noul"`; choices map to `type: "choice"` with each label
also used as its description. Requests share text as `state` and use the named
question `decision`. This is not an OpenAI chat interface. See the
[System One contract](https://github.com/Mesh-LLM/mesh-llm/blob/main/website/src/docs/pages/system-one-api.md).

Probabilities are model estimates, not calibrated guarantees. There is no
automatic tool execution, chat gating, or safety-screening layer here.

## Privacy and boundaries

- Server binds **127.0.0.1 only**. Host and Origin checks reject foreign sites.
- Upstream is configured at startup, never supplied by a browser request.
- No analytics, external browser assets, prompt logging, or persistent browser storage.
- Text is sent to the configured Mesh endpoint. That endpoint's own routing,
  retention and logging policies still apply. Do not send sensitive text to an
  untrusted endpoint.
- JSON request limit: 128 KiB; upstream response limit: 1 MiB; timeout: 30 seconds.
- No upstream authentication support. Do not expose unauthenticated Mesh publicly
  to make this demo reachable. This is a local demo, not a public multi-user service.

## Develop and verify

```sh
npm ci
npm test
npx playwright install chromium
npm run test:browser
```

Server/contract tests use a real loopback mock upstream. Browser tests cover
Yes/No, choices, loading, invalid choices, errors, empty discovery, refresh,
HTML-safe results and a 390px mobile layout. They use fixture responses and
refresh the README screenshot. **This app has not yet been browser-verified
against live inference**; the underlying published-model setup was previously
verified with released Mesh v0.77.0. Tests start and stop only their own web
servers, never Mesh.
