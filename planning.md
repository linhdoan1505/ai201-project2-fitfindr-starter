# FitFindr — planning.md

> Complete this document before writing any implementation code.
> Your spec and agent diagram are what you'll use to direct AI tools (Claude, Copilot, etc.) to generate your implementation — the more specific they are, the more useful the generated code will be.
> Your planning.md will be reviewed as part of your submission.
> Update it before starting any stretch features.

---

## Tools

List every tool your agent will use. For each tool, fill in all four fields.
You must have at least 3 tools. The three required tools are listed — add any additional tools below them.

### Tool 1: search_listings

**What it does:**

Searches the listings dataset for secondhand clothing items matching the user's description, size preference, and maximum price. Returns all matching listings sorted by relevance.

**Input parameters:**

- `description` (str): Keywords describing the item the user wants.
- `size` (str): Desired clothing size. Can be None.
- `max_price` (float): Maximum amount the user is willing to spend.

**What it returns:**

A list of matching listing dictionaries. Each listing contains fields such as:
- id
- title
- description
- category
- style_tags
- size
- condition
- price
- colors
- brand
- platform

**What happens if it fails or returns nothing:**

Returns an empty list `[]`. The agent stores an error message and stops instead of calling later tools.

---

### Tool 2: suggest_outfit

**What it does:**

Uses the selected listing and the user's wardrobe to suggest a complete outfit. The recommendation should explain how the new item fits with existing clothes.

**Input parameters:**

- `new_item` (dict): The selected clothing item returned by search_listings.
- `wardrobe` (dict): The user's wardrobe data.

**What it returns:**

A string containing one or more outfit recommendations and styling advice.

**What happens if it fails or returns nothing:**

If the wardrobe is empty, returns general styling advice using the new item. It should never crash or return an empty string.

---

### Tool 3: create_fit_card

**What it does:**

Creates a short, social-media-style caption describing the completed outfit. The output should sound shareable and personal.

**Input parameters:**

- `outfit` (str): Outfit recommendation produced by suggest_outfit.
- `new_item` (dict): Selected clothing item.

**What it returns:**

A short caption suitable for Instagram, TikTok, or a fashion-sharing app.

**What happens if it fails or returns nothing:**

Returns a descriptive error message such as "Unable to create fit card because outfit information is missing."

---

### Additional Tools (if any)

<!-- Copy the block above for any tools beyond the required three -->

---

## Planning Loop

1. Receive user query.

2. Call search_listings(description, size, max_price).

3. If no results are returned:
   - Store an error message in session["error"].
   - Stop execution.
   - Do not call any other tools.

4. If results are returned:
   - Select the first result.
   - Store it in session["selected_item"].

5. Call suggest_outfit(selected_item, wardrobe).

6. Store the returned outfit suggestion in session["outfit_suggestion"].

7. Call create_fit_card(outfit_suggestion, selected_item).

8. Store the result in session["fit_card"].

9. Return the completed session.

---

## State Management

The agent uses a session dictionary.

Tracked values:

- session["selected_item"]
- session["outfit_suggestion"]
- session["fit_card"]
- session["error"]

The selected item from search_listings is stored and passed directly into suggest_outfit. The outfit suggestion is then stored and passed into create_fit_card. This allows information to flow between tools without asking the user to repeat anything.

---

## Error Handling

| Tool | Failure mode | Agent response |
|------|-------------|----------------|
| search_listings | No results match the query | Store error message and stop execution |
| suggest_outfit | Wardrobe is empty | Return general styling advice instead of failing |
| create_fit_card | Outfit input is missing or incomplete | Return a descriptive error message |

---

## Architecture

User Query
     |
     v
Planning Loop
     |
     v
search_listings()
     |
     +---- no results ----> Error Message -> Return
     |
     v
selected_item
     |
     v
suggest_outfit()
     |
     v
outfit_suggestion
     |
     v
create_fit_card()
     |
     v
fit_card
     |
     v
Return Session
---

## AI Tool Plan

<!-- For each part of the implementation below, describe:
     - Which AI tool you plan to use (Claude, Copilot, ChatGPT, etc.)
     - What you'll give it as input (which sections of this planning.md, your agent diagram)
     - What you expect it to produce
     - How you'll verify the output matches your spec before moving on

     "I'll use AI to help me code" is not a plan.
     "I'll give Claude my Tool 1 spec (inputs, return value, failure mode) and ask it to implement
     search_listings() using load_listings() from the data loader — then test it against 3 queries
     before trusting it" is a plan. -->

**Milestone 3 — Individual tool implementations:**
For Milestone 3, I will use ChatGPT to implement each tool one at a time.

For search_listings:
- Input: Tool specification from the Tools section.
- Output: Python function that filters listings by description, size, and price.
- Verification: Test with at least 3 sample queries and verify returned items match the filters.

For suggest_outfit:
- Input: Tool specification and example wardrobe data.
- Output: Python function that generates outfit recommendations.
- Verification: Test with different wardrobe combinations and confirm recommendations are reasonable.

For create_fit_card:
- Input: Tool specification and sample outfit recommendations.
- Output: Python function that generates a short social-media-style caption.
- Verification: Test with multiple outfit descriptions and confirm captions are generated successfully.

**Milestone 4 — Planning loop and state management:**
For Milestone 4, I will use ChatGPT to implement the planning loop and session state.

Input:
- Planning Loop section
- State Management section
- Architecture diagram

Expected output:
- Agent logic that calls tools in the correct order
- Session dictionary for storing selected_item, outfit_suggestion, fit_card, and error

Verification:
- Run a complete user query from start to finish
- Confirm information passes correctly between tools
- Confirm failure paths stop execution when appropriate

---

## A Complete Interaction (Step by Step)

Write out what a full user interaction looks like from start to finish — tool call by tool call. Use a specific example query.

**Example user query:** "I'm looking for a vintage graphic tee under $30. I mostly wear baggy jeans and chunky sneakers. What's out there and how would I style it?"

**Step 1:**
The agent receives the query:

"I'm looking for a vintage graphic tee under $30. I mostly wear baggy jeans and chunky sneakers."

The agent calls:

search_listings(
    description="vintage graphic tee",
    size=None,
    max_price=30
)

The tool returns matching listings.

**Step 2:**
The agent selects the top matching listing and stores it in:

session["selected_item"]

The agent then calls:

suggest_outfit(
    new_item=session["selected_item"],
    wardrobe=user_wardrobe
)

The tool returns outfit recommendations.

**Step 3:**
The agent stores the recommendation in:

session["outfit_suggestion"]

Then calls:

create_fit_card(
    outfit=session["outfit_suggestion"],
    new_item=session["selected_item"]
)

The tool returns a fit card caption.

The caption is stored in:

session["fit_card"]

**Final output to user:**
Recommended Item:
Vintage Graphic Tee - $24

Suggested Outfit:
Pair the tee with your baggy jeans and chunky sneakers for a relaxed streetwear look.

Fit Card:
"Vintage vibes + oversized denim = today's perfect fit 🔥"
