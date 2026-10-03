# Ear

Interval and chord practice for a commute, by ear and by voice, on an Android
phone with the screen off.

It began as the third app inside a web server and has left: the browser could
not hold the microphone once the screen locked, and its speech recognition went
over the network, which is the opposite of a commute. A foreground service
typed `microphone` holds it. That is the entire reason this exists.

**Nothing leaves the phone.** The app asks for no network permission at all —
synthesis, recognition, scoring and your record are all local. The install
prompt can say so because it is true.

## How it works

**Your voice only ever answers the question.** Every control is a button. The
recogniser never has to tell an answer from an instruction, a stray word cannot
trigger one, and the grammar shrinks to the things being taught.

**Recognition is offline and grammar-constrained.** Vosk is told the only
things you could sensibly say — the interval and chord names, and their
aliases. It chooses between those, not between them and the whole English
language. `[unk]` is in the grammar, so humming to find a note comes back as
out-of-grammar and is ignored rather than scored.

**Every answer is also a key.** The vocabulary is a keypad: tap an answer and
it scores exactly as speaking it would. A recogniser that mishears you twice
running should not be the only way past a question.

**Silence is not a wrong answer.** If nothing is said the question plays again,
up to five times, and then the session pauses — silence usually means you did
not hear it, or that the phone is in a pocket.

**The questions never run out.** Each is generated on the spot: a fresh item
weighted towards what you miss, a root that never repeats twice running, and an
instrument picked at random. Melodic questions ascend. Reset clears the tally
for that exercise and keeps going.

**Three voices**: piano, classical guitar, acoustic guitar. Additive piano with
string inharmonicity; Karplus-Strong guitars with a fractionally interpolated
delay line, because integer delay lengths detune high notes by up to 17 cents
and interval identification is the whole task. Guitars land within 0.4 cents
absolute, interval error under 0.35 cents.

**Every question is the same loudness.** Notes are levelled on measured
loudness rather than peak — a peak tracks the sharpest transient, a piano
hammer or a steel pick, while what you hear is the body of the note. Peak
normalisation left an 8.8 dB spread across voices and chord shapes; this holds
it to 0.01 dB, with a soft knee on the peaks.

**A guitar neck, always on the page.** Six strings, twelve frets, playable:
tap a fret and it sounds. After a wrong answer it marks where the notes were,
and clears when the next question starts. The fingering is chosen as a whole —
lowest position on the neck, then most compact — so it is a shape a hand could
hold.

**The score belongs to the exercise.** Fifths answered right say nothing about
your sevenths, so each lesson carries its own ticks and crosses, on the main
page and in the menu alike.

**A headset takes the whole session or none of it.** Bluetooth earbuds are two
devices pretending to be one: good audio on one profile with no microphone, and
the microphone on a narrowband call channel that suspends the first. You cannot
have both. With a headset connected the tones, the prompts and the answers all
go over it; with nothing connected the phone keeps its microphone and its full
sample rate.

## Before it will build

The speech model is not in git — see `app/src/main/assets/PUT-THE-MODEL-HERE.md`.
Roughly 40 MB, downloaded once. CI fetches and caches it.

First time on a machine, read `FIRST-RUN.md`.

## Source of truth

`Intervals.kt` holds the interval table, the chord table and the lesson ladder,
and it is the only copy — there were three, in three languages, and a test
suite whose main job was proving they still agreed.

The tests run on the JVM, so CI runs them on every push without a device:

- `IntervalsTest` — the ladder and the tables, asserted directly.
- `SynthTest` — loudness measured with its own meter, because a normaliser
  checked with its own ruler proves nothing.
- `NeckTest` — every question the drill can ask, checked that each note is
  drawn on a fret that really sounds it.
- `PageTest` — the decisions a compiler cannot hold: an order, a colour, a word
  that should not be on the screen.

## Builds

Every push builds an APK and the default branch publishes it where a phone can
install it, at `releases/download/latest/ear.apk`. The build fails unless it
can prove that link answers 200.
