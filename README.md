# 🤖 AI Chatbot

LIVE:https://ai-chatbot-ashen-kappa.vercel.app/

An interactive, responsive chatbot web application where a user types a question and receives an AI-style reply in a chat interface. The front end is deployed on Vercel. This README also documents a proposed **full-stack architecture** (Node.js API + database) that the project is designed to grow into.

> **How to read this README**
> - Sections marked **(current)** describe what exists in the repository today: `index.html`, `styles.css`, `script.js`, `bgimage.jpg`, deployed on Vercel.
> - Sections marked **(proposed)** describe the backend that is *not yet built*. Say this clearly in interviews. "I designed the full-stack architecture and here is how I'd build it" is a strong answer; claiming a backend that doesn't exist is not, and one follow-up question will expose it.
> - I could not open `script.js`, so anything about how replies are generated is marked **[verify in script.js]**.

---

## 1. What is this project?

A single-page chat application. The user types a message, presses **Send**, and the bot's reply appears in the chat window. The page has a header with an info dialog, a scrollable chat window, and an input bar, and it works on desktop and mobile.

**Use of the project:** it demonstrates building an interactive, responsive UI, handling user events and async logic in JavaScript, managing conversation state, and deploying a web app. It is also the base for a full-stack AI assistant (customer support bot, FAQ bot, study helper).

## 2. Tech Stack

### Current
| Layer | Technology |
|---|---|
| Structure | HTML5 (semantic layout, native `<dialog>` element for the info popup) |
| Styling | CSS3 (responsive layout, background image) |
| Logic | Vanilla JavaScript (DOM manipulation, event handling) |
| Hosting / deployment | Vercel (static hosting, auto-deploy from GitHub) |
| Version control | Git / GitHub |

### Proposed full-stack version
| Layer | Technology |
|---|---|
| Front end | Current HTML/CSS/JS (or React/Vite) |
| Backend | Node.js + Express REST API |
| AI provider | LLM API (e.g. Anthropic Claude API) called **from the server only** |
| Database | MongoDB or PostgreSQL (users, conversations, messages) |
| Auth | JWT (JSON Web Tokens) + bcrypt password hashing |
| Validation / security | Zod or Joi, Helmet, CORS, express-rate-limit, environment variables |
| Testing | Jest + Supertest (API), Vitest/Jest (UI logic) |
| Deployment | Vercel (front end) + Render/Railway (API) + MongoDB Atlas / Neon |

## 3. Functional Requirements

### Current
- Show a chat interface with a header, message window and input bar.
- Let the user type a message and send it with the button (and ideally the Enter key).
- Render the user's message and the bot's reply as separate chat bubbles.
- Open and close a "ChatBot Info" dialog from the menu icon.
- Keep the layout usable on desktop and mobile screens.
- Maintain conversation continuity within a session **[verify in script.js how this is stored]**.

### Proposed (full-stack)
- User registration and login.
- Send a message to `POST /api/chat`; the server calls the AI model and returns the reply.
- Save every conversation and message in a database.
- List, open, rename and delete past conversations.
- Rate limiting and input validation on every endpoint.

## 4. Expected Behavior

1. User opens the app; an empty chat window and input box are shown.
2. User types a question and clicks **Send**.
3. The message appears on the right as a user bubble; the input is cleared.
4. A reply appears on the left as a bot bubble (optionally after a "typing..." indicator).
5. The chat window scrolls to the latest message.
6. Empty messages are ignored.
7. Clicking ☰ opens the info dialog; **Close** dismisses it.

**Proposed API flow:** browser → `POST /api/chat {conversationId, message}` → server validates and authenticates → server calls LLM with conversation history → server saves both messages → responds `{reply}` → browser renders it.

## 5. Backend

### Current
There is **no backend in this repository**. It is a static front-end site served by Vercel. **[verify in script.js]** whether replies come from a third-party AI API called directly from the browser, from a mock/rule-based function, or another source. If a key is used in `script.js`, it is visible to anyone using the site; that is the main reason a backend is needed.

### Proposed backend design
**Endpoints**

| Method | Route | Purpose |
|---|---|---|
| POST | `/api/auth/register` | Create account |
| POST | `/api/auth/login` | Return JWT |
| POST | `/api/chat` | Send message, get AI reply |
| GET | `/api/conversations?page=1&limit=20` | Paginated conversation list |
| GET | `/api/conversations/:id/messages?page=1&limit=50` | Paginated messages |
| PATCH | `/api/conversations/:id` | Rename |
| DELETE | `/api/conversations/:id` | Delete |
| GET | `/api/health` | Health check |

