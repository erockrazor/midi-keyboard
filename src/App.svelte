<script>
  import * as Tone from "tone";
  import Vex from "vexflow";
  import { onMount } from "svelte";

  const synth = new Tone.PolySynth().toDestination();
  let targetNote = null;
  let notation;
  const VF = Vex.Flow;

  onMount(async () => {
    // * onMount waits for the DOM to be loaded.
    // Request MIDI access
    requestMidiAccess();
    const randomMidiValue = getRandomNote(60, 72);
    targetNote = getToneNoteFromInteger(randomMidiValue);
    console.log(targetNote);
    renderNotation(targetNote);
  });

  function requestMidiAccess() {
    if (navigator.requestMIDIAccess) {
      console.log("This browser supports WebMIDI!");
      navigator.requestMIDIAccess().then(onMIDISuccess, onMIDIFailure);
    } else {
      console.log("WebMIDI is not supported in this browser.");
    }
  }

  function getRandomNote(min, max) {
    min = Math.ceil(min);
    max = Math.floor(max);
    return Math.floor(Math.random() * (max - min + 1)) + min;
    // 48 to 72 is c3 to c5
  }

  function removeCurrentInstanceOfNotation() {
    return (notation = {});
  }
  function removeContentFromNotationContainerBeforeRepaint() {
    return (document.getElementById("notation-container").innerHTML = "");
  }
  function renderNotation(randomNote) {
    removeCurrentInstanceOfNotation();
    removeContentFromNotationContainerBeforeRepaint();
    notation = new VF.Factory({
      renderer: { elementId: "notation-container" },
    });
    const ctx = notation.getContext();
    ctx.setFillStyle("#fff");
    ctx.setStrokeStyle("fff");
    const score = notation.EasyScore();
    const system = notation.System();
    system
      .addStave({
        voices: [score.voice(score.notes(`${randomNote}/w`, { stem: "up" }))],
      })
      .addClef("treble")
      .addTimeSignature("4/4");
    notation.draw();
  }

  // Function to run when requestMIDIAccess is successful
  function onMIDISuccess(midiAccess) {
    var inputs = midiAccess.inputs;
    var outputs = midiAccess.outputs;

    // Attach MIDI event "listeners" to each input
    for (var input of midiAccess.inputs.values()) {
      input.onmidimessage = getMIDIMessage;
    }
  }

  // Function to run when requestMIDIAccess fails
  function onMIDIFailure() {
    console.log("Error: Could not access MIDI devices.");
  }

  // Function to parse the MIDI messages we receive
  // For this app, we're only concerned with the actual note value,
  // but we can parse for other information, as well
  function getMIDIMessage(message) {
    var command = message.data[0];
    var note = message.data[1];
    var velocity = message.data.length > 2 ? message.data[2] : 0; // a velocity value might not be included with a noteOff command

    switch (command) {
      case 144: // note on
        if (velocity > 0) {
          noteOn(note);
        } else {
          noteOff(note);
        }
        break;
      case 128: // note off
        noteOff(note);
        break;
      // we could easily expand this switch statement to cover other types of commands such as controllers or sysex
    }
  }

  // Function to handle noteOn messages (ie. key is pressed)
  // Think of this like an 'onkeydown' event
  function noteOn(note) {
    var noteName = getToneNoteFromInteger(note);
    checkForCorrectNote(noteName);
    console.log(`${noteName} on`);
    synth.triggerAttack(noteName);
    //...
  }

  function checkForCorrectNote(note) {
    console.log("target note", targetNote);
    console.log("note played", note);
    if (note === targetNote) {
      const randomMidiValue = getRandomNote(60, 72);
      targetNote = getToneNoteFromInteger(randomMidiValue);
      console.log(targetNote);
      renderNotation(targetNote);
      console.log("correct note!");
    } else {
      console.log("WRONG note!");
    }
  }

  // Function to handle noteOff messages (ie. key is released)
  // Think of this like an 'onkeyup' event
  function noteOff(note) {
    var noteName = getToneNoteFromInteger(note);
    console.log(`${noteName} off`);
    synth.triggerRelease(noteName);
    //...
  }
  function getToneNoteFromInteger(note) {
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
  .w-100 {
    width: 100%;
  }
  .h-100 {
    height: 100vh;
  }
  .d-flex {
    display: flex;
  }
  .justify-content-center {
    justify-content: center;
  }
  .text-center {
    text-align: center;
  }
  section {
    display: flex;
    flex-direction: column;
    justify-content: space-around;
  }
</style>

<main>
  <section class="h-100 d-flex">
    <p class="text-center">Browser Based Sight Reading Practice Thingamabob</p>
    <div id="notation-container" class="w-100 d-flex justify-content-center" />
    <p class="text-center">{targetNote}</p>
    <p class="text-center">Keep learning.</p>
  </section>
</main>
