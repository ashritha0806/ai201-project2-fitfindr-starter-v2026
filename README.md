# FitFindr

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, every
> command, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> python app.py listings --full -n 6      # read the data (Milestone 1)
> python app.py fields                    # what you can filter on
> python app.py ask 'vintage graphic tee under $30'
> ```
>
> All three tools are stubs, so that last command will do nothing useful yet.
> That's the starting position.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     This is your submission. Fill each section in as you finish the milestone
     it belongs to — don't leave it all to the end.

     Unit 3 asks for the first five sections. Unit 4 adds the five below them.
     Leave the unit 4 sections alone until then; they're here so you know
     what's coming.

     Everything is pasted as TEXT. No screenshots, no images, no video links.
     A typed block of output gets full credit; a picture of the same output
     gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 3 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- Three or four sentences: what a user asks for, and what they get back. -->

FitFindr is an AI agent that acts as a personal stylist and shopper. A user provides a natural language request specifying the type of clothing they want, along with an optional size and max price. The agent searches listings to find a matching item, analyzes the user's existing wardrobe to suggest 1 or 2 personalized outfits incorporating the new find, and at last generates a social media caption about the outfit and the item's price. If the agent can not find a matching item, it will stop and ask the user to adjust their search criteria.
---

## Tool Inventory

<!-- Four lines per tool. This is worth 2 points and it's the single most
     common place students lose them.

     "Returns a list" earns NOTHING. The description has to say what is IN
     the list.

     The empty case isn't optional either — it's the thing your loop branches
     on, and if you don't decide it here you'll discover it as a crash in
     Milestone 5. -->

### `search_listings`

- **What it does:**Searches the listings data for items matching keyword, size, maximum price.
- **Inputs:** description (str),size (str or None) is optional, max_price (float or None).<!-- name and type each: `max_price` (float), not "a price" -->
- **Returns:**A list of matching listing dicts each containing id,title,description,category,style_tags,size,condition, price, colors, brand, platform, ordered by keyword relevance score up to SEARCH_RESULT_LIMIT.
- **When it has nothing:**Returns an empty list [].

### `suggest_outfit`

- **What it does:**Generates personalized styling ideas by combining a selected item with existing clothes from the user's wardrobe using an LLM.
- **Inputs:**new_item (dict), wardrobe (dict with an 'items' list of dicts).
- **Returns:**A non-empty string containing 1–2 outfit combination suggestions referencing items from the wardrobe.
- **When it has nothing:**Returns general styling ideas and outfit combinations for the item without referencing wardrobe pieces.

### `create_fit_card`

- **What it does:**Creates a short, engaging 2–4 sentence social media style caption about the find, mentioning its price, platform, and outfit vibe.
- **Inputs:**outfit (str), new_item (dict).
- **Returns:**A string containing a 2–4 sentence social media caption highlighting the item, price, platform, and styling vibe.
- **When it has nothing:**Returns a fallback caption mentioning the item, price, and platform without outfit details instead of error.

---

## Planning Loop

<!-- Your branch rule, stated as a rule — the condition AND both paths — plus
     the file and function that holds it.

     Like this:
       "If search_listings returns an empty list, put a message in the session
        and stop. Otherwise take the first result and go to suggest_outfit."
        — agent.py::run_agent

     The grader checks your code against what you claim here, so the file and
     function have to be real. -->

**Branch rule:**If search_listings returns an empty list, record an error message in session['error'] suggesting what the user could change (e.g. increase max price or widen search) and stop. Otherwise, take the first result as selected_item and proceed to suggest_outfit."

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:**Regex and string parsing (extracting max price with regex under \$?(\d+), size with regex size\s+([A-Za-z0-9/]+), and remaining words as description). <!-- regex, string splitting, or asking the model — say which -->

**What moves through the session:**A session is the history of what is happening through an agent loop. It's going to take the user query, then the parsed inputs, search results, the selected item, the wardrobe, outfit suggestion, and fit card and errors. And all these are going to carry through the session.

query and wardrobe (initial inputs) -> parsed (from parse_query) -> search_results (from search_listings) -> selected_item (first item from search results) -> outfit_suggestion (from suggest_outfit) -> fit_card (from create_fit_card), with error populated if search returns empty.
<!-- which fields, in what order -->

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   Here are two outfit ideas incorporating your new Y2K butterfly baby tee and pieces from your wardrobe:

**Outfit 1: Classic Y2K Streetwear**
* **Top:** Y2K Butterfly Baby Tee
* **Bottoms:** Baggy straight-leg jeans (dark wash)
* **Outerwear:** Black cropped zip hoodie
* **Footwear:** Chunky white sneakers
* **Accessories:** Black crossbody bag

*Why it works:* The fitted, cropped silhouette of the baby tee balances out the volume of the baggy straight-leg jeans for a classic early-2000s proportion play. Layering the black cropped zip hoodie on top keeps you warm while showing off the waistline, and the chunky sneakers and crossbody bag tie the casual, everyday street look together.

**Outfit 2: Edgy Contrast**
* **Top:** Y2K Butterfly Baby Tee
* **Bottoms:** Wide-leg khaki trousers
* **Outerwear:** Vintage black denim jacket
* **Footwear:** Black combat boots
* **Accessories:** Brown leather belt, Black crossbody bag

*Why it works:* This look leans into a cool high-low mix by pairing the feminine, playful butterfly print with tougher, structured pieces like the wide-leg khakis and combat boots. Tucking the baby tee in with the brown leather belt defines your waist against the relaxed trousers, and the vintage black denim jacket adds an effortlessly cool outer layer.

  Fit card: Living out my ultimate 2000s pop-star fantasy in this dreamy Y2K Baby Tee — Butterfly Print! 🦋✨ Grabbed this nostalgic little gem for just $18.0 to complete all my baggy-jean-and-combat-boot dreams. It’s live on my depop right now, so run don't walk before I change my mind and keep it!

0 model calls this session, 2 served from cache
```
$ python app.py ask '...'