**Data model**

```
User         { _id, email, passwordHash, createdAt }
Conversation { _id, userId, title, createdAt, updatedAt }
Message      { _id, conversationId, role: "user"|"assistant", content, createdAt }
```

**Chat handler (pseudo-code)**
```js
app.post("/api/chat", auth, validate(chatSchema), async (req, res) => {
  const { conversationId, message } = req.body;
  const history = await Message.find({ conversationId }).sort("createdAt").limit(20);
  const reply = await llm.generate([...history, { role: "user", content: message }]);
  await Message.insertMany([
    { conversationId, role: "user", content: message },
    { conversationId, role: "assistant", content: reply },
  ]);
  res.json({ reply });
});
```

## 6. Content

- **Chat UI:** header with title and menu icon, chat window, text input, Send button, footer credit.
- **Info dialog:** explains that the chatbot assists with queries using AI-powered responses.
- **Assets:** a background image (`bgimage.jpg`).
- **Conversation content:** user messages and bot replies, kept as an ordered list of `{role, content}`.

## 7. Bonus (suggested enhancements)

- Enter-to-send, typing indicator, and "Clear chat" button.
- Streaming replies (token by token) using Server-Sent Events.
- Markdown and code-block rendering in bot replies.
- Dark / light theme toggle.
- Voice input (Web Speech API) and text-to-speech.
- Copy-message button and message timestamps.
- Persist chat in `localStorage` (current) or the database (full-stack).
- Docker setup and a CI pipeline (GitHub Actions).

## 8. Pagination

- **Current:** not applicable; one chat window in one session.
- **Proposed:** conversation and message lists are paginated using `page` and `limit` (or cursor-based pagination with `before=<messageId>` for infinite scroll upward). Response shape:
```json
{ "data": [...], "page": 1, "limit": 20, "total": 134, "hasMore": true }
```
Cursor pagination is preferred for chat because new messages arrive constantly and offset pages would shift.

## 9. Code Quality Expectations

- Separate concerns: HTML for structure, CSS for style, JS for behavior.
- Meaningful names; small functions such as `addMessage()`, `getBotReply()`, `scrollToBottom()`.
- No secrets in front-end code; use environment variables on the server.
- Escape or sanitize message text before inserting into the DOM (use `textContent`, not `innerHTML`) to prevent XSS.
- Handle errors: network failure, empty response, API timeout, with a friendly message in the chat.
- Consistent formatting (Prettier) and linting (ESLint).
- Accessibility: labels, focus handling, keyboard support, sufficient colour contrast, `aria-live` region for new messages.
- Meaningful commits and a clear README.
- Fix the README clone URL: it points to `prasanthirapuru/AI-chatbot`, not this repository.

## 10. Testing

**Current (manual checklist)**
- Send a normal message: reply appears.
- Send empty/space-only message: nothing happens.
- Send a very long message and special characters (`<script>`): rendered as text.
- Resize the window or use a phone: layout stays usable.
- Open/close the info dialog with mouse and keyboard.
- Disconnect the network: an error message is shown, the app does not freeze.

**Proposed automated tests**
- Unit tests for message formatting and state functions.
- API tests (Jest + Supertest): register/login, 401 without token, validation errors (400), chat happy path with the AI client mocked, pagination.
- End-to-end test (Playwright/Cypress): type message, click send, assert bubble appears.

## 11. Constraints

- Static front end cannot safely hold API keys or a database.
- Without a backend, conversation history lives only in the browser (lost on refresh unless saved locally).
- AI calls add latency; the UI must show loading state.
- Model APIs have rate limits and cost per token; the server should cap input length and history size.
- Free hosting tiers may sleep or limit usage; the live demo link currently returned a **402 error** when I tried to open it, which usually points to a Vercel account/plan or billing limit, so check the deployment before an interview.

## 12. Interview Discussion Points

- Why vanilla JS instead of React, and when would you switch?
- How do you keep conversation context? Why send history with each request?
- Where should the AI API key live and why?
- How would you add a backend, database, authentication and pagination?
- How do you prevent XSS, abuse and runaway cost?
- How would you handle slow or failed AI responses?
- How does Vercel deployment work (GitHub push → build → CDN)?

## 13. Expected Scope

**In scope (current):** responsive chat UI, message sending/rendering, info dialog, Vercel deployment.

**Planned (proposed):** Express API, AI integration on the server, MongoDB/PostgreSQL storage, JWT auth, pagination, tests.

