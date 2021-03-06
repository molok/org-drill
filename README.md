# Introduction

Please see [the history](#history) of this repository.

Org-Drill is an extension for [Org
mode](http://orgmode.org/). Org-Drill uses a [spaced
repetition](http://en.wikipedia.org/wiki/Spaced_repetition) algorithm
to conduct interactive "drill sessions", using org files as sources of
facts to be memorised. Each topic is treated as a "flash card". The
material to be remembered is presented to the student in random
order. The student rates his or her recall of each item, and this
information is used to schedule the item for later revision.

Each drill session can be restricted to topics in the current buffer
(default), one or several files, all agenda files, or a subtree. A single
topic can also be drilled.

Different "topic types" can be defined, which present their information to the
student in different ways.

For more on the spaced repetition algorithm, and examples of other programs
that use it, see:

-   [SuperMemo](http://supermemo.com/index.htm) (see descriptions of the SM2, SM5 and SM8 algorithms)
-   [Anki](http://ichi2.net/anki/)
-   [Mnemosyne](http://mnemosyne-proj.org/index.php)

# Installation

Org-Drill is available on MELPA. You can install it with `M-x 
package-install RET org-drill RET`.

The easiest way is to customise the variable `org-modules` (`M-x
customize-variables RET org-modules`) and make sure 'drill' is
ticked. Org-drill will then be loaded when you restart Emacs or restart
Org-mode.

For manual installation, put the following in your `.emacs`:

    (require 'org-drill)

# Demonstration

Load the file 'spanish.org'. Press `M-x` and run the function `org-drill`. Follow
the prompts at the bottom of the screen.

When the drill finishes, you can look at 'spanish.org' to get some idea of how
drill topics are written.

# Writing the questions

Org-Drill uses org mode topics as 'drill items'. To be used as a drill item,
the topic must have a tag that matches the value of
`org-drill-question-tag`. This is `:drill:` by default. Any other org topics
will be ignored.

Drill items can have other drill items as children. When a drill item is being
tested, the contents of any child drill items will be hidden.

You don't need to schedule the topics initially.  Unscheduled items are
considered to be 'new' and ready for memorisation.

How should 'drill topics' be structured? Any org topic is a legal drill topic
&#x2013; it will simply be shown with all subheadings collapsed, so that only the
material beneath the main item heading is visible. After pressing a key, any
hidden subheadings will be revealed, and you will be asked to rate your
"recall" of the item.

This will be adequate for some items, but usually you will want to write items
where you have more control over what information is hidden from the user for
recall purposes. For this reason, some other card types are defined, including:

-   Two-sided cards
-   Multi-sided cards
-   Multi-cloze cards
-   User-defined card types

**A note about comments:** In org mode, comment lines start with '#'. The rest of
the line is ignored by Org (apart from some special cases). You may sometimes
want to put material in comments which you do not want to see when you are
being tested on the item. For this reason, comments are always rendered
invisible while items are being tested.

## Simple topics

The simplest drill topic has no special structure. When such a topic is
presented during a drill session, any subheadings are "collapsed" with their
contents hidden. So, you could include the question as text beneath the main
heading, and the answer within a subheading. For example:

	* Item                                   :drill:
	What is the capital city of Estonia?
	
	** The Answer
	Tallinn.

When this item is presented for review, the text beneath the main heading will
be visible, but the contents of the subheading ("The Answer") will be hidden.

## Cloze deletion

Cloze deletion can be used in any drill topic regardless of whether it is
otherwise 'simple', or is one of the specialised topic types discussed
below. To use cloze deletion, one or more parts of the body of the topic is
marked as *cloze text* by surrounding it with single square brackets, [like
so]. When the topic is presented for review, the text within square brackets
will be obscured. The text is then revealed after the user presses a key. For
example:

	* Item                                   :drill:
	The capital city of Estonia is [Tallinn].

During review, the user will see:

> The capital city of Estonia is <font style="background-color: blue;" color="cyan">
> <tt>[&#x2026;]</tt></font>.

When the user presses a key, the text "Tallinn" will become visible.

## Clozed text hints

Clozed text can contain a "hint" about the answer. If the text surrounded
by single square brackets contains \`||' (two vertical bars), all text
after that character is treated as a hint. During testing, the hint text will
be visible when the rest of the text is hidden, and invisible when the rest of
the text is visible.

Example:

    Type 1 hypersensitivity reactions are mediated by [immunoglobulin E||molecule]
    and [mast cells||cell type].

> Type 1 hypersensitivity reactions are mediated by
> <font style="background-color: blue;" color="cyan">
> <tt>[molecule&#x2026;]</tt></font>
> and <font style="background-color: blue;" color="cyan">
> <tt>[cell type&#x2026;]</tt></font>.

## Two-sided cards


The remaining topic types all use the topic property, `DRILL_CARD_TYPE`. This
property tells `org-drill` which function to use to present the topic during
review. If this property has the value `twosided` then the topic is treated as
a "two sided card". When a two sided card is reviewed, *one of the first two*
subheadings within the topic will be visible &#x2013; all other
subheadings will be hidden.

Two-sided cards are meant to emulate the type of flipcard where either side is
useful as test material (for example, a card with a word in a foreign language
on one side, and its translation on the other).

A two sided card can have more than 2 subheadings, but all subheadings after
the first two are considered as "notes" and will always be hidden during topic
review.

	* Noun                                               :drill:
	    :PROPERTIES:
	    :DRILL_CARD_TYPE: twosided
	    :END:
	
	Translate this word.
	
	** Spanish
	la mujer
	
	** English
	the woman
	
	** Example sentence
	??Qui??n fue esa mujer?
	Who was that woman?

In this example, the user will be shown the main text &#x2013; "Translate this word"
&#x2013; and either 'la mujer', *or* 'the woman', at random. The section 'Example
sentence' will never be shown until after the user presses a key, because it is
not one of the first two 'sides' of the topic.

## Multi-sided cards


The `multisided` card type is similar to `twosided`, except that any
subheading has a chance of being presented during the topic review. One
subheading is always shown and all others are always hidden.

	* Noun                                               :drill:
	    :PROPERTIES:
	    :DRILL_CARD_TYPE: multisided
	    :END:
	
	Translate.
	
	** Spanish
	la mesa
	
	** English
	the table
	
	** Picture
	[[file:table.jpg][PICTURE]]

The user will be shown the main text and either 'la mesa', *or* 'the table',
*or* a picture of a table.

## Multi-cloze cards


Often, you will wish to create cards out of sentences that express several
facts, such as the following:

    The capital city of New Zealand is Wellington, which is located in the
    North Island and has a population of about 400,000.

There is more than one fact in this statement &#x2013; you could create a single
'simple' card with all the facts marked as cloze text, like so:

    The capital city of [New Zealand] is [Wellington], which is located in
    the [North||North/South] Island and has a population of about [400,000].

But this card will be difficult to remember. If you get just one of the 4
hidden facts wrong, you will fail the card. A card like this is likely to
become a leech.

A better way to express all these facts using 'simple' cards is to create
several cards, with one fact per card. You might end up with something
like this:

	* Fact
	The capital city of [New Zealand] is Wellington, which has a population of
	about 400,000.
	
	* Fact
	The capital city of New Zealand is [Wellington], which has a population of
	about 400,000.
	
	* Fact
	The capital city of New Zealand is Wellington, which has a population of
	about [400,000].
	
	* Fact
	The capital city of [New Zealand] is Wellington, which is located in the
	the North Island.
	
	* Fact
	The capital city of New Zealand is [Wellington], which is located in
	the North Island.
	
	* Fact
	The capital city of New Zealand is Wellington, which is located in
	the [North||North/South] Island.

However, this is really cumbersome. Multicloze card types exist for this
situation. Multicloze cards behave like 'simple' cards, except that when there
is more than one area marked as cloze text, some but not all of the areas
can be hidden. There are several types of predefined multicloze card:

1.  `hide1cloze` &#x2013; one of the marked areas is hidden during review; the others
    all remain visible. The hidden text area is chosen randomly at each review.
    (Note: this type used to be called 'multicloze', and that card type is
    retained as a synonym for 'hide1cloze'.)
2.  `show1cloze` &#x2013; only one of the marked areas is visible during review; all
    the others are hidden. The hidden text area is chosen randomly at each
    review.
3.  `hide2cloze` &#x2013; like hide1cloze, but 2 marked pieces of text will be hidden,
    and the rest will be visible.
4.  `show2cloze` &#x2013; like show1cloze, but 2 marked pieces of text will be visible,
    the rest are hidden.

There are also some types of multicloze card where some pieces have an
increased or decreased chance of being hidden. These are intended for use when
studying languages: generally it is easy to translate a foreign-language
sentence into your own language if you have met it before, but it is much
harder to translate in the other direction. Therefore, you will want to test
the harder direction more often.

1.  `hide1_firstmore` &#x2013; only one of the marked pieces of text will be
    hidden. 75% of the time (guaranteed), the *first* piece is hidden; the rest
    of the time, one of the other pieces is randomly hidden.
2.  `show1_firstless` &#x2013; only one of the marked pieces of text will be
    visible. Only 25% of the time (guaranteed) will the *first* piece will be
    visible; the rest of the time, one of the other pieces is randomly visible.
3.  `show1_lastmore` &#x2013; only one of the marked pieces of text will be
    visible. 75% of the time (guaranteed), the *last* piece will be visible;
    the rest of the time, one of the other pieces is randomly visible.

So, for the above example, we can actually use the original 'bad' simple card,
but change its card type to 'hide1cloze'. Each time the card is presented for
review, one of 'New Zealand', 'Wellington', 'the North Island' or '400,000'
will be hidden.

	* Fact
	  :PROPERTIES:
	  :DRILL_CARD_TYPE: hide1cloze
	  :END:
	
	The capital city of [New Zealand] is [Wellington], which is located in
	the [North||North/South] Island and has a population of about [400,000].

## Explainers

It is sometimes useful to add notes that give context to the
answer. This can be achieved through subheadings. In the example
below, `Notes` will be hidden when the question is raised, and
displayed with the answer.

    *** Greeting 1                                       :drill:

    Translate into Spanish:
    What is your name? (formal)

    **** Answer

    ??C??mo se llama usted?

    **** Notes

    llamar = to be named

While this works well, there times when it would be useful to add the
same note to several cards. Explainers allows this. An explanation
goes in the super heading and will be displayed with the answer. For
example:

    ** Addition                                                         :explain:

    Addition is used to combine two values into a larger one

    *** Question                                                          :drill:

    2 + 2 = [4]

    *** Question

    3 + 3 = [6]

When `2 + 2 = 4` is shown the explanation will be shown
also. Higher-level of explanations can be used also. For example, in
this case, both explanations will be shown for any question.


    * Mathematical Operators                                            :explain:

    Mathematical operators are used to change several numbers into one

    ** Addition                                                         :explain:

    Addition is used to combine two values into a larger one

    *** Question                                                          :drill:

    2 + 2 = [4]

    *** Question

    3 + 3 = [6]

    ** Subtraction                                                      :explain:

    Subtraction is used to remove one value from another to make a smaller one

    *** Question                                                          :drill:

    3 - 2 = [1]

    *** Question                                                          :drill:

    5 - 2 = [3]

This can be very useful, for example, when learning multiple examples
for grammatical rules.

## User-defined card types

Finally, you can write your own emacs lisp functions to define new kinds of
topics. Any new topic type will need to be added to
`org-drill-card-type-alist`, and cards using that topic type will need to have
it as the value of their `DRILL_CARD_TYPE` property. For examples, see the
functions at the end of org-drill.el &#x2013; these include:

-   `org-drill-present-verb-conjugation`, which implements the 'conjugate'
    card type. This asks the user to conjugate a verb in a particular tense. It
    demonstrates how the appearance of an entry can be completely altered during
    a drill session, both during testing and during the display of the answer.
-   `org-drill-present-translate-number`, which uses a third-party emacs lisp
    library ([spell-number.el](http://www.emacswiki.org/emacs/spell-number.el)) to prompt the user to translate random numbers
    to and from any language recognised by that library.
-   `org-drill-present-spanish-verb`, which defines the new topic type
    `spanish_verb`. This illustrates how a function can control which of an
    item's subheadings are visible during the drill session.

See the file [spanish.org](spanish.md) for a full set of example material, including examples
of all the card types discussed above.

## Empty cards

If the body of a drill item is completely empty (ignoring properties and child
items), then the item will be skipped during drill sessions. The purpose of
this behaviour is to allow you to paste in 'skeletons' of complex items, then
fill in missing information later. For example, you may wish to include an
empty drill item for each tense of a newly learned verb, then paste in the
actual conjugation later as you learn each tense.

Note that if an item is empty, any child drill items will **not** be ignored,
unless they are empty as well.

If you have an item with an empty body, but still want it to be included in a
drill session, you can either:

1.  Put a brief comment ('# &#x2026;')  in the item body.
2.  Change the entry for its card type in `org-drill-card-type-alist` so that
    items of this type will always be tested, even if they have an empty body.
    See the documentation for `org-drill-card-type-alist` for details.

# Running the drill session

Start a drill session with `M-x org-drill`. By default, this tests all
non-hidden topics in the current buffer. `org-drill` takes an optional
argument, SCOPE, which allows it to take drill items from other
sources. See below for details.

During a drill session, you will be presented with each item, then asked to
rate your recall of it by pressing a key between 0 and 5. The meaning of these
numbers is (taken from `org-learn`):

<table>


<colgroup>
<col  class="right">

<col  class="left">

<col  class="left">

<col  class="left">
</colgroup>
<thead>
<tr>
<th scope="col" class="right">Quality</th>
<th scope="col" class="left">SuperMemo label</th>
<th scope="col" class="left">Fail?</th>
<th scope="col" class="left">Meaning</th>
</tr>
</thead>

<tbody>
<tr>
<td class="right">0</td>
<td class="left">NULL</td>
<td class="left">Yes</td>
<td class="left">Wrong, and the answer is unfamiliar when you see it.</td>
</tr>


<tr>
<td class="right">1</td>
<td class="left">BAD</td>
<td class="left">Yes</td>
<td class="left">Wrong answer.</td>
</tr>


<tr>
<td class="right">2</td>
<td class="left">FAIL</td>
<td class="left">Yes</td>
<td class="left">Almost, but not quite correct.</td>
</tr>


<tr>
<td class="right">3</td>
<td class="left">PASS</td>
<td class="left">No</td>
<td class="left">Correct answer, but with much effort.</td>
</tr>


<tr>
<td class="right">4</td>
<td class="left">GOOD</td>
<td class="left">No</td>
<td class="left">Correct answer, with a little thought.</td>
</tr>


<tr>
<td class="right">5</td>
<td class="left">BRIGHT</td>
<td class="left">No</td>
<td class="left">Correct answer, effortless.</td>
</tr>
</tbody>
</table>

You can press '?'  at the prompt if you have trouble remembering what the
numbers 0-5 signify.

At any time you can press 'q' to finish the drill early (your progress up to
that point will be saved), 's' to skip the current item without viewing the
answer, or 'e' to escape from the drill and jump to the current topic for
editing (again, your progress up to that point will be saved).

After exiting the drill session with 'e' or 'q', you can resume where you left
off, using the command `org-drill-resume`. This will return you to the item
that you were viewing when you left the session. For example, if you are shown
an item and realise that it is poorly formulated, or contains an error, you can
press 'e' to leave the drill, then correct the item, then press
`M-x org-drill-resume` and continue where you left off.

Note that 'drastic' edits, such as deleting or moving items, can sometimes
cause Org-Drill to "lose its place" in the file, preventing it from
successfully resuming the session. In that case you will need to start a new
session.

# Multiple sequential drill sessions

Org-Drill has to scan your entire item database each time you start a new drill
session. This can be slow if you have a large item collection. If you have a
large number of 'due' items and want to run a second drill session after
finishing one session, you can use the command `org-drill-again` to run a new
drill session that draws from the pool of remaining due items that were not
tested during the previous session, without re-scanning the item collection.

Also note that if you run `org-drill-resume` and you have actually finished the
drill session, you will be asked whether you want to start another drill
session without re-scanning (as if you had run `org-drill-again`).

# Cram mode

There are some situations, such as before an exam, where you will want to
revise all of your cards regardless of when they are next due for review.

To do this, run a *cram session* with the `org-drill-cram` command (`M-x
org-drill-cram`). This works the same as a normal drill session, except
that all items are considered due for review unless you reviewed them within
the last 12 hours (you can change the number of hours by customising the
variable `org-drill-cram-hours`).

Cram sessions are not considered to be part of the normal learning process for
the tested items. Cramming will not affect when items are next due for
revision.

# Leeches


From the Anki website, <http://ichi2.net/anki/wiki/Leeches>:

> Leeches are cards that you keep on forgetting. Because they require so many
> reviews, they take up a lot more of your time than other cards.

Like Anki, Org-Drill defines leeches as cards that you have "failed" many
times. The number of times an item must be failed before it is considered a
leech is set by the variable `org-drill-leech-failure-threshold` (15 by
default). When you fail to remember an item more than this many times, the item
will be given the `:leech:` tag.

Leech items can be handled in one of three ways. You can choose how Org-Drill
handles leeches by setting the variable `org-drill-leech-method` to one of the
following values:

-   **nil:** Leech items are tagged with the `leech` tag, but otherwise treated the
    same as normal items.
-   **skip:** Leech items are not included in drill sessions.
-   **warn:** Leech items are still included in drill sessions, but a warning
    message is printed when each leech item is presented.

The best way to deal with a leech is either to delete it, or reformulate it so
that it is easier to remember, for example by splitting it into more than one
card.

See [the SuperMemo website](http://www.supermemo.com/help/leech.htm) for more on leeches.

# Customisation

Org-Drill has several settings which you change using
`M-x customize-group org-drill <RET>`. Alternatively you can change these
settings by adding elisp code to your configuration file (`.emacs`).

## Visual appearance of items during drill sessions

If you want cloze-deleted text to show up in a special font within Org mode
buffers, add this to your .emacs:

    (setq org-drill-use-visible-cloze-face-p t)

Item headings may contain information that "gives away" the answer to the item,
either in the heading text or in tags. If you want item headings to be made
invisible while each item is being tested, add:

    (setq org-drill-hide-item-headings-p t)

## Duration of drill sessions

By default, a drill session will end when either 30 items have been
successfully reviewed, or 20 minutes have passed. To change this behaviour, use
the following settings.

    (setq org-drill-maximum-items-per-session 40)
    (setq org-drill-maximum-duration 30)   ; 30 minutes

If either of these variables is set to nil, then item count or elapsed time
will not count as reasons to end the session. If both variables are nil, the
session will not end until *all* outstanding items have been reviewed.

## Saving buffers after drill sessions

By default, you will be prompted to save all unsaved buffers at the end of a
drill session. If you don't like this behaviour, use the following setting:

    (setq org-drill-save-buffers-after-drill-sessions-p nil)

## Sources of items for drill sessions (scope)


By default, Org-Drill gathers drill items from the current buffer only,
ignoring any non-visible items. There may be times when you want Org-Drill to
gather drill items from other sources. You can do this by changing the value of
the variable `org-drill-scope`. Possible values are:

-   **file:** The current buffer, ignoring hidden items. This is the default.
-   **tree:** The subtree starting with the entry at the cursor. (Alternatively you
    can use `M-x org-drill-tree` to run the drill session &#x2013; this will
    behave the same as `org-drill` if 'tree' was used as the value of
    SCOPE.)
-   **file-no-restriction:** The current buffer, including both hidden and
    non-hidden items.
-   **file-with-archives:** The current buffer, and any archives associated with it.
-   **agenda:** All agenda files.
-   **agenda-with-archives:** All agenda files with any archive files associated
    with them.
-   **directory:** All files with the extension '.org' in the same directory as the
    current file. (The current file will also be included if its
    extension is .org)
-   **(file1 file2 &#x2026;):** A list of filenames. All files in the list will be
    scanned.

## Definition of old and overdue items

Org-Drill prioritises *overdue* items in each drill session, presenting them
before other items are seen. Overdue items are defined in terms of how far in
the past the item is scheduled for review. The threshold is defined in terms
of a proportion rather than an absolute number of days. If days overdue is
greater than

    last-interval * (factor - 1)

and is at least one day overdue, then the item is considered 'overdue'. The
default factor is 1.2, meaning that the due date can overrun by 20% before the
item is considered overdue.

To change the factor that determines when items become overdue, use something
like the following in your .emacs. Note that the value should never be less
than 1.0.

    (setq org-drill-overdue-interval-factor 1.1)

After prioritising overdue items, Org-Drill next prioritises *young*
items. These are items which were recently learned (or relearned in the case of
a failure), and which therefore have short inter-repetition intervals.
"Recent" is defined as an inter-repetition interval less than a fixed number of
days, rather than a number of repetitions. This ensures that more difficult
items are reviewed more often than easier items before they stop being 'young'.

The default definition of a young item is one with an inter-repetition interval
of 10 days or less. To change this, use the following:

    (setq org-drill-days-before-old 7)

## Spaced repetition algorithm

### Choice of algorithm

Org-Drill supports three different spaced repetition algorithms, all based on
SuperMemo algorithms. These are:

-   **[SM2](http://www.supermemo.com/english/ol/sm2.htm):** an early algorithm, used in SuperMemo 2.0 (1988), which remains very
    popular &#x2013; Anki and Mnemosyne, two of the most popular spaced repetition
    programs, use SM2. This algorithm stores an 'ease factor' for each item,
    which is modified each time you rate your recall of the item.
-   **[SM5](http://www.supermemo.com/english/ol/sm5.htm) (default):** used in SuperMemo 5.0 (1989). This algorithm uses 'ease
    factors' but also uses a persistent, per-user 'matrix of optimal factors'
    which is also modified after each item repetition.
-   **Simple8:** an experimental algorithm based on the [SM8](http://www.supermemo.com/english/algsm8.htm) algorithm. SM8 is used
    in SuperMemo 8.0 (1998) and is almost identical to SM11 which is
    used in SuperMemo 2002. Like SM5, it uses a matrix of optimal
    factors. Simple8 differs from SM8 in that it does not adapt the
    matrix to the individual user, though it does adapt each item's
    'ease factor'.

If you want Org-Drill to use the `SM2` algorithm, put the following in your
`.emacs`:

    (setq org-drill-spaced-repetition-algorithm 'sm2)

### Random variation of repetition intervals

The intervals generated by the SM2 and SM5 algorithms are pretty
deterministic. If you tend to add items in large, infrequent batches, the lack
of variation in interval scheduling can lead to the problem of "lumpiness" &#x2013;
one day a large batch of items are due for review, the next there is almost
nothing, a few days later another big pile of items is due, and so on.

This problem can be ameliorated by adding some random "noise" to the interval
scheduling algorithm. The author of SuperMemo actually recommends this approach
for the SM5 algorithm, and Org-Drill's implementation uses [his code](http://www.supermemo.com/english/ol/sm5.htm).

To enable random "noise" for item intervals, set the variable
`org-drill-add-random-noise-to-intervals-p` to true by putting the following in
your `.emacs`:

    (setq org-drill-add-random-noise-to-intervals-p t)

### Adjustment for early or late review of items

Reviewing items earlier or later than their scheduled review date may affect
how soon the next review date should be scheduled. Code to make this adjustment
is also presented on the SuperMemo website. It can be enabled with:

    (setq org-drill-adjust-intervals-for-early-and-late-repetitions-p t)

This will affect both early and late repetitions if the Simple8 algorithm is
used. For the SM5 algorithm it will affect early repetitions only. It has no
effect on the SM2 algorithm.

### Adjusting the first interval (SM5 algorithm only)

In the SM5 algorithm, the initial interval after the first successful
presentation of an item is *always* 4 days. If you wish to change this for some
reason, you can do so with:

    (setq org-drill-sm5-initial-interval 5.0)

note that this will have no effect if you are not using the SM5 algorithm.

### Adjusting item difficulty globally

The `learn fraction` is a global value which affects how quickly the intervals
(times between each retest of an item) increase with successive repetitions,
for *all* items. The default value is 0.5, and this is the value used in
SuperMemo. For some collections of information, you may find that you are
reviewing items too often (they are too easy and the workload is too high), or
too seldom (you are failing them too often). In these situations, it is
possible to alter the learn fraction from its default in order to increase or
decrease the frequency of repetition of items over time. Increasing the value
will make the time intervals grow faster, and lowering it will make them grow
more slowly. The table below shows the growth in intervals (in days) with some
different values of the learn fraction (F). The table assumes that the item is
successfully recalled each time, with an average quality of just under 4.

<table>


<colgroup>
<col  class="left">

<col  class="right">

<col  class="right">

<col  class="right">

<col  class="right">

<col  class="right">
</colgroup>
<thead>
<tr>
<th scope="col" class="left">Repetition</th>
<th scope="col" class="right">F=0.3</th>
<th scope="col" class="right">F=0.4</th>
<th scope="col" class="right">**F=0.5**</th>
<th scope="col" class="right">F=0.6</th>
<th scope="col" class="right">F=0.7</th>
</tr>
</thead>

<tbody>
<tr>
<td class="left">1st</td>
<td class="right">2</td>
<td class="right">2</td>
<td class="right">2</td>
<td class="right">2</td>
<td class="right">2</td>
</tr>


<tr>
<td class="left">2nd</td>
<td class="right">7</td>
<td class="right">7</td>
<td class="right">7</td>
<td class="right">7</td>
<td class="right">7</td>
</tr>


<tr>
<td class="left">5th</td>
<td class="right">26</td>
<td class="right">34</td>
<td class="right">46</td>
<td class="right">63</td>
<td class="right">85</td>
</tr>


<tr>
<td class="left">10th</td>
<td class="right">85</td>
<td class="right">152</td>
<td class="right">316</td>
<td class="right">743</td>
<td class="right">1942</td>
</tr>


<tr>
<td class="left">15th</td>
<td class="right">233</td>
<td class="right">501</td>
<td class="right">1426</td>
<td class="right">5471</td>
<td class="right">27868</td>
</tr>
</tbody>
</table>

To alter the learn fraction, put the following in your .emacs:

    (setq org-drill-learn-fraction 0.45)   ; change the value as desired

## Per-file customisation settings


Most of Org-Drill's customisation settings are safe as file-local
variables. This means you can include a commented section like this at the end
of your .org file to apply special settings when running a Drill session using
that file:

    # Local Variables:
    # org-drill-maximum-items-per-session:    50
    # org-drill-spaced-repetition-algorithm:  simple8
    # End:

You can achieve the same effect by including the settings in the 'mode line'
(this must be the **first line** in the file), like so:

    # -*- org-drill-maximum-items-per-session: 50; org-drill-spaced-repetition-algorithm: simple8 -*-

In either case you will need to reload the file for the changes to take effect.

# Coping with large collections

If you keep all your items in a single file, it may eventually get very
large. The file will be slow to load, and Emacs may have trouble
syntax-highlighting the file contents correctly.

The easiest way to solve this problem is:

1.  Move your file into its own dedicated directory.
2.  Divide the file into two or more smaller files.
3.  Within each file, set `org-drill-scope` to 'directory'. See
    above for instructions about how to do this.

# Sharing, merging and synchronising item collections

Every drill item is automatically given a persistent unique "ID" the first time
it is seen by Org-Drill. This means that if two different people subsequently
edit or reschedule that item, Org-Drill can still tell that it is the same
item. This in turn means that collections of items can be shared and edited in
a collaborative manner.

There are two commands that are useful in this regard:

1.  `org-drill-strip-all-data` - this command deletes all user-specific
    scheduling data from every item in the current collection. (It takes the
    same optional 'scope' argument as `org-drill` to define which items will
    be processed by the command). User-specific data includes scheduling dates,
    ease factors, number of failures and repetitions, and so on. All items are
    reset to 'new' status. This command is useful if you want to share your
    item collection with someone else.
2.  `org-drill-merge-buffers` - When called from buffer A, it prompts you for
    another buffer (B), which must also be loaded into Emacs. This command
    imports all the user-specific scheduling data from buffer B into buffer A,
    and deletes any such information in A. Matching items are identified by
    their ID. Any items in B that do not exist in A are copied to A, in
    the same hierarchical location if all the parent headings exist, otherwise
    at the end of the buffer.

An example scenario:

Tim decides to learn Swedish using an item collection (`.org` file) made
publically available by Jane.  (Before publishing it Jane used
'org-drill-strip-all-data' to remove her personal scheduling data from the
collection.)  A few weeks later, Jane updates her collection, adding new items
and revising some old ones. Tim downloads the new collection and imports his
progress from his copy of the old collection, using 'org-drill-merge-buffers',
using the new collection as buffer A and the old one as buffer B. He can then
discard the old copy. Any items HE added to HIS copy of the old collection
(buffer B) will not be lost &#x2013; they will be appended to his copy of the new
collection.

Of course the sharing does not need to be 'public'. You and a friend might be
learning a language or some other topic together. You each maintain a card
collection. Periodically your friend sends you a copy of their collection &#x2013;
you run `org-drill-merge-buffers` on it, always using your own collection as
buffer B so that your own scheduling progress is carried over. Other times you
send your friend a copy of your collection, and he or she follows the same
procedure.

# Incremental reading

An innovative feature of the program SuperMemo is so-called "incremental
reading". This refers to the ability to quickly and easily make drill items
from selected portions of text as you read an article (a web page for
example). See [the SuperMemo website](http://www.supermemo.com/help/read.htm) for more on incremental reading.

Much of the infrastructure for incremental reading is already provided by Org
Mode, with the help of some other emacs packages. You can provide yourself with
an incremental reading facility by using 'org-capture' alongside a package that
allows you to browse web pages either in emacs (w3 or [emacs-w3m](http://www.emacswiki.org/emacs/emacs-w3m)) or in the
external browser of your choice ([org-protocol](http://orgmode.org/worg/org-contrib/org-protocol.php)).

Another important component of incremental reading is the ability to save your
exact place in a document, so you can read it *incrementally* rather than all
at once. There is a large variety of bookmarking packages for emacs which
provide advanced bookmarking functionality: see the [Emacs Wiki](http://www.emacswiki.org/emacs/BookMarks) for details.
Bookmarking exact webpage locations in an external browser seems to be a bit
more difficult. For Firefox, the [Wired Marker](http://www.wired-marker.org/) addon works well.

An example of using Org-Drill for incremental reading is given below. First,
and most importantly, we need to define a couple of `org-capture` templates for
captured facts.

    (setq org-capture-templates
           `(("u"
             "Task: Read this URL"
             entry
             (file+headline "tasks.org" "Articles To Read")
             ,(concat "* TODO Read article: '%:description'\nURL: %c\n\n")
             :empty-lines 1
             :immediate-finish t)
    
            ("w"
             "Capture web snippet"
             entry
             (file+headline "my-facts.org" "Inbox")
             ,(concat "* Fact: '%:description'        :"
                      (format "%s" org-drill-question-tag)
                      ":\n:PROPERTIES:\n:DATE_ADDED: %u\n:SOURCE_URL: %c\n:END:\n\n%i\n%?\n")
             :empty-lines 1
             :immediate-finish t)
            ;; ...other capture templates...
        ))

Using these templates and `org-protocol`, you can set up buttons in your web
browser to:
-   Create a task telling you to read the URL of the currently viewed webpage
-   Turn a region of selected text on a webpage, into a new fact which is saved
    to whichever file and heading you nominate in the template. The fact will
    contain a timestamp, and a hyperlink back to the webpage where you created
    it.

For example, suppose you are reading the Wikipedia entry on tuberculosis [here](http://en.wikipedia.org/wiki/Tuberculosis).

You read the following:

> The classic symptoms of tuberculosis are a chronic cough with blood-tinged
> sputum, fever, night sweats, and weight loss. Infection of other organs causes
> a wide range of symptoms. Treatment is difficult and requires long courses of
> multiple antibiotics. Antibiotic resistance is a growing problem in
> (extensively) multi-drug-resistant tuberculosis. Prevention relies on screening
> programs and vaccination, usually with Bacillus Calmette-Gu??rin vaccine.

You decide you want to remember that "Bacillus Calmette-Gu??rin vaccine" is the
name of the vaccine against tuberculosis. First, you select the \`interesting'
portion of the text with the mouse:

> The classic symptoms of tuberculosis are a chronic cough with blood-tinged
> sputum, fever, night sweats, and weight loss. Infection of other organs causes
> a wide range of symptoms. Treatment is difficult and requires long courses of
> multiple antibiotics. Antibiotic resistance is a growing problem in
> (extensively) multi-drug-resistant tuberculosis.
> <font style="background-color: yellow;">Prevention relies
> on screening programs and vaccination, usually with Bacillus Calmette-Gu??rin
> vaccine.</font>

Then you press the button you created when setting up `org-protocol`, which is
configured to activate the capture template "w: Capture web snippet". The
selected text will be sent to Emacs, turned into a new fact using the template,
and filed away for your later attention.

(Note that it might be more efficient to turn the entire paragraph into a drill
item &#x2013; since it contains several important facts &#x2013; then split it up into
multiple items when you edit it later in Emacs.)

Once you have had enough of reading the article, save your place, then go to
your "fact" file in Emacs. You should see that each piece of text you selected
has been turned into a drill item. Continuing the above example, you would see
something like:

	** Fact: 'Tuberculosis - Wikipedia, the Free Encyclopedia'        :drill:
	
	Prevention relies on screening programs and vaccination, usually with Bacillus
	Calmette-Gu??rin vaccine.

You need to edit this fact so it makes sense independent of its context, as
that is how it will be presented to you in future. The easiest way to turn the
text into a 'question' is by cloze deletion. All you need to do is surround the
'hidden' parts of the text with square brackets.

    Prevention of tuberculosis relies on screening programs and vaccination,
    usually with [Bacillus Calmette-Gu??rin vaccine].

You can of course define browser buttons that use several different "fact"
templates, each of which might send its fact to a different file or subheading,
or give it different tags or properties, for example.

# Author

Org-Drill is maintained by Phillip Lord.

Org-Drill was originally written by Paul Sexton.

# History

This version of org-drill is a fork of the original written by Paul.
I (Phil Lord) made this fork as org-drill was unmaintained.

https://bitbucket.org/eeeickythump/org-drill/issues/63/maintainership

Paul did email me and tell me that he was happy with this, but I have
subsequently been unable to get access to the original project; as
suggested in the original issue, I have moved to git because I am more
familiar with it.

Currently, I am refactoring org-drill significantly. It's quite hard
to test all of its functionality automatically, so I expect
breakages. In addition, some of the interfaces for card types has
changed, and some functionality which is unmaintainable is being
removed! Please be patient and report problems!