```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=30))"
[{'id': 'lst_002', 'title': 'Y2K Baby Tee — Butterfly Print', 'description': 'Super cute early 2000s baby tee with butterfly graphic. Fitted crop length. Tag says medium but fits like a small.', 'category': 'tops', 'style_tags': ['y2k', 'vintage', 'graphic tee', 'cottagecore'], 'size': 'S/M', 'condition': 'excellent', 'price': 18.0, 'colors': ['white', 'pink', 'purple'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_006', 'title': 'Graphic Tee — 2003 Tour Bootleg Style', 'description': 'Vintage-style bootleg tee with faded graphic. Slightly boxy fit. 100% cotton, soft and worn-in.', 'category': 'tops', 'style_tags': ['graphic tee', 'vintage', 'grunge', 'streetwear', 'band tee'], 'size': 'L', 'condition': 'good', 'price': 24.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_017', 'title': 'Mesh Long-Sleeve Top — Black', 'description': 'Sheer black mesh long-sleeve. Great for layering under a graphic tee or over a bralette. Stretchy material, fits true to size.', 'category': 'tops', 'style_tags': ['y2k', 'grunge', 'goth', 'layering'], 'size': 'S/M', 'condition': 'excellent', 'price': 15.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_033', 'title': 'Vintage Band Tee — Faded Grey', 'description': 'Faded grey band-style tee with distressed graphic. Crew neck. Fits boxy. Well-loved but no holes or major damage.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'band tee', 'graphic tee', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 19.0, 'colors': ['grey', 'charcoal'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_011', 'title': 'Low-Rise Cargo Pants — Khaki', 'description': 'Y2K era low-rise cargo pants. Lots of pockets. Khaki color, slightly distressed at the hems. Great for layering with a long tee.', 'category': 'bottoms', 'style_tags': ['y2k', 'cargo', '2000s', 'streetwear'], 'size': 'W29', 'condition': 'fair', 'price': 27.0, 'colors': ['khaki', 'tan'], 'brand': None, 'platform': 'poshmark'}, {'id': 'lst_012', 'title': 'Oversized Crewneck Sweatshirt — Vintage Navy', 'description': 'Perfectly faded navy crewneck. Genuinely vintage — not manufactured distressed. Ribbed cuffs and hem. No graphics, clean.', 'category': 'tops', 'style_tags': ['vintage', 'basics', 'oversized', 'classic'], 'size': 'XL (fits oversized)', 'condition': 'good', 'price': 20.0, 'colors': ['navy'], 'brand': None, 'platform': 'thredUp'}, {'id': 'lst_015', 'title': 'Vintage Graphic Hoodie — Faded Black', 'description': 'Faded black pullover hoodie with barely-visible vintage graphic on the chest. Cozy interior. Some pilling but adds to the worn-in look.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'graphic', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 26.0, 'colors': ['black', 'charcoal'], 'brand': None, 'platform': 'depop'}]
```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"
Here are 2 outfit ideas incorporating your new vintage Levi's 501s with pieces from your wardrobe:

