<script>
  import * as Tone from "tone";
  import { onMount } from "svelte";
  import abcjs from "abcjs";
  import WebMidi from "webmidi";

  const synth = new Tone.PolySynth().toDestination();
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
    const randomMidiValue = getRandomNote(60, 72);
    targetNote = getNoteFromMidiInteger(randomMidiValue);
    renderAbcNotation();
    console.log(targetNote);
  });

  function renderAbcNotation() {
    abcjs.renderAbc("paper", generateAbcString(targetNote), {
      responsive: "resize",
      add_classes: true,
    });
  }

  function generateAbcString(targetNote) {
    return `X:1
      M:C
      L:1/4
      K:C
      |:${convertNoteToAbcNotation(targetNote)},|]|`;
  }

  function convertNoteToAbcNotation(note) {
    let abcNote = "";
    // * We need to first find sharps ex: C#4 to convert them into ^C4
    // * Then flats are ex: _C4
    if (note.includes("#")) {
      let split = note.split("#");
      abcNote = `^${split.join("")}`;
      console.log(abcNote);
    } else {
      return note;
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
          noteOn(e.note.number);
        });
        input.addListener("noteoff", "all", function (e) {
          noteOff(e.note.number);
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

  // Function to handle noteOn messages (ie. key is pressed)
  // Think of this like an 'onkeydown' event
  function noteOn(note) {
    var noteName = getNoteFromMidiInteger(note);
    checkForCorrectNote(noteName);
    console.log(`${noteName} on`);
    synth.triggerAttack(noteName);
    if (!timerRunning && readyToStartNewGame) {
      points = 0;
      timer();
    }
    //...
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
    console.log("target note", targetNote);
    console.log("note played", note);
    if (note === targetNote) {
      const randomMidiValue = getRandomNote(60, 72);
      targetNote = getNoteFromMidiInteger(randomMidiValue);
      console.log(targetNote);
      renderAbcNotation();
      console.log("correct note!");
      if (timerRunning) {
        points = points + 1;
      }
    } else {
      console.log("WRONG note!");
    }
  }

  // Function to handle noteOff messages (ie. key is released)
  // Think of this like an 'onkeyup' event
  function noteOff(note) {
    var noteName = getNoteFromMidiInteger(note);
    console.log(`${noteName} off`);
    synth.triggerRelease(noteName);
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
