# Troubleshooting Log

Problems I ran into while building the closet database in Oracle Live SQL, and how I fixed them.

## 1. Typos in the CREATE TABLE command

**Problem:** Creating the categories table failed with ORA-02000: missing ( keyword.

**What I checked:** I compared my code line by line to the example, starting near the line number in the error.

**Cause:** Two typos. I left out the word IDENTITY (I wrote GENERATED ALWAYS AS PRIMARY KEY), and I spelled UNIQUE as UNQIQUE. (typing too fast)

**Fix:** Changed it to GENERATED ALWAYS AS IDENTITY PRIMARY KEY and fixed the spelling of UNIQUE.

**What I'd do differently:** Check the spelling of keywords first. Oracle won't guess what I meant.

## 2. Table name mismatch: "location" vs. "locations"

**Problem:** Creating the items table failed with ORA-00942: table or view LOCATIONS does not exist, even though I had already made a locations table.

**What I checked:** I ran SELECT table_name FROM user_tables; to see what tables actually existed.

**Cause:** I had named the table location (no "s"), but the items table looks for locations. To Oracle those are two different names. After fixing it I had both tables.

**Fix:** Created locations, then removed the extra one with DROP TABLE location;

**What I'd do differently:** Pick one naming rule (I use plural names for all tables) and check user_tables before building tables that depend on each other.

---

## 3. One season per item: a design limit

**Problem:** While entering her clothes, I found that many items work in more than one season. For example, a top she wears in spring and summer but not winter. The season column only holds one value, and its CHECK rule only allows 'Spring', 'Summer', 'Fall', 'Winter', or 'All'.

**What I checked:** I thought about typing 'Spring, Summer' into the one column. That breaks the CHECK rule, and even if it didn't, a search like WHERE season = 'Summer' would miss it.

**Cause:** The table was designed as if each item belongs to just one season. In reality, one item can have many seasons, and one season has many items. So only one column can't store it.

**Workaround for now:** I marked items like this as 'All' or picked their main season, so I could keep entering data.

**What I'd do differently:**  Add a bridge table, item_seasons with one row per item per season (like outfit_items does for outfits). Then the top gets two rows, Spring and Summer, and "what can she wear in winter?" will return the right items.

## 4.  Two items with the same name

**Problem:** I entered two pairs of Tecovas Annie Boots, one red and one black, both named Tecovas Annie Boot. Oracle accepted both without an error, because item_name doesn't have a UNIQUE rule.

**What I checked:** When I started linking items to outfits and the wear log by name, I ran:
SELECT item_id, item_name, color FROM items WHERE item_name = 'Tecovas Annie Boot';
It returned two rows, so there was no way to tell which boot was meant.

**Cause:** Duplicate names aren't wrong to Oracle, but they make lookups by name ambiguous. A query could log a wear for the wrong pair, or for both.

**Fix:** I used UPDATE to give each pair a unique name based on its color:
UPDATE items SET item_name = 'Tecovas Annie Boot (Red)' WHERE item_name = 'Tecovas Annie Boot' AND color = 'Red';
UPDATE items SET item_name = 'Tecovas Annie Boot (Black)' WHERE item_name = 'Tecovas Annie Boot' AND color = 'Black';
Then I ran COMMIT to save it.

**What I'd do differently:** Include the color or another detail in the name whenever there's more than one of the same item.

---

## 5.  Item IDs skipped numbers (34 to 221)

**Problem:** After fixing some errors and re-entering items my item IDs jumped from 34 straight to 221 instead of continuing in order.

**What I checked:** I ran SELECT item_id, item_name FROM items ORDER BY item_id; and saw the gap. I had recently had INSERT statements fail with errors, and I had deleted my shoe rows and re-entered them.

**Cause:** The item_id column is GENERATED ALWAYS AS IDENTITY, so Oracle assigns the numbers itself.
- When an INSERT failed, it could still use up an ID number.
- When I deleted rows their ID numbers were gone.
- Unused numbers in a batch get skipped which explains a big jump.

**Fix:** No fix needed. Gaps in ID numbers are normal, because an ID is a label, not a count. I can't renumber them anyway.
**What I'd do differently:** Check that a statement is correct before running it, and not assume a gap in IDs means data is missing. Something that looks like a bug can be normal system behavior so the first step is understanding why it happens.

---

## 6. Learned: CROSS JOIN

**What I was trying to do:** Link items to outfits in outfit_items without looking up every ID number by hand.

**What I learned:** A CROSS JOIN pairs every row in one table with every row in another. For example, 7 outfits × 45 items = 315 combinations. Adding a WHERE keeps the pairs I was looking for, like the items in one outfit:

INSERT INTO outfit_items (outfit_id, item_id)
SELECT o.outfit_id, i.item_id
FROM outfits o
CROSS JOIN items i
WHERE o.outfit_name = 'Date Night'
  AND i.item_name IN ('Black Mini Dress', 'Shoulder Bag');

The o and i are aliases, short names for the tables, so o.outfit_id means the outfit_id column from outfits.

**Takeaway:** CROSS JOIN lets SQL find the IDs by name. It should have a WHERE or it creates every possible combination.