### Outfit 1: Off-Duty Casual
* **Top:** White ribbed tank top
* **Outerwear:** Oversized grey crewneck sweatshirt (worn draped over the shoulders or layered on top)
* **Shoes:** Chunky white sneakers
* **Accessories:** Black crossbody bag
* **Why it works:** This is the ultimate effortless, model-off-duty look. The casual, lived-in feel of the medium-wash 501s pairs naturally with the crisp white tank and chunky sneakers, while the oversized grey crewneck adds a cozy, textured layer for transitional weather.

### Outfit 2: Edgy Contrast
* **Top:** White ribbed tank top (tucked in)
* **Bottoms:** Vintage Levi's 501 Jeans
* **Waist:** Brown leather belt
* **Outerwear:** Black cropped zip hoodie
* **Shoes:** Black combat boots
* **Accessories:** Black crossbody bag
* **Why it works:** This outfit plays with proportions by pairing the high-waisted, straight-leg 501s with a cropped hoodie and chunky combat boots. Tucking in the white tank and adding the brown leather belt pulls the waist in and anchors the darker black layers with a classic vintage touch.
```

```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"
Nothing beats the effortless, 90s-grunge cool of fresh white sneakers paired with the ultimate off-duty uniform. I just scored these dreamy Vintage Levi's 501 Jeans — Medium Wash and the fit is absolute perfection. Grab them over on my depop right now for just $38.0 before I change my mind and keep them!
```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:*I asked the AI to help me implement the three tools in tools.py, specifically how to handle the size filtering.
- *What came back:*The AI suggested using the `re` (regex) module with word boundaries (`\b`) to ensure the size string is matched as a distinct word rather than a plain substring. It wrote the implementation for `search_listings` using this logic.
- *What I changed:* I used the AI's regex approach for the size filtering, and then asked it to help structure the planning loop in `agent.py` to correctly branch when `search_listings` returns an empty list, ensuring the state is passed via the `session` dictionary.


**Moment 2**

- *What I asked for:*I gave the AI my five finalized acceptance criteria and gave it a strict prompt: "For each one, tell me exactly how you would test it using only what the sentence says. Don't suggest improvements — just tell me what you'd do.
- *What came back:*The AI returned a step-by-step test plan for all five criteria. For example, for Criterion 3, it explained exactly how it would intercept the `item_id` from the search tool and string-compare it against the input to the suggest outfit tool. 
- *What I changed:*The AI was able to clearly explain how to test every single criterion based purely on my wording, it confirmed my sentences were objective and measurable. I kept my wording exactly as it was, knowing it passed the clarity test.

<!-- ═══════════════════════ UNIT 4 — THE TEST ═══════════════════════

     Don't fill these in during unit 3.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Run Log — Before

<!-- Five criteria, five tries each, in this exact format.

     Five, because your criteria are written out of five. Mark each try PASS
     or FAIL, count the passes, and read that count against your target — a
     row targeting 4 of 5 with three PASS cells is MISSED (3/5).

     `python run_eval.py --label before` runs everything and writes the table
     into results/. Paste it here and fill in the verdicts. -->

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. matching query completes | 4 out of 5 | PASS | PASS |PASS  | PASS | PASS | MET (5/5) |
| 2. impossible query stops early | 5 out of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. state item match   | 5 out of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. fit card includes price | 3 out of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. model unavailable error | 5 out of 5 | FAIL | FAIL | FAIL | FAIL | FAIL | MISS (0/5) |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

```
### matching query completes

- stopped early: no
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 10

Outfit suggestion:

```
Here is a 2000s-inspired outfit using your new Y2K baby tee and pieces from your wardrobe:

**Outfit: Casual Y2K Streetwear**
*   **Top:** Y2K Baby Tee — Butterfly Print
*   **Bottoms:** Baggy straight-leg jeans, dark wash
*   **Footwear:** Chunky white sneakers
*   **Outerwear layer (optional):** Black cropped zip hoodie
*   **Accessories:** Black crossbody bag

**Why it works:** 
The slim, cropped fit of the baby tee contrasts perfectly with the voluminous, low-slung silhouette of the baggy straight-leg jeans, capturing an authentic early-2000s streetwear look. Tossing on the black cropped zip hoodie keeps the proportions balanced, while the chunky white sneakers and crossbody bag tie the casual, everyday aesthetic together.
```

