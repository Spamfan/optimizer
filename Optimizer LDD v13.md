## **LIVING DESIGN DOCUMENT (LDD) & PROJECT BIBLE** 

## **Project: Optimizer & ecosystem (incl IM, Prints, IRIS/scanner, etc)** 

Last Updated: optimizer 1.3.3, Installment Manager beta version 21, prints v0.5, and Scanner (internally called IRIS) version 0.0.9 

## **PART 0: PROJECT ROLES & WORKFLOW** 

## **A. THE TEAM** 

1. Human Developer (Me/Project Lead/team lead ): 

- I am the final authority on all decisions. 

- I am responsible for field testing on the WMPC, finding bugs, and defining 

- feature requirements. 

- I maintain this LDD. 

2. AI Developer (You/AIA): 

- You are the programmer and logic analyst. 

- You translate my requests into code, brainstorm solutions, and maintain the codebase. 

- You are responsible for strictly following the protocols defined below. 

## **B. THE "GOLDEN RULE" (STRICT WATERFALL)** 

1. The Workflow Cycle: 

- You must strictly adhere to this development loop for every task: 

- a. Sprint Plan (SP): High-level logic and goals. 

- b. Approval: Wait for my explicit "Yes". 

- c. Tech Spec (TS): Detailed technical blueprint of the changes. 

- d. Approval: Wait for my explicit "Yes". 

- e. Rewrite/Patch: Writing the actual code (usually code patches) 

## **2. The Trigger Protocol (CRITICAL):** 

- You must STOP after every single phase. 

- You are NEVER allowed to proceed to the next step without a specific, case-by-case command from me (e.g., "Please now write the TS"). 

- Hints, general agreement, or phrases like "looks good" do NOT count as authorization. If I do not explicitly tell you to write code, do NOT write code. 

## **3. Versioning & Delivery:** 

- Every change must have a unique Version number, usually up to 3 decimals (0.0.1, 0.2.4, 1.3, etc). The only exception to this rule currently is the project landing page, called <u>spamfan.github.io, which for now does NOT use version numbers. The coding flow is</u> otherwise the same. 

- Your standard delivery method is via find and replace patches using markdown. Full File Rewrites creations may ONLY be used when starting a new separate app / project or when otherwise explicitly directed by the team lead. Patches must always be used unless otherwise explicitly noted on a case by case basis. 

-heres an example of how patches should be perfectly formatted by the AIA when the time comes. Code sections must be entirely unabbreviated so users can find exact text string matches. Patches should always be numbered, and never have sub steps. One patch is one pair of one find and one replace block. If another patch is needed it has a separate number. 

Here is an example of one patch  below, perfectly formatted as they should be, in markdown: 

Patch 1/5 

Find 
```html 
Old code here </div>
 ``` 

Replace with
 ```html
 New code here
 ``` 

Etc. 

As for "1/5" that's just me asking u to plz show how many full patches there are whenever patches r sent; it makes it easier to see which patch I'm on and how many more I need to execute. 

## **PART I: PROJECT ARCHITECTURE & INFRASTRUCTURE** 

## **A. THE ECOSYSTEM (GitHub Repositories at spamfan.github.io/<repo name here>)** 

1. optimizer: The Stable version of the Main App. 

2. optimizerunstable: (rarely used, for unstable public tests) 

3. installmentmanager: The Stable Admin Tool. 

4. installmentmanagerunstable: (rarely used, for unstable public tests) 

5. Prints 

6. PrintsUnstable 

7. <u>Spamfan.github.io</u> (front end landing page) 

8. Scanner (very basic redirect to IRIS) 

9. IRIS (internal name for the initial/current scanner prototype) 

10. Some other  repos may have not been listed here. 

## **B. THE "HYBRID CLOUD BRIDGE" MODEL (DATA FLOW esp regarding optimizer, prints, and scanner)** 

1. The Problem: WMPC blocks direct Firebase reads, GitHub API calls, and GitHub Raw reads (`raw.githubusercontent.com`). 

