# Closet-App
Closet Organizer using Oracle SQL
# Closet App

A closet organizer and style tracker built on an Oracle database using standard ANSI SQL. It keeps track of some of my wife's clothing, where each item is stored, which outfits she builds, and how often she wears each piece. I will be updating it until the wardrobe is complete. Just need proof of concept first. So minimal data was used.

**Status:** In progress. Database built; queries and web app in development.

## Why I built it

I'm teaching myself SQL and database design to move from restaurant operations into IT support. I wanted a project that could solve a serious issue/be useful and an unorganized closet does the trick. It made the most sense since it's a small inventory system: items, categories, locations, and usage history.

## Tables

| Table | What it stores |
| categories | Types of clothing (tops, pants, shoes, etc.) |
| locations | Where items are kept (closet, dresser, storage bin, laundry) |
| items | Each clothing item with color, season, brand, price, and purchase date |
| outfits | Named outfits and the occasion they're for |
| outfit_items | Links items to outfits (one outfit has many items; one item can be in many outfits) |
| wear_log | Each time an item was worn |

## Planning the tables

I sat down with my wife (the app's user) to decide what the app needed to track:

- Her clothing items and details like color, season, brand, and price
- Categories, so items can be grouped (tops, shoes, etc.)
- Where each item is kept (closet, dresser, storage bin, laundry)
- Outfits, and which items make up each one
- Each time something is worn

That turned into six tables. One design choice: an outfit has many items, and one item can be in many outfits, so I added a bridge table, outfit_items, to connect them.
## Questions it can answer

- What items are in each category or color?
- What's right for the current season?
- Which items have never been worn?
- How many times has each item been worn?
- What's the cost per wear of each item?

## Files

- CreateTables.sql: creates the six tables and inserts data
- Closet_queries.sql: report queries
- troubleshooting.md: problems I hit while building and how I fixed them or plan to

## What I learned

- Designing related tables with primary keys and foreign keys
- Using constraints (NOT NULL, UNIQUE, CHECK) to keep data clean
- Reading Oracle error messages and fixing the first error first, since later errors are often side effects
- Using COMMIT to save data changes and ROLLBACK to undo them.
- Cross JOIN to link Rows specifically connecting to the Outfits Table (Creating Separate Aliasases) 
- 
## What's next

- Run ALL SQL reports (Working through a report query for the most expensive "OUTFIT" using a bridge table")
- PowerShell script automation on local storage
- A simple web app so she can browse her closet and plan outfits from her phone


## Tools

Oracle Live SQL, SQL (ANSI), GitHub