Fit card:

```
Channeling major 2000s street style with this fitted Y2K Baby Tee — Butterfly Print paired with baggy low-slung denim and chunky kicks! Grabbed this absolute gem for just $18.0, and it’s officially available now on depop to complete your ultimate retro rotation. Run, don't walk! 🦋✨
```

Trace:

```
[1] search_listings
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
[2] suggest_outfit
      in:  dict with keys: new_item, wardrobe
      out: Here is a 2000s-inspired outfit using your new Y2K baby tee and pieces from your wardrobe:  **Outfit: Casual Y…
[3] create_fit_card
      in:  dict with keys: outfit, new_item
      out: Channeling major 2000s street style with this fitted Y2K Baby Tee — Butterfly Print paired with baggy low-slun…
```

### impossible query stops early

- Query: `designer ballgown size XXS under $5`
- Wardrobe: example

**Try 1**

- stopped early: yes — No matching items found. Try removing the size filter, increasing your price limit, or using fewer keywords.
- selected_item: (none)
- search_results: 0

Trace:

```
[1] search_listings
      in:  dict with keys: description, size, max_price
      out: [] (empty)
```

### empty wardrobe

- Query: `denim jacket under $50`
- Wardrobe: empty

**Try 1**

- stopped early: no
- selected_item: Denim Jacket — Light Wash, Cropped ($42.0, poshmark)
- search_results: 7

Outfit suggestion:

```
Since your wardrobe is currently a blank slate, this light-wash, cropped denim jacket is actually the **ultimate foundational piece** to start with. A cropped jacket is particularly versatile because it naturally defines your waist and elongates your legs, making it easy to balance with different silhouettes. 

Because it’s a "blank canvas," it bridges the gap between casual and chic. Here is some general styling advice and a roadmap of the types of pieces you should look for next to build out your wardrobe around it.

---

### 1. General Styling Rules of Thumb
* **Play with Proportions (Tight/Loose):** Since the jacket is structured on top and cropped, it looks incredible when paired with high-waisted bottoms or looser, relaxed-fit pants. This creates a balanced "fitted top, relaxed bottom" silhouette.
* **The Denim-on-Denim Rule:** You *can* wear denim on denim, but with a light-wash jacket, make sure your jeans are either a **very distinct dark wash** (for high contrast) or an **exact matching light wash** (for a monochromatic Canadian-tuxedo look). Avoid mid-treads that almost match, as they can look mismatched.
* **Layering:** Because of the cropped cut, it looks amazing layered over things that peek out from the bottom (like a longer t-shirt or hoodie hem) to create intentional dimension.

---

### 2. Key Pieces to Add to Your Wardrobe

To make this jacket work hard for you, look for these foundational items next:

#### **Bottoms**
* **High-Waisted Wide-Leg Trousers (Black, Beige, or Olive):** Because the jacket is cropped, high-waisted pants will hit right at the jacket's hemline, giving you a very chic, put-together shape. Tailored trousers dress down nicely with the casual denim.
* **Straight-Leg Medium or Dark Wash Jeans:** A classic denim pairing. Straight-leg jeans keep the outfit looking modern rather than dated.
* **A Slip Skirt or Pleated Tennis Skirt:** The boxy, structured shoulders of the jacket contrast beautifully against the feminine, flowy movement of a skirt. 

#### **Tops (Layers Underneath)**
* **Basic Ribbed Tank Tops & Crop Tops (White, Black, Gray):** Essential for warm weather. Tucking a tight tank into high-waisted pants with the jacket thrown over top is an effortless, go-to outfit.
* **Oversized Graphic Tees:** Let the hem of a cool vintage tee peek out from under the cropped jacket for an edgy, streetwear-inspired vibe.
* **Striped Breton Turtleneck or Crewneck:** Light wash denim and classic French stripes (navy and white or black and white) are a timeless combination that instantly looks chic.

#### **Shoes**
* **Retro Sneakers (e.g., Adidas Sambas, Nike Air Force 1s, or New Balance):** Keeps the outfit grounded, casual, and street-style ready.
* **Chunky Black Boots (Doc Martens or Chelsea boots):** Adds a bit of toughness to balance out the light, breezy feel of the jacket.
* **Pointed-Toe Booties or Strappy Sandals:** Instantly elevates the jacket for a night out or a more dressed-up look.

---

### 3. Three Outfit Formulas to Try First

1. **The Casual Everyday:** White ribbed tank top + high-waisted straight-leg jeans + retro sneakers + the denim jacket.
2. **High/Low Mix:** Black tailored wide-leg trousers + a simple black t-shirt tucked in + chunky loafers + the denim jacket.
3. **Feminine Contrast:** A black slip dress or midi skirt + a fitted baby tee + ankle boots + the denim jacket draped over your shoulders.
```