2. The "Smuggler's Route" (EDLPs & Optimizer Data): 
- Write Path (Admin via Installment Manager): User edits prices in IM -> Saves to Firebase (Realtime DB) for fast UI syncing -> IM uses a Personal Access Token (PAT) stored in local storage to send a direct API `PUT` request to the GitHub repository, overwriting `edlp_data.json` on the `main` branch. 
- The Cache Purge (GitHub Actions): Whenever `edlp_data.json` is updated, a GitHub Workflow (`purge_cdn.yml`) automatically triggers. It runs a staggered 10-minute loop, pinging the jsDelivr Purge API to instantly clear the global CDN cache and bypass the standard 15-minute CDN latency.
- Read Path (WMPC/Kiosk): Optimizer loads -> Fetches `edlp_data.json` from the jsDelivr CDN (using a `?t=Date.now()` cache-buster) -> Updates UI. 

3. The "Prints" Exception (GitHub Pages): 
- While `raw.githubusercontent` is blocked, standard web hosting via `github.io` (GitHub Pages) is allowed by the WMPC firewall. The "Prints" app successfully loads its assets/PDFs because it routes through the allowed GitHub Pages domain rather than the raw code domain.

4. Current Cloud JSON Structure (`edlp_data.json`):
- Currently strictly limited to a single root object: `{"devices": { "device_key": { "name": "...", "att": {...}, "vzw": {...}, "tmo": {...} } } }`. 

## **C. SECURITY STRATEGY & AUTHENTICATION** 

1. The "2027 Timebomb": 

- Firebase is currently in "Test Mode" with public read/write access expiring in 2027. 
- Constraint: Do NOT enable strict "auth != null" rules until a Login UI is fully implemented. 

2. GitHub Credentials (PAT) & The "Bot" Clarification: 
- GitHub Personal Access Tokens (PATs) are stored in `localStorage` (`gh_config`) on the Admin device only. They never touch the WMPC. 
- Note: The ecosystem does NOT require a dedicated "optimizer-bot" server or account. The direct GitHub API `PUT` request simply uses whichever PAT is saved in the Admin's browser. If the PL uses their main account's PAT, the commit history will cleanly reflect the PL's main account. The "bot" was simply a legacy concept for token isolation.

## **PART II: ENVIRONMENT CONSTRAINTS (THE "WMPC")** 

## **A. THE ENVIRONMENT** 

1. Target Machine: Walmart PC AKA WMPC (Kiosk Mode), usually for user end of optimizer and prints 

1a. Network Status: Hostile / Restricted / Whitelist-based. 

2.  Other target machine (currently used to SEE inventory, which users scan in via OCR on their personal mobile device): the ESP tablet.  (What the user uses to just see inventory reports) 

2a. Likely same network status as WMPC. 

3. Another target machine lol: user’s personal phone (iOS Android etc), usually no network restrictions, used by admin for accessing installment manager and by admin (and eventually others) for running scanner. 

## **B. BLOCKED TECHNOLOGIES (DO NOT USE)** 

1. GitHub Raw: raw.githubusercontent.com is blocked. 

2. Firebase WebSockets: Blocked on WMPC. 

3. Modern Clipboard API: navigator.clipboard is often restricted. Must maintain document.execCommand('copy') fallback. 

## **C. ALLOWED TECHNOLOGIES** 

1. JSDelivr CDN: cdn.jsdelivr.net (The primary data transport). 

2. Google APIs: ajax.googleapis.com (Potential future route). 

## **D. MAINTENANCE RULES (STABILITY)** 

1. Library Pinning: Do NOT upgrade libraries without WMPC testing. 

- Firebase: 10.7.1 (via gstatic). 

- PDF.js: 2.10.377. 

- jsPDF: 2.3.1. 

