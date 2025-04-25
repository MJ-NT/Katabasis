# Katabasis
#### Video Demo:  <URL HERE>

## Introduction

_Katabasis_ is an interactive fiction browser game based on the world of Harry Tuffs' 2D exploration RPG [_A House of Many Doors_](https://www.harrytuffs.com/a-house-of-many-doors), drawing design and aesthetic inspiration from the Failbetter browser game [_Fallen London_](https://www.failbettergames.com/games/fallen-london).

In it, the player character takes on the role of a ‘visitant’, a being stolen from another world by the parasite dimension that is the ‘House’.

> It is a mad, disturbing sprawl of wonders and terrors, but you don’t have time to worry about all that right now. Plucked from the fantastic dark only to be pitched through a mirror and set adrift in a kaleidoscopic dimension, you are propelled through the House as blood quickened by the heart.
 
> Find the next mirror. Delve ever deeper. Do not look back.

I created the game as my final project for Harvard's CS50 course.

The backend is powered by Flask, with a SQLite database to store user data and game state.
The frontend is built with HTML, CSS, and JavaScript, using Bootstrap for styling and layout.

## Background and creative process

_A House of Many Doors_ (henceforth referred to as HOMD) was a solo effort, written and designed by the one-man team of Harry Tuffs. It is a behemoth: wildly ambitious in scope, both narratively and technically. According to an AMA he ran on the game's discord, Tuffs had never written so much as a 'Hello World' before creating HOMD, and by his own admission, the code - written using GameMaker - is 'nightmare spaghetti'. Despite the enjoyment he gained from writing the content, it’s clear that the game became unwieldy due to this coding quagmire.

Perhaps unsurprisingly then, after landing on the idea to create a browser game based on the world of HOMD, I was acutely aware of the limits of my own technical knowhow. My comparative deficiencies as a writer aside, I needed to temper my ambitions to make what I wanted narratively achievable in code, particularly given my shorter development timeframe.

I therefore decided to narrow the scope of the game to a single activity within the engaging whole: delves. These are 'dungeons' of a sort: contained narrative experiences in which the player character explores a series of descending floors, finally arriving at a central location with a final revelation or choice (that in the original HOMD would have consequences that stretched into the game exterior to the delve).

In HOMD, the titular House contains 4 such delves: the Underbelly of Carapas, the Founder in Founder's Fire, the Svadilfari in Pannachak, and the Stupefaction Baths in Phobetor Quinn.

For _Katabasis_, I opted to create a new delve. Since I wanted to explore as much of the world of HOMD as possible within my limited scope, I set both floors of my delve in regions of the House that do not feature a delve in the original HOMD. Moving between floors is possible through the ‘Mirrorwise’, which is a feature of the fiction that can best be described as an alternate dimension that allows travel between mirrors.

### Player stats

In HOMD, players have 8 stats. I opted for the more streamlined 4 stats used in _Fallen London_: ‘Watchful’, ‘Shadowy’, ‘Dangerous’ and ‘Persuasive’.

With the stats decided upon, I turned to the method of progression. In _Fallen London_, Change Points (CP) are awarded every time you undertake a challenge. The number of CP you receive is dependent on your chance of success at completing the challenge (easier challenges award fewer CP), and whether you succeed or fail (success earns more CP than failure). A number of CP proportionate to your level (until level 70) is required to increase the associated stat.

After tinkering with this kind of system in code, I decided that it would be far easier to return to my original inspiration and implement a system like that in HOMD, or _Sunless Seas_, another Failbetter game. In these games, stats are improved through spending a resource. In HOMD, this resource is 'Apprehensions', gained through progression in various stories (in addition to visiting locations and writing poetry!). For my game, I settled on awarding a changing number of apprehensions through successes and losses on each challenge.

Finally, I abstracted away health, sanity, or other named ‘threat level’ stats to one single stat called ‘Menace’, which increases when players fail certain challenges, and sometimes when they succeed on others. When Menace reaches 100, the player must restart the game (see ‘overcome.html’).

### Narrative structure

I settled on a ‘branch-and-bottleneck’ approach to this ‘choose-your-own-adventure’-style narrative, with occasional story-ending decisions similar to a ‘gauntlet’ structure. The player’s ‘story state’ denotes their progress within the story, and each challenge is associated with a particular story state.