Fit card:

```
Building a new wardrobe from scratch? The ultimate foundational piece has officially arrived! 

Channel that effortless, effortless-chic streetwear vibe by tossing this versatile **Denim Jacket — Light Wash, Cropped** over a basic ribbed tank and high-waisted trousers. Grab it now on Poshmark for just $42.0 before someone else steals your new favorite outfit formula!
```

Trace:

```
[1] search_listings
      in:  dict with keys: description, size, max_price
      out: 7 items: Denim Jacket — Light Wash, Cropped, Vintage Levi's 501 Jeans — Medium Wash, 90s Track Jacket — Navy/White Stripe … +4 more
[2] suggest_outfit
      in:  dict with keys: new_item, wardrobe
      out: Since your wardrobe is currently a blank slate, this light-wash, cropped denim jacket is actually the **ultima…
[3] create_fit_card
      in:  dict with keys: outfit, new_item
      out: Building a new wardrobe from scratch? The ultimate foundational piece has officially arrived!   Channel that e…
```
### state item match

- Query: `vintage graphic tee under $30`
- Wardrobe: example

**Try 1**

- stopped early: no
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 10

Outfit suggestion:

```
Here are two outfit ideas incorporating your new Y2K butterfly baby tee and pieces from your wardrobe:

### Outfit 1: Casual Y2K Streetwear
*Pair the baby tee with:*
* **Baggy straight-leg jeans** (Dark wash)
* **Black cropped zip hoodie** (worn open or layered casually)
* **Chunky white sneakers**
* **Black crossbody bag**

**Why it works:** This plays on the classic Y2K silhouette of a fitted, cropped top balanced with baggy, low-key denim. Throwing the black cropped zip hoodie over top keeps the midsection exposed or adds dimension while leaning into the 2000s aesthetic, finished off with chunky sneakers and a practical crossbody.

---

### Outfit 2: Edgy Contrast
*Pair the baby tee with:*
* **Wide-leg khaki trousers**
* **Brown leather belt**
* **Black combat boots**
* **Vintage black denim jacket**

**Why it works:** This outfit mixes preppier khaki trousers with the playful, girly butterfly graphic of the baby tee for an effortless high-low contrast. Tucking the baby tee in with the brown leather belt adds definition, while the black combat boots and vintage black denim jacket ground the look with an edgy, worn-in feel.
```

Fit card:

```
Channeling peak 2000s pop-star energy with this adorable Y2K Baby Tee — Butterfly Print, featuring the dreamiest nostalgic graphic. Grab this absolute steal for just $18.0 before I change my mind and keep it for myself! Head over to my depop shop right now to claim it before it’s gone. 🦋✨
```

Trace:

```
[1] search_listings
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
[2] suggest_outfit
      in:  dict with keys: new_item, wardrobe
      out: Here are two outfit ideas incorporating your new Y2K butterfly baby tee and pieces from your wardrobe:  ### Ou…
[3] create_fit_card
      in:  dict with keys: outfit, new_item
      out: Channeling peak 2000s pop-star energy with this adorable Y2K Baby Tee — Butterfly Print, featuring the dreamie…
```
### fit card includes price

- Query: `vintage graphic tee under $30`
- Wardrobe: example

**Try 1**

- stopped early: no
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 10

Outfit suggestion:

