<script>
  import * as Tone from "tone";
  import Vex from "vexflow";
  import { onMount } from "svelte";

  var synth = new Tone.PolySynth().toDestination();
  onMount(async () => {
    // Request MIDI access
    if (navigator.requestMIDIAccess) {
      console.log("This browser supports WebMIDI!");

      navigator.requestMIDIAccess().then(onMIDISuccess, onMIDIFailure);
    } else {
      console.log("WebMIDI is not supported in this browser.");
    }
    renderNotation();
  });

  function renderNotation() {
    // * onMount waits for the DOM to be loaded.
    const VF = Vex.Flow;

    // Create an SVG renderer and attach it to the DIV element named "vf".
    const renderer = new VF.Renderer(
      "notationContainer",
      VF.Renderer.Backends.SVG
    );

    // Configure the rendering context.
    renderer.resize(500, 500);
    const context = renderer.getContext();
    context.setFont("Arial", 10, "").setBackgroundFillStyle("#eed");

    // Create a stave of width 400 at position 10, 40 on the canvas.
    const stave = new VF.Stave(10, 40, 400);

    // Add a clef and time signature.
    stave.addClef("treble");

    // Connect it to the rendering context and draw!
    stave.setContext(context).draw();
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
    console.log(`${noteName} on`);
    synth.triggerAttack(noteName);
    console.log(note);
    //...
  }

  // Function to handle noteOff messages (ie. key is released)
  // Think of this like an 'onkeyup' event
  function noteOff(note) {
    console.log(note);
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
</style>

<main>
  <h2>Midi Keyboard App</h2>
  <div id="notationContainer" />
</main>