![A diagram of a branch-and-bottleneck story structure.](https://emshort.blog/wp-content/uploads/2019/11/branchbottleneck.jpg?w=768)

‘Storylets’ are pieces of content that have prerequisites determining when they can play and have an effect on the world state after playing them. If you are interested in reading more about the uses of storylets, Emily Short gives a more comprehensive overview [here](https://emshort.blog/2019/11/29/storylets-you-want-them/).

![A diagram of a branch-and-bottleneck story structure using storylets, with example ending storylets labelled 'Revenge', 'Love' and 'Prison'](https://emshort.blog/wp-content/uploads/2019/11/bb-storylets-2.jpg)

I debated using a bank of storylets (i.e. a list of challenges) associated with the same story state (or alternatively a given range of story states), and implementing code that would fire a random challenge from this list when the player state was equal to that state (or within that range). Story state would then be increased to a new level with a new bank of associated storylets. Instead, I opted for a more traditional narrative-branching structure, still using storylets, with each storylet’s story state being unique but returning the player to bottleneck storylets with milestone story state values.

I mapped out the story structure as follows:

| Narrative section | Story state |
| --- | ---
| Introduction | 0-50 |
| Mirrorwise Transition 1 | 100 |
| Ghoulwatch Delve | 101-195 |
| Mirrorwise Transition 2 | 200 |
| Empire of Thread Delve | 201-250 |
| Ending | 275-300 |

The introduction is more linear than the two delves and contains 3 stat checks (i.e. 3 challenges) to serve as tutorials. The first is used to demonstrate apprehension gain through success and failure on a stat check, the second is made impossible to demonstrate menace gain, and the third is used to show how spending apprehensions increases your chances of success.

Each transition bottlenecks the players to the journey through the Mirrorwise to the next level. Each has 4 options (1 associated with each stat) to navigate the Mirrorwise. The first transition therefore introduces the idea of using different stats.

The delves branch and bottleneck, particularly at the transitions (or at the Ending). The Ending has one final choice: 2 challenges to determine which of 4 endings you get (1 ending for success and 1 for failure for either option).

## The Code:

## Python files

### app.py:

This file contains the bulk of my code.

I enable Cross-Origin Resource Sharing with `CORS(app)`.

In my setup, sessions are considered permanent using `app.config["SESSION_PERMANENT"] = True` in order to preserve player stats and story state when the player closes the browser and comes back to the game later. I set `PERMANENT_SESSION_LIFETIME` to determine how long they should last (putting in 14 days as a placeholder, since this is not a very big game and can be completed in one sitting). Since this is a small-scale project, it doesn’t store sensitive information, and I am not expecting many concurrent users, I decided to stick with client-side sessions using ‘filesystem’. I implemented extra security measures to prevent session-tampering.

The `/` route queries the database for the user’s username and renders the home page, passing that username to index.html.

The `/about` route renders the about page. I used `target="_blank" rel="noopener noreferrer">` within the anchor tags to have links open in new tabs and not be vulnerable to phishing attacks described in [this article](https://dev.to/ben/the-targetblank-vulnerability-by-example).

The `/account` route queries the database for the user’s username and password, then processes changing their password, handling all related errors (“User not found”, “Invalid current password”, “New password cannot be the same as the old password”, etc). The new password is hashed and the users table updated.

The `/challenge` route first connects to the database and queries the Menace level for the user logged in. If `result["menace"] >= 100` then the route redirects to overcome.html. Otherwise, the route queries for the user’s story state and subsequently queries for the options associated with the challenge with that story state. If no options are found, then an ending message story-text is passed into the challenge.html.

If the user reached the route via the GET method, it fetches all 4 user stats, creates a dictionary called ‘challenge_data’ and iterates through each challenge option to calculate its success chance using the formula Fallen London uses: `success_chance = int((SCALER * stat_value) / opt["difficulty"] * 100)`, where SCALER is the constant 0.6 defined at the top of app.py. The route then appends the resulting data to the ‘options’ key as a list of dictionaries, containing the option text, number, the stat it tests and the success chance. These are then rendered in the challenge.html.

If the user reached the route via POST, it extracts the selected option from form data, then implements a generator expression to find the chosen option and handle errors with the default value None for the next() function. The route then queries for the user’s stat level for the stat tested by the chosen option, identifies the difficulty of the chosen option and calculates the user’s success chance according to the aforementioned formula. User success is measured against a random number generator: 
```
rng = random.randint(1, 100)
success = rng <= success_chance
```
The user’s Apprehensions, Menace, and story_state are all updated according to success or failure, as defined in each option in `challenges.py`.

In my `/login` route, a user’s username and password are queried in the users table, redirecting them to the home page on a success, flashing a success message. I ran into a problem here whereby clearing session data with `session.clear()` in the `/login` (and `/register`) routes caused the flashed error messages (for “invalid username or password” etc) to not appear upon redirecting. I have implemented a workaround by saving the flashed messages as a list via `session.get("_flashes", [])` and reflashing once the session was cleared. With this method, no sensitive information is retained, the flashed messages are stored client-side, and memory concerns are minimal since the messages are text-only.

The `/myself` route queries the user and their stats from the database and assigns the story text to None. If the user reached the route via GET, the myself.html page is rendered with the user’s name, current stats and no story text. If the user reaches the route via POST, the route gets the stat and amount from the form, handles Value and Type errors out of an abundance of caution (since the form uses a dropdown menu) and instances where the user doesn’t have enough Apprehensions to improve the stat. It then defines valid stats to guard against SQL-injection attacks so it can use an f-string to update the user’s stats and Apprehensions. The user’s stats are then refetched and the story text is created based on the stat improvement.

The `/overcome` route renders the overcome.html when reached by GET. This only happens via the challenge route when the user’s Menace is greater than or equal to 100. If reached via the POST method, the route will reset the user’s stats to 1, their Menace to 0 and their story_state to 0, flashing a success message and returning them to the homepage. This will occur when a user clicks the “Begin Anew” button on the overcome.html page or the “Restart Game” button on the challenge.html page when story text is present.

The `/register` route is very similar to the login route, again handling for invalid details or cases in which the username is already taken. It inserts new users into the database and hashes their passwords.

Finally, the `/logout` route clears session data, flashes a success message and redirects the user back to the login page.

### challenges.py

This file contains a list of dictionaries containing the ‘challenges’ (i.e. the story content). Each challenge contains a story state, text describing the challenge, and an inner list of dictionaries containing the options available to address the challenge.

Each option has within it the text, the stat it tests against, the difficulty of the test, and a number of values tied to keys corresponding to success or failure: the text displayed, the next (story) state it directs to, apprehension change and menace change.

Originally, I used a json file (‘stories.json’). With this approach, I found I had to use a lot of escape characters and unicode codes (e.g. /n for line breaks or \u2018 and \u2019 for curly apostrophes) in my text. I therefore switched to python, where triple quotes can be used for multi-line strings and escape characters are much easier to handle.


### helpers.py

This file contains functions to be imported into app.py.

`get_db_connection` connects to `game.db`. By default, sqlite3 represents each row as a tuple. Using `conn.row_factory = sqlite3.Row` returns a dictionary, allowing access to the column names when calling the function, as per the documentation: “Row provides indexed and case-insensitive named access to columns, with minimal memory overhead and performance impact over a tuple.” [source](https://docs.python.org/3/library/sqlite3.html)
`login_required(f)` is a decorator function that only allows the original function f (such as account or myself) to run if `session["user_id"]` exists (i.e. the user is logged in).

### import_stories.py

This file imports the challenges from the `challenges.py` file. It connects to ‘game.db’, deletes old data from the challenges table, then uses the imported challenges to repopulate it using `SQL INSERT`.

Since each challenge entry may have multiple options (i.e. the aforementioned list of dictionaries), this necessitates the nested for loop `for i, option in enumerate(story["options"], start=1)` to iterate through each option within that challenge and assign it a number, starting at 1. This number is inserted into the challenge table’s ‘option_number’ field.

It prints a message in the terminal upon success.


### reset_db.py

This file connects to game.db, drops any existing tables therein, and recreates them in the database. It prints a message in the terminal on success.

I created this file because I found that, as my game increased in complexity, I was creating more tables and adding columns to existing tables in the terminal. It became simpler to write a python file that could be easily edited to update table structure and ensure that the schema was as readable as possible. It also allowed me to wipe user data and test routes more fully.


## Static files

### script.js

This file contains almost all the Javascript used in the templates. The bulk of the file is an asynchronous AJAX handler for submitting the stat form (i.e. spending your Apprehensions) on the ‘Myself’ page. It prevents the default page reload from submitting the form, packages the form data, sends it to the server and retrieves it as JSON, then updates the page dynamically with the new stat levels and the new number of Apprehensions available to spend.


### styles.css

The css file expands upon the styling already implemented in Bootstrap.

Among other features:

- I altered the colour scheme, margins, padding, and text-alignment of multiple elements, which was important to establish the tone of the game.
- I added a page-wrapper class (and added it to the `<main>` tag in layout.html) to apply a muted container wrapper around content.
- I added borders to headings and altered the Bootstrap card class to apply borders as well. In the myself page, for example, they provide a cleaner layout.
- I added pseudo-classes to buttons, the nav-bar links, and ‘about’ page hyperlinks to make the pages more dynamic and tonally consistent.
- I added a fade-in class to the challenges to fade in content using the `@keyframes` rule.
- I implemented a `.prelanding` class to create a smaller container box around the account, login and register forms that would change size dynamically with the content within.
- I created a fixed footer using:
. Bootstrap’s `flex-grow-1` on the container class for `<main>` (which grows the container to take up all available space, pushing the footer down if there is not enough content to do so naturally).
. Bootstrap’s `mt-auto` for the footer (which pushes the footer down to the bottom when the content is shorter than the screen height).
. I added `display: flex` to the body, turning it into a flex container. Also `flex-column`, which changes the flex direction to be vertical, so the content and footer are stacked vertically.


## Templates

### about.html

This page describes what the game is. It references my influences and inspiration for the project - namely ‘A House of Many Doors’ and ‘Fallen London’ - and provides links to both. It also contains a content warning and copyright, credits and technical information.

### account.html

This page offers users the option to change their password through submitting a form.

### challenge.html

This page is the template for all challenges. It uses jinja syntax and conditional logic throughout:
1) To display the challenge text. The syntax `{{ challenge.text | replace('\n', '<br>') | safe }}` allows line breaks from the challenges dictionary to be retained in the HTML.
1) To populate a form with radio input-type options to respond to challenges if a challenge is available.
2) To populate a customised alert box with ‘story text’ (dependent on success or failure in the previous challenge) if available and implement a ‘Continue’ button. A ‘Restart Game’ button is always visible after story text and will reset the user’s story state and stats via the /overcome route. One line of inline Javascript is used to prompt the user for confirmation should they click this button.

### index.html

This is the home page. It contains a couple of headings and a button directing users to the challenge page (and back to the game).

### layout.html

This file provides the base template for all other .html files using jinja.
In the `<head>`, it uses `<meta charset="UTF-8">` as is standard, and ensures mobile compatibility by using `<meta name="viewport" content="width=device-width, initial-scale=1.0">`. It links in custom CSS from styles.css and also Bootstrap CSS. Finally in the header, it links in custom javascript from ‘script.js’ and also Bootstrap javascript, using `defer` to wait until HTML and CSS content are fully loaded before triggering.

Because I altered the way messages are flashed by adding categories from Bootstrap (e.g. `danger` or `success`) to the routes, I also altered the global function `get_flashed_messages` to `get_flashed_messages(with_categories=true)`. This returns a list of tuples, where the first element is the category (i.e.success or danger), and the second element is the message itself. This is implemented within a jinja conditional loop.

In `<body>`, layout.html implements a navbar, using conditional logic and jinja syntax to determine what is visible depending on whether a user is logged in. Alert messages (if present) are then handled in a `<header>` tag within a conditional loop. The page content of .html pages extending this template is held within `<main>` tags. Finally, the `<footer>` element contains a single line “Created by Matt Taylor”.

### login.html

This page contains a form in which users submit their username and password to gain access to the game.

### myself.html

On this page, users can see their attribute stats, number of Apprehensions, and menace level in three stylised ‘cards’ in a Bootstrap grid system. Players can improve their attributes on a one-to-one basis by spending apprehensions in the middle form. They can return to the game by clicking the ‘Return’ button at the bottom of the page.

### overcome.html

This is the page that the `/challenge` route renders if the user’s menace reaches (or passes) 100. The button ‘Begin Anew’ resets the user’s stats and story state, allowing them to restart the game.

### register.html

This page contains a form in which new users can register by providing a username, a password, and a confirmation of that password.

## Copyright and Credits

This game was written and designed by Matt Taylor.
It is a work of fan fiction based on Harry Tuffs' _A House of Many Doors_ and is not intended for commercial use.
All rights to the original game and its characters, settings, and lore are owned by Harry Tuffs.

