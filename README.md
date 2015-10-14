Last year we designed and implemented a programming language called Apollo with a team of five.

#### Description of the project
Apollo is a functional programming language for algorithmic and musical composition.
It provides an interface to leverage functional paradigms in order to produce a target program that generates a musical output when run.
It is intended to be usable by a programmer with knowledge of basic functional constructs and no prior experience with music creation.

#### Why I chose this
Although it was for a class, I decided to share this example because I am very passionate about Apollo.
We put much more work into the project than the class required and spent months designing and implementing it.

In addition to that, we wrote Apollo in Haskell, which in my opinion is a very beautiful language.
It makes you really think about what you're coding and you often write very elegant and concise code.

#### Code sample
Since it was such a large project, we all had to design and write different parts that worked together.
One of the tasks was to tie the data structures representing a musical piece in the language with a MIDI library in order to output a MIDI file.
I wrote `midi.hs` - a mini library that converts `Music` (our data type) into a MIDI file.

What I want to explain in more detail is how one of the helper functions, `partToTrack` works.

Functional programming code is dense, so I tried my best to explain it in detail.

#### partToTrack:

##### Description:
`partToTrack` takes a list of Atoms and returns a list of Tracks.

* A `Track` is a list of Messages which signal the midi player to perform an action (such as play a note).
* The midi executes the list of Tracks in parallel (so it can execute two messages simultaniously).

##### Code
``` haskell
-- Takes a part and outputs [[Track]] with padding using partToTracKHelp
partToTrack :: [Atom] -> [[(Ticks, Message)]]
partToTrack atoms = partToTrackHelp (replicate (longestAtom atoms) []) atoms

-- Appends all Atoms to a list of tracks with Rest padding
partToTrackHelp :: [[(Ticks, Message)]] -> [Atom] -> [[(Ticks, Message)]]
partToTrackHelp container []
    = zipWith (++) container $ replicate (length container) [(0, TrackEnd)]
partToTrackHelp container (x:xs)
    = partToTrackHelp (zipWith (++) container (appendRests (length container) x)) xs
```

##### Explanation

`partToTrack` essentially takes a list of Atoms, converts them to Messages and transposes them vertically.

It converts this:
```
Atoms
[[A],
 [B, E, F],
 [C, D]]
```
To this:
```
Tracks
[["Play A", "Play B", "Play C", "Track End"],
 ["Play -", "Play E", "Play D", "Track End"],
 ["Play -", "Play F", "Play -", "Track End"]]
```

First it gets the size of the largest Atom to determine the number of rows the matrix for Tracks should be.
(In the example above the row count is three)

Then it passes the list of Atoms and an empty matrix with the appropriate size to `partToTrackHelp`.

`partToTrackHelp` takes the empty container and the list of Atoms and returns a list of corresponding Tracks.

In order to create Tracks it goes through these:

If the head Atom is empty then return a list of "Track End" messages. (base case)

Otherwise:

1. Take the head Atom of the list and convert it into a list of messages with padding ("Play -").

2. Concatenate the converted Atom onto the container.

3. Recursively call `partToTrackHelp` with the tail of the list

So with the example input above, the recursion would go to depth four.

The container would be passed down like so:

```
[[]
 []
 []]
```
```
[["Play A"],
 ["Play -"],
 ["Play -"]]
```
```
[["Play A", "Play B"],
 ["Play -", "Play E"],
 ["Play -", "Play F"]]
```
```
[["Play A", "Play B", "Play C"],
 ["Play -", "Play E", "Play D"],
 ["Play -", "Play F", "Play -"]]
```
```
[["Play A", "Play B", "Play C", "Track End"],
 ["Play -", "Play E", "Play D", "Track End"],
 ["Play -", "Play F", "Play -", "Track End"]]
```

The MIDI would  have three tracks (rows)

And it would play the music by going through the columns:

1. Play A, nothing, and nothing

2. Play B, E and F

3. Play C, D and nothing

4. End