**PART III: APPLICATION 1 - THE OPTIMIZER (this section could be outdated partially as we adjust to using Scanner (OCR scans of ESP) in replacement of the now inaccessible inventory PDF downloads.** 

## **A. CORE PURPOSE** 

1. Parse inventory files from Scanner. 

2. Fetch "Live" EDLP prices from the Cloud (CDN). 

3. Generate a condensed "Dashboard Style" PDF report to print. 

**B. VISUAL STRUCTURE** 

1. Header (Dashboard): Contains Date/Time, Store Info, Sources (Grouped by 30-minute window using a 2D Map/Reduce algorithm), and EDLP Citations. 

2. Footer: Contains the "Hidden Items" list (consolidated auto-hidden Unlocked devices, and smart-grouped manually hidden devices). 

3. Tutorial: Collapsible accordion (HOW TO USE 🔽) built with custom CSS max-height transitions. 

4. Accessibility: Status indicators must use icons (☁ / ❌/💾) instead of colors. 

5. Global Warning: A dynamic red warning banner that appears if any uploaded file is ≥ 4 hours old or from a future timezone (calculated via absolute math: Math.abs(hoursOld) >= 4). 

## **C. LOGIC ENGINES (THE "INVISIBLE RULES")** 

1. Data Transformation: Optimizer transforms fetched data (Device-centric) into display format 

(Carrier-centric) on the fly. 

2. Apple Isolation & Wearables Engine: If toggled, the app actively extracts iPhones and Apple Watches from carrier lists and builds a consolidated "Apple Devices" category. The UI Wearables toggle dynamically locks/unlocks based on whether watches are detected in the parsed PDFs. 

3. The Print Engine (No Downloads): Optimizer uses a Direct Print Engine. Generating the PDF creates a Blob URI. Desktop/WMPC loads it into a hidden iframe and calls .print(). Mobile safely falls back to window.open('_blank'). Pre-print modals and download fallback buttons are strictly forbidden to prevent WMPC hard drive bloat. 

4. The Vaporize Engine: Legacy devices (e.g., iPhone 11/12/13) are eradicated from the text stream via `vaporizeRegex` before any processing occurs. 

5. The Date Engine: The PDF header date generator pulls exclusively from the raw `appState`. It uses a highly tolerant regex (allows 2-digit years and missing seconds) and actively wipes old memory upon re-upload to prevent ghost dates. 

6. The "GB" Rule: Parser MUST normalize "\s+GB" to "GB" (e.g., "128 GB" -> "128GB"). 

7. The "Zero Filter": 

- If a price is $0 or missing, treat it as null. 

- Output "___" (three underscores) for missing prices to maintain visual alignment. 

- Citations: Do not generate citations ([1]) for missing/$0 items. 

8. T-Mobile Math Fallback: If "mo" is missing in DB but "total" exists, calculate (Total-DP)/24. 

9. Typo Detection: Regex must include "/un\s?-?\s?loc\s?k\s?e\s?d/i" to catch "unloc ked". 

10. Hardcoded Dictionaries: `HIDDEN_CLEANUP`, `STORE_NICKNAMES`, 

- `DEVICE_ABBREVIATIONS`, and `COLOR_ABBREVIATIONS` (e.g., 'Soft Pink': 'PNK') must be strictly preserved during rewrites. 

11. The Smart Flow Layout Engine: The PDF generator does not use static columns. It uses a dynamic queue (`att` -> `tmo` -> `vzw` -> `apple`) that calculates the pixel height of each carrier's inventory block. It attempts to place them in preferred columns (left/right) but will dynamically route them to the alternate column or force a page break if the remaining `pageHeight` is insufficient. 

**PART IV: APPLICATION 2 - INSTALLMENT MANAGER (IM)** 

## **A. CORE PURPOSE** 

The "Writer." Edits prices, saves to Firebase, and Publishes to GitHub. 

## **B. THE "PUBLISHER" ENGINE** 

1. Trigger: "Save All" button (and individual row save buttons). 

2. Action: Saves to Firebase -> Fetches full DB -> PUTs to GitHub. 

3. Delete Warning: Deleting a row removes it from Firebase immediately but REQUIRES a 

- "Save All" event to purge it from GitHub/CDN. 

## **C. MATH LOGIC (THE "ANCHOR" SYSTEM)** 

1. Total is the Anchor. Editing Total clears DP/Mo (Unknown state). 

2. DP and MOADP are "Architects." They calculate based on each other using Total as a constant. 

3. Positive Input Gate: Calculations only trigger if the input value is > 0. 

4. Zero Logic: Clearing a field (0) clears its partner field. 

## **D. UI & FEATURES** 

1. Lock System: Clickable 🔒/🔓 icons. Locked fields are immune to auto-calc. 

2. Validation: A ⚠ icon appears if the math (Total vs Parts) is off by > $0.24. 

3. Compatibility Layer: Logic to Hide/Gray out "Incompatible" items (e.g., Galaxy S series on T-Mobile) unless overridden by a toggle. 

## **PART V: APPLICATION 3 - PRINTS - this section likely outdated from newer changes** 

## **A. CORE PURPOSE** 

A Dashboard SPA to print specific store documents using JS Templates. 

## **B. DOCUMENTS** 

1. Take Home Sheet: 2-up, horizontal split. 

2. Intake Sheet: 4-up, quadrant split. 

3. Att promos (date last updated in its title) 

4. VZW promos also w date 

5. Guide for WARP calls 

- 6.. Verizon trade in guide 

7. iPhone transfer guide 

## **C. PHYSICAL CONSTRAINTS (THE "GHOST PAGE" FIX)** 

1. Typewriter Mode: Flexbox is banned for form lines. Must use Fixed Widths and vertical-align: bottom. 

2. CSS: @media print must have "height: 100%; overflow: hidden;" and page height set to 10.8in (not 11in) to prevent blank second pages. 

## **D. MOBILE LOGIC** 

1. "Tap-to-Return": Uses requestAnimationFrame and a one-time click listener to handle mobile print dialogs safely. 

2. Current Hotfix: "Take Home Sheet" is running in "Line Only Mode" (Opacity 0 text) for pre-printed forms. 

# **PART VI:  4 - PROJECT IRIS (THE SCANNER)** 

# **A. CORE PURPOSE** 

1. The Mobile "Write Path" to replace WARP 2.0 inventory prints. 

2. Uses the device camera or photo gallery to OCR inventory screens and format them perfectly for Optimizer. 

# **B. THE "SMUGGLER'S" PUBLISHER** 

1. Saves temporary scans to Firebase Realtime DB. 

2. The "Publish" button fetches the whole DB, formats it into **inventory_data.json** , and PUTs it directly to the GitHub Repo using a PAT stored in **localStorage** . 

# **C. THE OCR PIPELINE (THE 7-STAGE SANITIZER)** 

Because **isTable=true** causes the OCR API to spit out glued text and UI noise, IRIS uses a strict JS pipeline. Note: The camera/upload feed MUST be high resolution (4K ideal) or the OCR engine will fail to read dense inventory tables. 

1. **Garbage Cleaver V4:** Uses Pre-Split Wildcard regex to vaporize Walmart UI tabs (e.g., "Please select a row", "Prepaid"). 

2. **Ultimate Space Crusher V2:** Squashes all \s{2,} into single spaces to ensure identical strings for deduplication. 

3. **Auto-Fixer:** Forces "Available" to lowercase, injects missing "1"s for quantities, and fixes typos like **IPhone** . 

4. **Strict Blacklist:** Drops timestamps and isolated UI noise. 

5. **Smart Orphan Stitcher:** Glues broken lines back together, but uses a Brand Name dictionary ( **Galaxy** , **iPhone** , **Moto** , etc.) to prevent gluing two distinct phones together. 

6. **Dynamic Header Injector:** Automatically prefixes new scans with the exact **[Carrier] Date:** 

- **[Time] Store: [Store] Quantity** format that Optimizer demands. 

7. **Deduplicator:** Runs **[...new Set()]** to instantly vaporize overlapping lines caused by scrolling. 

# **D. UI MODES & THE CROP ENGINE** 

1. **Live Camera:** Requests a 4K feed with continuous autofocus. 

2. **Image Uploader (The Smart Crop Engine):** Allows users to pick high-res screenshots/photos. To handle perspective correction and aspect ratios gracefully, it utilizes a strict Two-Mode Toggle Architecture. **CRITICAL:** The toggle must be calculated on release (`touchend`/`mouseup`) by checking if the interaction was a "Tap" (distance < 10px and time < 300ms). It must NEVER toggle on `touchstart`, as this instantly ruins panning interactions.

3. **Mode A: Perspective & Edit Mode (The "Shape" Mode)**
- **Corner Handles:** 4 semi-transparent points. Dragging these alters the shape (Perspective Correction/Homography).
- **Edge Handles:** 4 thick white lines at the midpoints. Dragging these moves two corners simultaneously along an axis.
- **Aspect Ratio Lock (🔗):** A toggle that, when active, does NOT force a standard rectangle. Instead, dragging a corner calculates the distance to the diagonal opposite corner and applies a uniform scale multiplier, perfectly scaling the custom perspective shape up or down without altering the homographic ratios.
- **The Magnifying Glass (Loupe):** When dragging a corner, a hidden canvas grabs the exact pixels under the finger, magnifies them 2.5x, and displays them in a circular UI bubble offset above the finger. It uses a transparent crosshair with a center "winning circle" to strictly prevent thumb occlusion.

4. **Mode B: Viewport Pan/Zoom Mode (The "Window" Mode)**
- Triggered by tapping the selected zone. The handles vanish, and the crop box becomes a fixed "window".
- Dragging/panning inside the window applies inverse translation to the *underlying image*.
- Pinching inside the window applies scaling to the *underlying image* while the crop mask stays fixed in place.

# **E. CROP ENGINE PHYSICS & DEVELOPER WARNINGS (CRITICAL)**

1. **The "Squish" Bug & 0px Collapse Constraint:** NEVER force a static CSS aspect ratio (e.g., `aspect-ratio: 5 / 4`) on the crop container. Mobile cameras output native formats. The container must dynamically read the native aspect ratio (`naturalWidth / naturalHeight`). **CRITICAL FIX:** To prevent the container from collapsing to `0px` height when `aspect-ratio: auto` is set, JS must dynamically inject the image's native aspect ratio into the container's inline styles (`cropContainer.style.aspectRatio = ${w} / ${h}`) *before* measuring `clientHeight`.

2. **The Auto-Scale / Centering Tween (The Magic):** Future AIAs MUST preserve this math for the Mode A `touchend` release event:
- Calculate a target scale multiplier by dividing the available screen viewport dimensions by the new crop box dimensions.
- Apply that exact same scale multiplier to BOTH the crop mask and the underlying image container simultaneously.
- Compute the translation X/Y offsets needed to shift the center of the newly scaled crop box exactly to the center of the screen.
- Wrap all those property changes in a fast ~300ms animation curve (`transition: transform 0.3s`) triggered on the user's release so the transition looks buttery smooth.

3. **The "Single Engine" Rule (Ghost Engine Warning):** The Crop Engine uses a singular, unified interaction engine (`startInteraction`, `handleInteractionMove`, `endInteraction`). Future AIAs must NEVER attach secondary legacy drag listeners (like `startPtDrag` or `handleDragMove`) to the SVG elements. Doing so causes double-firing, 1-inch touch offsets, and totally broken physics.

4. **Dynamic UI Counter-Scaling:** Because the crop container scales up (up to 5x) when zooming into small areas, the SVG UI handles (blue dots, white lines) must dynamically *shrink* to prevent becoming massive. Future AIAs MUST preserve the math in `updateSVGUI()` where radius (`r`), stroke widths, and line lengths are actively divided by the current zoom multiplier (`cwScale`).

5. **Anti-Flicker CSS:** `-webkit-tap-highlight-color: transparent;` and `outline: none;` are strictly required on the crop container and SVG elements to prevent native mobile browser blue/grey flash overlays during rapid tapping.

## **PART VII: GLOSSARY & ACRONYMS** 

1. IMU: Installment Manager Unstable. 

2. OU: Optimizer Unstable. 

3. THS: Take Home Sheet. 

4. MOADP: Monthly After Down Payment (The white box). 

5. MONDP: Monthly No Down Payment (The red box). 

6. WMPC: Walmart PC. 

7. PAT: Personal Access Token (GitHub Key). 

8. CDN: Content Delivery Network (JSDelivr). 

9. Canary: A specific debug log entry (e.g., Galaxy A03) used to verify if data is fresh or stale. 10. PL: project lead (human) 

## **Part VII:default introductions and other communication items.** 

1. Introductions: If you are an AIA that just received this document, please read it thoroughly. Then, communicate to your project lead that you understand by showing two unique sample patches in perfect formatting. If you have any questions,.please inform the PL immediately. Please also briefly share with the PL in one to two sentences if you understand the project scopes, if you do. PL will likely start this conversation by only sending you the LDD PDF and an html file or a few. If so, plz also review that html file (usually index.html) to make sure u understand which project(s) are currently being worked on. The assurance of understanding from u need not be long unless otherwise noted by PL. By default, plz just briefly hit the high points to prove that u know what's going on (and send the sample patches). 

2. If the project lead mentions a form of media that SHOULD already be attached in the chat by the time you receive their message (cues such as “please see attached concept art, look at this picture, look at this document, etc etc), and such media is NOT presently attached, you are NOT to respond by saying ANYTHING except “error: you didn't attach media!”. This is to prevent excessive token usage when the user forgets to send media,this way they can attach the important item(s) without the AIA thinking harder than it should/wasting context limits. This rule can only be overridden, but ONLY explicitly by direct communication from your project lead on. Case by case basis. 