```
Since your wardrobe is currently a blank slate ("None" for all listed items), you have the ultimate freedom to build your first complete Y2K-inspired looks around this cute butterfly baby tee! 

Here are two distinct outfit formulas you can create by shopping for a few key companion pieces from your list:

### Outfit 1: The Ultimate Y2K Streetwear Look
*Pair the baby tee with baggy denim, a hoodie, and sneakers for that classic early 2000s off-duty model vibe.*

*   **Top:** Y2K Butterfly Baby Tee
*   **Bottoms:** Baggy straight-leg jeans (dark wash)
*   **Layer (Optional):** Black cropped zip hoodie (worn open or tied around the waist)
*   **Shoes:** Chunky white sneakers
*   **Accessories:** Black crossbody bag

**Why it works:** The fitted, cropped silhouette of the baby tee balances out the voluminous, slouchy fit of the baggy dark wash jeans. Adding the cropped zip hoodie and chunky white sneakers completely nails the effortless Y2K aesthetic.

---

### Outfit 2: Casual Retro-Prep
*Mix the ultra-feminine butterfly graphic with tailored trousers and a vintage jacket for a cool, high-low contrast.*

*   **Top:** Y2K Butterfly Baby Tee
*   **Bottoms:** Wide-leg khaki trousers
*   **Accessories:** Brown leather belt (threaded through the trousers)
*   **Outerwear:** Vintage black denim jacket
*   **Shoes:** Black combat boots

**Why it works:** Pairing a tight graphic tee with relaxed, wide-leg trousers creates a great proportion play. Tucking in the baby tee and accentuating the waist with a brown leather belt pulls the look together, while the vintage black denim jacket and combat boots add an edgy, timeless finish.
```

Fit card:

```
Channel your inner 2000s off-duty model with this sweet **Y2K Baby Tee — Butterfly Print**, featuring the ultimate effortless retro vibe. Grab this nostalgia-soaked staple now on **depop** for just **$18.0** before it flies away!
```

Trace:

```
[1] search_listings
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
[2] suggest_outfit
      in:  dict with keys: new_item, wardrobe
      out: Since your wardrobe is currently a blank slate ("None" for all listed items), you have the ultimate freedom to…
[3] create_fit_card
      in:  dict with keys: outfit, new_item
      out: Channel your inner 2000s off-duty model with this sweet **Y2K Baby Tee — Butterfly Print**, featuring the ulti…
```
### model unavailable error

- Query: `vintage graphic tee`
- Wardrobe: example

**Try 1**

- stopped early: no
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 10

Outfit suggestion:

```
Here are two outfit ideas featuring your new Y2K butterfly baby tee and pieces from your current wardrobe:

### Outfit 1: Effortless Off-Duty Y2K
*Pair the baby tee with your baggy straight-leg jeans and the black cropped zip hoodie for a balanced silhouette (fitted top meets loose bottoms).*

* **Top:** Y2K Butterfly Baby Tee
* **Outerwear:** Black cropped zip hoodie (wear open to show off the graphic)
* **Bottoms:** Baggy straight-leg jeans (dark wash)
* **Shoes:** Chunky white sneakers
* **Bag:** Black crossbody bag

### Outfit 2: Edgy Contrast
*Combine the feminine, retro butterfly graphic with tougher, utilitarian pieces like black combat boots and denim.*

* **Top:** Y2K Butterfly Baby Tee
* **Outerwear:** Vintage black denim jacket
* **Bottoms:** Baggy straight-leg jeans (dark wash)
* **Shoes:** Black combat boots
* **Accessories:** Brown leather belt (to add a nice contrast against the all-black and dark wash denim)
```

Fit card:

```
Channeling ultimate off-duty bratz energy with this angelic Y2K Baby Tee — Butterfly Print! 🦋✨ Snagged this nostalgic retro gem for just $18.0, and it’s officially live on depop waiting for its next icon. Run, don't walk!
```

Trace:

```
[1] search_listings
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
[2] suggest_outfit
      in:  dict with keys: new_item, wardrobe
      out: Here are two outfit ideas featuring your new Y2K butterfly baby tee and pieces from your current wardrobe:  ##…
[3] create_fit_card
      in:  dict with keys: outfit, new_item
      out: Channeling ultimate off-duty bratz energy with this angelic Y2K Baby Tee — Butterfly Print! 🦋✨ Snagged this no…
```