**Out of scope for now:** payments, multi-user chat rooms, file uploads, model fine-tuning.

## 14. Final Goal

A production-ready AI chat application: a clean, accessible front end talking to a secure backend that authenticates users, calls an AI model safely, stores conversation history, and scales cleanly, with tests and deployment automation.

## 15. Project Structure

**Current**
```
Ai_chatbot/
├── index.html      # Page structure: header, dialog, chat window, input
├── styles.css      # Layout, chat bubbles, responsive rules
├── script.js       # Chat logic: send, render, reply
├── bgimage.jpg     # Background image
└── README.md
```

**Proposed full-stack structure**
```
ai-chatbot/
├── client/
│   ├── index.html
│   ├── styles.css
│   └── script.js            # calls /api/chat instead of an AI API directly
├── server/
│   ├── src/
│   │   ├── app.js           # Express setup, middleware
│   │   ├── server.js        # starts the server
│   │   ├── config/          # env, DB connection
│   │   ├── routes/          # auth.routes.js, chat.routes.js
│   │   ├── controllers/     # request handlers
│   │   ├── services/        # llm.service.js, chat.service.js
│   │   ├── models/          # User, Conversation, Message
│   │   ├── middleware/      # auth, validate, rateLimit, errorHandler
│   │   └── utils/
│   ├── tests/
│   ├── .env.example
│   └── package.json
├── .github/workflows/ci.yml
└── README.md
```

## 16. Setup

**Current front end**
```bash
git clone https://github.com/rapuruprasanth/Ai_chatbot.git
cd Ai_chatbot
# open index.html in a browser, or serve it:
npx serve .
```

**Proposed backend**
```bash
cd server
npm install
cp .env.example .env     # set MONGO_URI, JWT_SECRET, AI_API_KEY
npm run dev
```

---

# Deep Dive: Explain the Project (Interview Preparation)

## 17. Overall Explanation

### 17.1 What is it?
A web-based chatbot: a chat screen where a user asks questions and gets AI-generated replies. Built with plain HTML, CSS and JavaScript and deployed on Vercel.

### 17.2 What is it used for?
Answering user questions in a conversational way. The same pattern is used for customer support, FAQ bots, onboarding assistants, and study helpers. As a project, it shows UI building, event-driven JavaScript, async programming and deployment.

### 17.3 Why did I build it? (adapt to your real reason)
"I wanted to build something interactive that people can use right away, and AI chat is a good way to practice DOM handling, async requests, state management and responsive design. I deployed it on Vercel so it's live, and I've designed how to extend it into a full-stack app with a secure backend, database and authentication."

### 17.4 How does it work?
1. `index.html` provides the layout: header, `<dialog>` info popup, `#chat-window`, `#user-input` and `#send-btn`.
2. `styles.css` styles the bubbles and makes the layout responsive.
3. `script.js` listens for the Send click, reads and validates the input, appends the user bubble, obtains a reply **[verify: API call or local logic]**, appends the bot bubble, and scrolls down.
4. Vercel serves the files from a CDN and redeploys on every push to GitHub.

### 17.5 Full-stack architecture explained (proposed)
```
Browser (HTML/CSS/JS)
      │  HTTPS  POST /api/chat  (JWT in header)
      ▼
Express API ── validate ── auth ── rate limit
      │                         │
      │                         └── MongoDB/PostgreSQL (users, conversations, messages)
      ▼
AI provider API (key stored only on the server)
```
Why this is better: the key stays secret, history is stored per user, requests can be validated and rate-limited, and the front end stays simple.

### 17.6 Limitations (say these confidently)
- No backend or database yet; no user accounts.
- History is lost on refresh unless stored locally.
- If the AI key is used in the browser, it is exposed.
- No automated tests; no streaming; limited error handling.
- Live demo currently returns a 402 error, which needs fixing in Vercel.

### 17.7 How I would improve it
Add the Express backend, move the AI call server-side, add JWT auth, store conversations, paginate history, add streaming, tests, CI and better accessibility.

## 18. Short Pitches

**30 seconds:**
"It's a responsive AI chatbot built with HTML, CSS and JavaScript and deployed on Vercel. Users type a question and get a reply in a chat interface. I've also designed a full-stack version with an Express API, a database and JWT authentication, so the AI key stays on the server and conversations are saved."

**2 minutes:**
Cover: problem and use, the UI and how a message flows through `script.js`, deployment on Vercel, the limitations of a front-end-only app (secrets, no persistence), then the proposed backend: endpoints, data model, security and pagination.

