# The design review

Token compliance is not design. A screen can use every hex value in `docs/DESIGN.md` and still be a grey column of 13px text with nothing to look at. A screen like that passes a token audit and is still boring. This file is the check that catches it.

Run it in **Sync** on any task that touched a screen, and in **Design pass** to diagnose before prompting. It takes five minutes and two screenshots.

## Get the evidence

Open the deployed URL yourself (`claude-in-chrome` or the built-in browser). Screenshot the screen at **1440 wide** and at **390 wide**. Have `docs/DESIGN.md` and the screen's mockup in `docs/design/` open beside them. If there is no mockup, that is finding number one.

## The score

Five checks, one point each. Be strict; a generous four is how the flat clone gets shipped.

| # | Check | How to run it | Pass looks like |
|---|---|---|---|
| 1 | **Squint test** | Blur the 1440 screenshot, or squint at it. | You can still tell what the most important thing on the screen is. One element clearly leads. Fail: a uniform grey texture. |
| 2 | **Five-second test** | Look at it fresh for five seconds. Say what the screen is for and what you would do first. | Both come easily. Fail: you had to read to find out. |
| 3 | **Space** | Measure the main content area against the viewport at 1440. Count the empty vertical space below the last element. | Main area uses the width the layout section allows. No centred column with empty sides on an app screen. Content or a deliberate empty state fills the height. Fail: 40 percent of the width empty, or the bottom half blank. |
| 4 | **Words** | Count the words on the screen that are not data (labels, helper text, explanations, status lines). | Within the DESIGN.md copy budget, normally under 60. Fail: helper text under every field, a paragraph per list item. |
| 5 | **Picture and motion** | Count the non-text elements: icons, sparklines, rings, bars, images, colour used for meaning. Hover one thing. | At least one hero visual, icons where the brief says, status carried by colour, and something responds to hover. Fail: numbers with labels and nothing else, and a static page. |

Then compare against the mockup: same layout, same hierarchy, same hero. Small drift in values is fine; a different layout is not.

**Four or five:** pass. Note the score in the PROGRESS entry.
**Three or under:** fail on design. Do not tick the task `[x]`. Mark it `[?]`, say which checks failed in one line each, and queue a Design pass as the next prompt before any new feature. The user should not have to say "it looks generic" for this to happen; you just looked, you know.

Also check at 390: nothing horizontally scrolls, the hero still leads, touch targets look 44px or more, and the sidebar has collapsed the way the layout section says.

## Diagnose before you prompt

A failing score has one of five causes, and each has a different fix. Prompting "make it look better" fixes none of them.

| Symptom | Cause | Fix |
|---|---|---|
| Layout differs from what the user expected; the screen is "fine" but not right | **No mockup**, or the mockup was not named in the prompt | Draft the mockup with the `design` skill, get their yes, commit the PNG, then a UI prompt that names it |
| Everything the same size and grey; squint test fails | **Hierarchy** was never specified for this screen | Fill in the Screens entry: hero, three tiers, which numbers get a picture. Then a UI prompt with those values |
| Narrow centred column, empty sides, empty bottom half | **Layout** section missing, or the 1200px max-width applied to an app shell | Fix the Layout section in DESIGN.md first, then a UI prompt that says "main area fills the viewport" and names the grid |
| Too much to read | **Copy budget** missing or ignored | Add the budget to DESIGN.md if absent. UI prompt that lists what to cut and where the explanation moves to (tooltip, guide page) |
| Values off: wrong greys, wrong sizes, wrong radius | **DESIGN.md too vague**, so Claude Code filled the gaps with defaults | Fix the file, not the screen. Then a prompt that points at the changed section |
| Nothing wrong with the design, it just looks empty | **Placeholder content** or no empty state | Real words and a designed empty state. Not a design problem |

Say which one it is in one line, fix the document first where the table says to, then write the UI prompt from `prompt-recipes.md`. The prompt names the mockup, the hero, the layout rule and the copy budget, with real values, and its done-checks include "screenshot at 1440 matches `docs/design/<screen>.png` in layout and hierarchy".

## Polish tasks

Every milestone in `docs/ROADMAP.md` ends with a polish task (`Mx-Tn Design polish: <screens>`). It exists so the review has a scheduled home rather than only running when something looks wrong. The prompt for it is the UI recipe with the whole milestone's screens in scope, and its done-checks are the five checks above scoring four or better on each screen. If the screens already score four or five in Sync, the polish task is ticked with a note and costs nothing.