```

---

## Verdicts and Diagnoses

<!-- MET or MISSED per criterion against LAST UNIT's target, plus a sentence on
     how you decided.

     Then, for every miss: which of the four places it happened — a tool, the
     loop's branch, the session, or the model's output — AND the mechanism.

     Not a diagnosis:  "The fit card was bad."
     A diagnosis:      "The fit card criterion missed on 2 of 5 items. Both had
                        an empty brand field. My prompt puts the brand in the
                        first sentence, so the card opened with a blank and read
                        like a fragment. The tool worked; the prompt assumed a
                        field that isn't always there."

     Look for a pattern. Three misses on the same tool is one problem, not
     three. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**



---

## Loop Trace

<!-- One full run, printed step by step, with the MCP call visible in it.

     `python app.py ask '...' --trace` once you've added the trace.step()
     calls in Milestone 2.

     Worth pasting BOTH the happy path and the empty-search path. The empty
     one should be visibly shorter, because it stops. If your two traces are
     the same length, your branch isn't working — and this is the fastest way
     anyone will ever find that out. -->

**Happy path**

```
[1] search_listings
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
[2] suggest_outfit
      in:  dict with keys: new_item, wardrobe
      out: Here are two outfit ideas incorporating your new Y2K butterfly baby tee and pieces from your wardrobe:  **Outf…
[3] create_fit_card
      in:  dict with keys: outfit, new_item
      out: Living out my ultimate 2000s pop-star fantasy in this dreamy Y2K Baby Tee — Butterfly Print! 🦋✨ Grabbed this n…

  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   Here are two outfit ideas incorporating your new Y2K butterfly baby tee and pieces from your wardrobe:

**Outfit 1: Classic Y2K Streetwear**
* **Top:** Y2K Butterfly Baby Tee
* **Bottoms:** Baggy straight-leg jeans (dark wash)
* **Outerwear:** Black cropped zip hoodie
* **Footwear:** Chunky white sneakers
* **Accessories:** Black crossbody bag

*Why it works:* The fitted, cropped silhouette of the baby tee balances out the volume of the baggy straight-leg jeans for a classic early-2000s proportion play. Layering the black cropped zip hoodie on top keeps you warm while showing off the waistline, and the chunky sneakers and crossbody bag tie the casual, everyday street look together.

**Outfit 2: Edgy Contrast**
* **Top:** Y2K Butterfly Baby Tee
* **Bottoms:** Wide-leg khaki trousers
* **Outerwear:** Vintage black denim jacket
* **Footwear:** Black combat boots
* **Accessories:** Brown leather belt, Black crossbody bag

*Why it works:* This look leans into a cool high-low mix by pairing the feminine, playful butterfly print with tougher, structured pieces like the wide-leg khakis and combat boots. Tucking the baby tee in with the brown leather belt defines your waist against the relaxed trousers, and the vintage black denim jacket adds an effortlessly cool outer layer.

  Fit card: Living out my ultimate 2000s pop-star fantasy in this dreamy Y2K Baby Tee — Butterfly Print! 🦋✨ Grabbed this nostalgic little gem for just $18.0 to complete all my baggy-jean-and-combat-boot dreams. It’s live on my depop right now, so run don't walk before I change my mind and keep it!
0 model calls this session, 2 served from cache

```

**Empty search**

```
[1] search_listings
      in:  dict with keys: description, size, max_price
      out: [] (empty)

  No matching items found. Try removing the size filter, increasing your price limit, or using fewer keywords.

```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->



---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

**Which failure it was meant to fix:**

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 3

       [ ] criteria.md has five numbered criteria, each with a target
       [ ] Each criterion has a reason underneath it
       [ ] All five unit 3 sections above have real content
       [ ] Tool Inventory: all three tools, inputs WITH TYPES, a specific
           return value, and the empty case
       [ ] Planning Loop names the branch rule and agent.py::run_agent
       [ ] Sample Run: one full query plus the three per-tool tests, as text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN, you submit the same one
           next unit

     SUBMISSION CHECKLIST — unit 4

       [ ] mcp_server.py exists with one tool registered
           (or a written record of exactly where the rewire broke)
       [ ] Run Log — Before, five criteria, five tries each
       [ ] Real output pasted underneath, naming file and function
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, naming a place AND a mechanism
       [ ] Loop Trace, with the MCP call visible in it
       [ ] All three failure modes triggered and handled
       [ ] One improvement, with Run Log — After in the same format
       [ ] What's Still Broken
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository. Your commit history is what
     shows your criteria existed before your results did.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