## 19. Interview Questions and Answers

**Q1. Is this a full-stack project?**
Honest answer: "The implemented and deployed part is the front end. I've designed the full-stack version, with endpoints, data model and security choices, and I can walk you through how I'd build it." (If you build the backend before the interview, update this answer.)

**Q2. Explain your project in one line.**
A responsive chat web app that sends user questions to an AI and shows the replies in a conversational UI.

**Q3. Why HTML, CSS and vanilla JS?**
Small scope, no build step, fast to load, and it shows I understand the fundamentals before frameworks. For a bigger app I'd move to React.

**Q4. How do you send and render a message?**
Read the input value, trim and validate it, create a bubble element, set its `textContent`, append to the chat window, clear the input and scroll to the bottom.

**Q5. Why `textContent` instead of `innerHTML`?**
To prevent XSS: user or AI text must never be interpreted as HTML.

**Q6. What is the `<dialog>` element?**
A native HTML modal. `showModal()` opens it, and a `<form method="dialog">` button closes it, with built-in focus handling and Escape support.

**Q7. How does the bot get its response?** **[answer from script.js]**
If it calls an API: describe `fetch`, async/await, JSON parsing and error handling. If it is rule-based/mock: say so and explain how it would be replaced by a real model call.

**Q8. What is async/await and why is it needed here?**
Network calls take time; async/await lets the UI stay responsive while waiting, with cleaner code than nested callbacks.

**Q9. How do you maintain conversation context?**
Store messages as an array of `{role, content}` and send the recent history with each request, since language-model APIs are stateless.

**Q10. Why must the API key be on a server?**
Anything in front-end code is visible in the browser's dev tools. A backend proxy keeps the key secret and lets you add limits.

**Q11. How would you design the backend?**
Express with routes → controllers → services → models; JWT auth middleware; validation; rate limiting; a service that calls the AI; MongoDB or PostgreSQL for storage.

**Q12. SQL or NoSQL for this?**
Either works. MongoDB fits nested chat documents and flexible schema; PostgreSQL gives strong relations and transactions. I'd pick based on the team's stack.

**Q13. How does JWT authentication work?**
On login the server signs a token containing the user id; the client sends it in the `Authorization` header; middleware verifies the signature and expiry before allowing access. Passwords are stored as bcrypt hashes.

**Q14. How would you paginate messages?**
Cursor-based pagination (`before=<id>&limit=50`) so new messages don't shift pages; return `hasMore`.

**Q15. How would you handle slow or failed AI responses?**
Show a typing indicator, set a timeout, catch errors and show a retry message, and on the server return proper status codes (502/504) and log the failure.

**Q16. How do you prevent abuse and high cost?**
Rate limiting per user/IP, maximum input length, capped history size, authentication, and usage monitoring.

**Q17. What is CORS and why does it matter?**
Browsers block cross-origin requests unless the server allows them. With the front end on one domain and the API on another, the API must allow the front end's origin.

**Q18. How is it deployed?**
The repo is connected to Vercel; each push builds and deploys to a CDN. For the backend I'd use Render/Railway and set secrets as environment variables.

**Q19. How do you make it responsive?**
Flexible units and flexbox, media queries for small screens, the viewport meta tag, and a chat window that scrolls inside a fixed-height container.

**Q20. How would you test it?**
Manual checklist now; then Jest/Supertest for the API with the AI client mocked, and Playwright for an end-to-end send-and-reply flow.

**Q21. How would you add streaming responses?**
Server-Sent Events or fetch streaming: the server forwards tokens as they arrive and the client appends them to the current bubble.

**Q22. What was the hardest part?**
Replace with your real experience (for example: keeping the layout responsive, scrolling behavior, or handling async responses).

**Q23. What did you learn?**
DOM and event handling, async JavaScript, responsive CSS, deploying with Vercel, and why secrets and persistence require a backend.

**Q24. What would you do differently next time?**
Plan the backend from the start, add tests early, and use a component framework for state management.

**Q25. How would you scale this?**
Stateless API instances behind a load balancer, database indexes on `conversationId` and `createdAt`, caching, queueing for long requests, and monitoring.

### Interview tips
- Be upfront that the current build is a front-end app, then confidently explain the full-stack design.
- Open `script.js` before the interview and be able to explain every line.
- Fix the 402 on the live demo so you can show it working.
- Better still, build the small Express backend so the answer to Q1 becomes "yes".
