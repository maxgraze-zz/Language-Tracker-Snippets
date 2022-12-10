You can find your anki data by following this path in finder: `~/Library/Application Support/Anki2 folder`. Your collection will be under `collection.anki`. Remember this path, since you will need it for later. 

Install [Datasette](https://datasette.io/)
`brew install datasette`

datasette ~/Library/Application Support/Anki2 folder/data/collection.anki -o

## Tables in the database from [Josh](https://github.com/joshdavham/Short-Data-Analyses/blob/main/How%20does%20the%20word-frequency%20of%20my%20Anki%20cards%20change%20with%20time%3F.ipynb) [Anki Docs]([https://addon-docs.ankiweb.net/](https://docs.ankiweb.net/stats.html))
### `cards`

#### Schema
id -- (*primary key*) card id in epoch miliseconds (corresponding to when the card was created). The time at which the review was conducted, as the number of milliseconds that had passed since midnight UTC on January 1, 1970. (This is sometimes known as 'Unix epoch time', especially when in straight seconds instead of milliseconds.)



nid -- (*foreign key*) the card's corresponding notes id

type -- 0 = new, 1 = learning, 2 = review, 3 = relearning

reps -- number of times the card has been reviewed

lapses -- number of times the card went from a "was answered correctly" to "was answered incorrectly" state (meaning if you lapse once and then answer that same card incorrectly again, that won't cound as another lapse)

### `notes`

#### Schema
id -- (*primary key*) note id in epoch miliseconds (corresponding to when the note was created)

tags -- whether a note is retired or not

flds -- data contained in the fields of each note


![From Anki2 database documentation](https://user-images.githubusercontent.com/18546773/206876257-789e6030-77a8-40f3-be3f-1b03f48197a5.png)


### `revlog`

#### Schema

id -- (*primary key*) when I performed the review in epoch miliseconds (primary key)

cid -- (*foreign key*) The ID of the card that was reviewed. You can look up this value in the id field of the 'cards' table to get more information about the card, although note that the card could have changed between when the revlog entry was recorded and when you are looking it up. It is also the millisecond timestamp of the card’s creation time.

ease -- button pushed when reviewing the card -- review: 1(wrong), 2(hard), 3(ok), 4(easy) -- learn/relearn: 1(wrong), 2(ok), 3(easy)

ivl -- The new interval that the card was pushed to after the review. Positive values are in days; negative values are in seconds (for learning cards).

lastIvl -- previous interval

time -- how long the review took in miliseconds up to 60000 miliseconds (60 seconds)

type -- what type of card at the time of review --0 = learn, 1 = review, 2 = relearn


## Useful Joins
### `cards` INNER JOIN `notes`
Match cards to notes. Useful for:
- See how many times a card was reviewed before retired 
- Average number of lapses

```
SELECT *
FROM cards
    JOIN notes 
        ON cards.nid = notes.id
LIMIT 3
```

### `revlog` LEFT JOIN `cards`
Match reviews to cards. Useful for:
- Seeing the amount of time spent reviewing in relation to lapses 

```
SELECT *
FROM revlog
    LEFT JOIN cards 
        ON revlog.cid = cards.id
ORDER BY revlog.cid ASC
LIMIT 10
OFFSET 346
```
