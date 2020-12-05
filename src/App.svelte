<script>
  import * as Tone from "tone";
  import { onMount } from "svelte";
  import abcjs from "abcjs";
  import WebMidi from "webmidi";
  import { Piano } from "@tonejs/piano";

  const piano = new Piano({
    velocities: 5,
  }).toDestination();
  piano.load().then(() => {
    console.log("Piano Samples loaded.");
  });
  let targetNote = null;
  let actionPrompt = "Press any midi keyboard key to start the game";
  let points = 0;
  let highScore = 0;
  let timerRunning = false;
  let readyToStartNewGame = true;

  onMount(async () => {
    // * onMount waits for the DOM to be loaded.
    // Request MIDI access
    requestMidiAccess();
    const randomMidiValue = getRandomNote(48, 72);
    targetNote = getNoteFromMidiInteger(randomMidiValue);
    renderAbcNotation();
  });

  function renderAbcNotation() {
    abcjs.renderAbc("paper", generateAbcString(targetNote), {
      responsive: "resize",
      add_classes: true,
    });
  }

  function generateAbcString(targetNote) {
    console.log(`X:1
      M:C
      L:1/4
      K:C
      |:${convertNoteToAbcNotation(targetNote)},|]|`);
    return `X:1
      M:C
      %%staves {V1 V2}
      V: V1 clef=treble
      V: V2 clef=bass
      L:1/4
      K:C
      |:${convertNoteToAbcNotation(targetNote)},|]|`;
  }

  function convertNoteToAbcNotation(note) {
    let abcNote = note;
    // * We need to first find sharps ex: C#4 to convert them into ^C4
    // * Then flats are ex: _C4
    if (note.includes("#")) {
      let split = abcNote.split("#");
      abcNote = `^${split.join("")}`;
      console.log(abcNote);
    }
    if (abcNote.includes("5")) {
      abcNote = abcNote.replace("5", "").toLowerCase();
      console.log(abcNote);
    } else if (abcNote.includes("3")) {
      abcNote = `,${abcNote.replace("3", "")}`;
      console.log(abcNote);
    }
    return abcNote;
  }

  function requestMidiAccess() {
    WebMidi.enable(function (err) {
      if (err) {
        console.log("WebMidi could not be enabled.", err);
      } else {
        console.log("WebMidi enabled!");
        var input = WebMidi.inputs[1];
        // Listen for a 'note on' message on all channels
        input.addListener("noteon", "all", function (e) {
          noteOn(e.note.number, e.velocity);
        });
        input.addListener("noteoff", "all", function (e) {
          noteOff(e.note.number);
        });
        input.addListener("controlchange", "all", function (e) {
          if (e.controller.name === "holdpedal") {
            if (e.value === 127) {
              piano.pedalDown();
            }
            if (e.value === 0) {
              piano.pedalUp();
            }
          }
        });
      }
    });
  }

  function getRandomNote(min, max) {
    min = Math.ceil(min);
    max = Math.floor(max);
    return Math.floor(Math.random() * (max - min + 1)) + min;
    // 48 to 72 is c3 to c5
  }

  function timer() {
    timerRunning = true;
    readyToStartNewGame = false;
    actionPrompt = 10;
    const interval = setInterval(() => {
      actionPrompt--;
      if (actionPrompt <= 0) {
        clearInterval(interval);
        timerRunning = false;
        actionPrompt = "Press any midi keyboard key to start the game";
        if (points > highScore) {
          highScore = points;
        }
        setTimeout(() => {
          readyToStartNewGame = true;
        }, 5000);
      }
    }, 1000);
  }

  function checkForCorrectNote(note) {
    if (note === targetNote) {
      const randomMidiValue = getRandomNote(60, 72);
      targetNote = getNoteFromMidiInteger(randomMidiValue);
      renderAbcNotation();
      if (timerRunning) {
        points = points + 1;
      }
    } else {
      points = points - 1;
    }
  }

  // Function to handle noteOn messages (ie. key is pressed)
  // Think of this like an 'onkeydown' event
  function noteOn(note, velocity) {
    var noteName = getNoteFromMidiInteger(note);
    checkForCorrectNote(noteName);
    piano.keyDown({ note: noteName, velocity: velocity });
    if (!timerRunning && readyToStartNewGame) {
      points = 0;
      timer();
    }
    //...
  }

  // Function to handle noteOff messages (ie. key is released)
  // Think of this like an 'onkeyup' event
  function noteOff(note) {
    var noteName = getNoteFromMidiInteger(note);
    piano.keyUp({ note: noteName });

    //...
  }
  function getNoteFromMidiInteger(note) {
    var noteSub = note % 12;
    var octave = (note - 60 - noteSub) / 12 + 4;

    var noteName =
      ["C", "C#", "D", "D#", "E", "F", "F#", "G", "G#", "A", "A#", "B"][
        noteSub
      ] + octave;

    return noteName;
  }
</script>

<style>
</style>

<main>
  <section class="h-100 d-flex">
    <h2 class="text-center">{actionPrompt}</h2>
    <div id="paper" />
    <div class="d-flex justify-content-around">
      <p class="text-center">Points: {points}</p>
      <p class="text-center">High Score: {highScore}</p>
      <p class="text-center">Note Hint: {targetNote}</p>
    </div>
    <p class="text-center">Keep learning.</p>
  </section>
</main>
