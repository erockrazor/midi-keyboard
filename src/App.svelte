<script>
  import * as Tone from "tone";
  var synth = new Tone.PolySynth(Tone.Synth).toDestination();
  // Request MIDI access
  if (navigator.requestMIDIAccess) {
    console.log("This browser supports WebMIDI!");

    navigator.requestMIDIAccess().then(onMIDISuccess, onMIDIFailure);
  } else {
    console.log("WebMIDI is not supported in this browser.");
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
        noteOffCallback(note);
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

  function noteOffCallback(note) {
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

  // This function will trigger certain animations and advance gameplay
  // when certain criterion are identified by the noteOn/noteOff listeners
  // For instance, a lock is unlocked, the timer expires, etc.
  function runSequence(sequence) {
    //...
  }
</script>

<style>
</style>

<main>
  <h2>Midi Keyboard App</h2>
</main>
